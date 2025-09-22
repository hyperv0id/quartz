## RTFM

### 并发性说明：

   理解谁是`struct pthread`或`PD`（指函数参数`struct pthread *pd`的值）
   的所有者至关重要，以确定哪些操作是允许的，哪些是不允许的，以及何时允许，特别是
   当涉及到`pthread_create`、`pthread_join`、`pthread_detach`等所有操作`PD`的函数的实现时。

   PD的所有者负责释放与PD相关联的最终资源，并且可以在任何时候检查PD下的内存，
   直到它将其释放回操作系统或重新使用。

   调用pthread_create的线程称为创建线程。创建线程起初是PD的所有者。

   在启动期间，新线程可以与拥有者线程（可能是它自己）协调检查PD。

   所有权转移的四种情况是：

   1. 新线程以可加入状态开始后，PD的所有权被释放到进程（所有线程都可以使用）
       即pthread_create返回一个可用的pthread_t。

   2. 新线程以分离状态开始时，PD的所有权被释放给新线程。

   3. 通过pthread_detach，PD的所有权被动态释放给一个运行中的线程。

   4. 调用pthread_join的线程获得了PD的所有权。

### 实现说明：

   PD->stopped_start和thread_ran变量用于确定我们正处于四种所有权状态中的哪一种，
   因此可以采取什么行动。例如，在(2)之后，我们不能再从PD读取或写入了，
   因为线程可能不再存在，内存可能已经被取消映射。

   重要的是要指出PD->lock被用作一次性信号量，随后又被用作互斥锁。
   父线程获取锁以迫使子线程等待，然后子线程释放锁。然而，这种类似信号量的效果
   仅用于同步父线程和子线程。启动后，锁被用作互斥锁，以创建一个临界区，
   在这个区域内，单个所有者修改线程参数。

   最复杂的情况发生在线程启动期间：

   1. 如果创建的线程处于分离（PTHREAD_CREATE_DETACHED）或可加入（默认PTHREAD_CREATE_JOINABLE）状态，
       并且STOPPED_START为真，则创建线程拥有PD的所有权，直到pthread_create释放PD->lock。
       如果发生任何错误，我们处于下面的(c)或(d)状态。

   2. 如果创建的线程处于分离状态（PTHREAD_CREATED_DETACHED），并且STOPPED_START为假，
       那么创建线程拥有PD的所有权，直到它调用操作系统内核的线程创建例程。
       如果此例程返回且没有错误，则创建线程拥有PD；否则，见下面的(c)或(d)。

   3. 如果可加入或分离线程设置失败并且THREAD_RAN为真，则创建线程将所有权释放给新线程，
       创建的线程通过PD->setup_failed成员看到失败的设置，释放PD所有权，并退出。
       创建线程将负责清理分配的资源。THREAD_RAN是创建线程的局部变量，指示线程创建或设置是否失败。

   4. 如果线程创建失败并且THREAD_RAN为假（意味着ARCH_CLONE失败），则创建线程保留PD的所有权
       并且必须清理分配的资源。不需要等待新线程，因为它从未开始。

### nptl_db接口：

   与nptl_db的接口要求我们将PD加入到一个链表中，然后调用一个函数，调试器将捕获这个函数。
   然后PD将从链表中删除，控制权返回给线程。调用者在此时必须拥有PD的所有权，并且在控制权返回给线程后，
   这样的所有权仍然保留。被加入的PD由nptl_db回调td_thr_event_getmsg从链表中删除。
   调试器必须确保线程不会恢复执行，否则可能会失去PD的所有权，并且无法检查PD。


### 公理：

   * create_thread函数永远不能将stopped_start设置为false。
   * 创建的线程可以读取stopped_start，但永远不能写入它。
   * 变量thread_ran在操作系统线程创建例程返回后的某个时间点被设置，
     线程创建后多久设置是未指定的，但应该尽可能快。



### 创建线程说明

   create_thread必须初始化PD->stopped_start。如果STOPPED_START参数为真，
   或者如果create_thread需要新线程在启动时进行某些其他实现原因的同步，则应该是true。
   如果STOPPED_START将为真，那么create_thread有义务在启动线程之前锁定PD->lock。
   然后pthread_create解锁PD->lock，这与子线程中的create_thread同步，
   子线程在调用用户入口点之前对PD->lock进行获取/释放，这是最后一个动作。
   所有这些的目的是确保创建线程在新线程运行用户代码之前应用所需的初始线程属性。
   请注意，`pthread_getschedparam`、`pthread_setschedparam`、`pthread_setschedprio`、
   `__pthread_tpp_change_priority`和`__pthread_current_priority`等函数出于类似目的重用相同的锁，
   `PD->lock`，例如同步设置类似的线程属性。这些函数从未在线程创建之前被调用，
   所以不参与启动同步，但鉴于锁已经存在且处于解锁状态，重用它节省了空间。

   返回值是零表示成功，或者是失败的错误码errno。如果返回值是ENOMEM，
   那将被转换为`EAGAIN`，所以`create_thread`不需要这样做。失败时，`*THREAD_RAN`应该被设置为true，
   如果线程实际启动了但在调用用户代码`*PD->start_routine`之前。 
## 源码

```c
static int create_thread (struct pthread *pd, const struct pthread_attr *attr,
			  bool *stopped_start, void *stackaddr,
			  size_t stacksize, bool *thread_ran)
{
  /* 确认新创建的线程是否需要在启动时停止，因为我们需要设置调度参数或设置亲和性。 */
  bool need_setaffinity = (attr != NULL && attr->extension != NULL
			   && attr->extension->cpuset != 0);
  if (attr != NULL
      && (__glibc_unlikely (need_setaffinity)
	  || __glibc_unlikely ((attr->flags & ATTR_FLAG_NOTINHERITSCHED) != 0)))
    *stopped_start = true;

  pd->stopped_start = *stopped_start;
  if (__glibc_unlikely (*stopped_start))
    lll_lock (pd->lock, LLL_PRIVATE);

/* 我们高度依赖CLONE函数理解的各种标志：

  CLONE_VM, CLONE_FS, CLONE_FILES
  这些标志选择共享地址空间和文件描述符的语义，符合POSIX的要求。

  CLONE_SIGHAND, CLONE_THREAD
  此标志选择POSIX信号语义和各种共享（itimers, POSIX timers等）。

  CLONE_SETTLS
  CLONE的第六个参数确定新线程的TLS区域。

  CLONE_PARENT_SETTID
  内核将新创建线程的线程ID写入CLONE的第五个参数指向的位置。

  注意，使用CLONE_CHILD_SETTID在语义上是等价的，但在内核中会更昂贵。

  CLONE_CHILD_CLEARTID
  线程调用sys_exit()后，内核清除指向CLONE的第七个参数位置的线程ID。

  终止信号选择为零，这意味着不发送信号。
*/
  const int clone_flags = (CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SYSVSEM
			   | CLONE_SIGHAND | CLONE_THREAD
			   | CLONE_SETTLS | CLONE_PARENT_SETTID
			   | CLONE_CHILD_CLEARTID
			   | 0);

  TLS_DEFINE_INIT_TP (tp, pd);

  struct clone_args args =
    {
      .flags = clone_flags,
      .pidfd = (uintptr_t) &pd->tid,
      .parent_tid = (uintptr_t) &pd->tid,
      .child_tid = (uintptr_t) &pd->tid,
      .stack = (uintptr_t) stackaddr,
      .stack_size = stacksize,
      .tls = (uintptr_t) tp,
    };
  int ret = __clone_internal (&args, &start_thread, pd);
  if (__glibc_unlikely (ret == -1))
    return errno;

  /* 线程开始运行，但是如果失败，会清理好  */
  *thread_ran = true;

  /* Now we have the possibility to set scheduling parameters etc.  */
  if (attr != NULL)
    {
      /* Set the affinity mask if necessary.  */
      if (need_setaffinity)
	{
	  assert (*stopped_start);

	  int res = INTERNAL_SYSCALL_CALL (sched_setaffinity, pd->tid,
					   attr->extension->cpusetsize,
					   attr->extension->cpuset);
	  if (__glibc_unlikely (INTERNAL_SYSCALL_ERROR_P (res)))
	    return INTERNAL_SYSCALL_ERRNO (res);
	}

      /* Set the scheduling parameters.  */
      if ((attr->flags & ATTR_FLAG_NOTINHERITSCHED) != 0)
	{
	  assert (*stopped_start);

	  int res = INTERNAL_SYSCALL_CALL (sched_setscheduler, pd->tid,
					   pd->schedpolicy, &pd->schedparam);
	  if (__glibc_unlikely (INTERNAL_SYSCALL_ERROR_P (res)))
	    return INTERNAL_SYSCALL_ERRNO (res);
	}
    }

  return 0;
}

```