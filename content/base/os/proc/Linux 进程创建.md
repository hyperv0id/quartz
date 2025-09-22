## 源码

这段代码是Linux内核中用于创建一个新进程的函数，具体是`_do_fork`函数。这个函数是内核中处理`fork`系统调用的核心函数之一。

逐行解释：

1. 函数签名：`_do_fork`函数接受多个参数，包括克隆标志`clone_flags`、栈起始地址`stack_start`、栈大小`stack_size`、父进程的线程ID指针`parent_tidptr`、子进程的线程ID指针`child_tidptr`以及线程本地存储`tls`。

2. 定义局部变量：`vfork`是一个用于vfork操作的完成结构体，`pid`是新进程的PID，`p`是指向新创建的进程结构体`task_struct`的指针。

3. 检查是否需要报告跟踪事件：如果`clone_flags`中没有设置`CLONE_UNTRACED`标志，那么根据克隆类型设置跟踪事件类型`trace`。如果当前进程被跟踪，并且启用了相应的跟踪事件，则设置`trace`变量。

4. 复制进程：调用`copy_process`函数复制当前进程，创建一个新的进程结构体。如果复制失败，返回错误。

5. 添加熵：调用`add_latent_entropy`函数为内核的熵池添加熵，以提高随机数生成器的安全性。

6. 获取新进程的PID：通过`get_task_pid`函数获取新进程的PID。

7. 将子进程的PID写入父进程提供的地址：如果设置了`CLONE_PARENT_SETTID`标志，将新进程的PID写入父进程提供的地址。

8. 对于vfork操作：如果设置了`CLONE_VFORK`标志，初始化一个完成结构体`vfork`，并增加对新进程结构体的引用计数。

9. 唤醒新任务：调用`wake_up_new_task`函数唤醒新创建的进程，使其开始运行。

10. 如果设置了跟踪事件，通知跟踪器：如果`trace`变量不为0，则调用`ptrace_event_pid`函数通知跟踪器。

11. 对于vfork操作：如果设置了`CLONE_VFORK`标志，等待父进程调用`exec`或者退出，然后通知跟踪器vfork操作完成。

12. 释放PID资源：调用`put_pid`函数释放PID资源。

13. 返回新进程的PID：函数返回新创建的进程的PID。

这个函数处理了进程创建的多个方面，包括复制进程、处理跟踪事件、处理vfork特殊操作等。它是Linux内核中进程管理的一个重要组成部分。

```c
/*
 *  Ok, this is the main fork-routine.
 *
 * It copies the process, and if successful kick-starts
 * it and waits for it to finish using the VM if required.
 */
long _do_fork(unsigned long clone_flags, // 是否clone
	      unsigned long stack_start, // 桟起始
	      unsigned long stack_size, // 桟大小
	      int __user *parent_tidptr, // 父进程的线程ID指针
	      int __user *child_tidptr, // 子进程的线程ID指针
	      unsigned long tls) // 线程本地存储
{
	struct completion vfork;
	struct pid *pid;
	struct task_struct *p;
	int trace = 0;
	long nr;

	/*
	 * Determine whether and which event to report to ptracer.  When
	 * called from kernel_thread or CLONE_UNTRACED is explicitly
	 * requested, no event is reported; otherwise, report if the event
	 * for the type of forking is enabled.
	 */
	if (!(clone_flags & CLONE_UNTRACED)) {
		if (clone_flags & CLONE_VFORK)
			trace = PTRACE_EVENT_VFORK;
		else if ((clone_flags & CSIGNAL) != SIGCHLD)
			trace = PTRACE_EVENT_CLONE;
		else
			trace = PTRACE_EVENT_FORK;

		if (likely(!ptrace_event_enabled(current, trace)))
			trace = 0;
	}

	p = copy_process(clone_flags, stack_start, stack_size,
			 child_tidptr, NULL, trace, tls, NUMA_NO_NODE);
	add_latent_entropy();

	if (IS_ERR(p))
		return PTR_ERR(p);

	/*
	 * Do this prior waking up the new thread - the thread pointer
	 * might get invalid after that point, if the thread exits quickly.
	 */
	trace_sched_process_fork(current, p);

	pid = get_task_pid(p, PIDTYPE_PID);
	nr = pid_vnr(pid);

	if (clone_flags & CLONE_PARENT_SETTID)
		put_user(nr, parent_tidptr);

	if (clone_flags & CLONE_VFORK) {
		p->vfork_done = &vfork;
		init_completion(&vfork);
		get_task_struct(p);
	}

	wake_up_new_task(p);

	/* forking complete and child started to run, tell ptracer */
	if (unlikely(trace))
		ptrace_event_pid(trace, pid);

	if (clone_flags & CLONE_VFORK) {
		if (!wait_for_vfork_done(p, &vfork))
			ptrace_event_pid(PTRACE_EVENT_VFORK_DONE, pid);
	}

	put_pid(pid);
	return nr;
}
```