
内核一共包含5种调度算法：

```c
struct sched_class stop_sched_class; // 停机调度类: 停机进程
struct sched_class dl_sched_class;   // 限期调度类: 限期进程
    //SCHED_DEADLINE：限期进程调度策略，使task选择Deadline调度 器来调度运行;
struct sched_class rt_sched_class;   // 实时调度类：实时进程
    //SCHED_FIFO ： 实时进程调度策略，先进先出调度没有时间片，没有更高优先级的状态下，只有等待主动让出CPU;
    //SCHED_RR ：实时进程调度策略，时间片轮转,进程使用完时间片之后加入优先级对应运行队列当中的尾部，把CPU让给同等优先级的其它进程;
struct sched_class fair_sched_class; // 公平调度类 : 普通进程
    //SCHED_NORMAL ： 普通进程调度策略，使task选择CFS调度器来调度运行:
    //SCHED_IDLE： 普通进程调度策略，使task以最低优先级选择CFS调度器来调度运行:
struct sched_class idle_sched_class; // 空闲调度类 : 每个处理器上的空闲线程
```

## 停机调度

停机调度类是**优先级最高**的调度类，停机进程（stop-task）是优先级最高的进程，可以抢占所有其他进程，其他进程不可以抢占停机进程。停机（stop是指stop machine）的意思是使处理器停下来，做更紧急的事情。

迁移线程： 在Linux内核中，每个CPU都有一个迁移线程守护程序来执行资源平衡作业。

```c
/*
 * Simple, special scheduling class for the per-CPU stop tasks:
 */
const struct sched_class stop_sched_class = {
	.next			= &dl_sched_class,

	.enqueue_task		= enqueue_task_stop,
	.dequeue_task		= dequeue_task_stop,
	.yield_task		= yield_task_stop,

	.check_preempt_curr	= check_preempt_curr_stop,

	.pick_next_task		= pick_next_task_stop,
	.put_prev_task		= put_prev_task_stop,

#ifdef CONFIG_SMP
	.select_task_rq		= select_task_rq_stop,
	.set_cpus_allowed	= set_cpus_allowed_common,
#endif

	.set_curr_task          = set_curr_task_stop,
	.task_tick		= task_tick_stop,

	.get_rr_interval	= get_rr_interval_stop,

	.prio_changed		= prio_changed_stop,
	.switched_to		= switched_to_stop,
	.update_curr		= update_curr_stop,
};
```

目前只有迁移线程属于停机调度类，每个处理器有一个迁移线程（名称是 migration/<cpu＿id>），用来把进程从当前处理器迁移到其他处理器，迁移线程对外伪装成实时优先级是99的先进先出实时进程。

停机进程没有时间片，如果它不主动让出处理器，那么它将一直霸占处理器。

引入停机调度类的一个原因是：为了**支持限期调度**类，迁移线程的优先级必须比限期进程的优先级高，能够抢占所有其他进程，才能快速处理调度器发出的迁移请求，把进程从当前处理器迁移到其他处理器。

## 限期调度

限期调度类使用最早期限优先算法，使用**红黑树**把进程按照绝对截止期限从小到大排序，每次调度时选择**绝对截止期限最小**的进程。

如果限期进程用完了它的运行时间，它将让出处理器，并且把它从运行队列中删除。在下一个周期的开始，重新把它添加到运行队列中。

```c
const struct sched_class dl_sched_class = {
	.next			= &rt_sched_class,
	.enqueue_task		= enqueue_task_dl,
	.dequeue_task		= dequeue_task_dl,
	.yield_task		= yield_task_dl,

	.check_preempt_curr	= check_preempt_curr_dl,

	.pick_next_task		= pick_next_task_dl,
	.put_prev_task		= put_prev_task_dl,

#ifdef CONFIG_SMP
	.select_task_rq		= select_task_rq_dl,
	.migrate_task_rq	= migrate_task_rq_dl,
	.set_cpus_allowed       = set_cpus_allowed_dl,
	.rq_online              = rq_online_dl,
	.rq_offline             = rq_offline_dl,
	.task_woken		= task_woken_dl,
#endif

	.set_curr_task		= set_curr_task_dl,
	.task_tick		= task_tick_dl,
	.task_fork              = task_fork_dl,

	.prio_changed           = prio_changed_dl,
	.switched_from		= switched_from_dl,
	.switched_to		= switched_to_dl,

	.update_curr		= update_curr_dl,
};
```

有些任务必须在指定时间窗口内完成。例如视频的编码与解码，CPU 必须以特定频率完成对应的数据处理；这类任务是优先级最高的用户任务，CPU 应该首先满足。

## 实时调度类

```c
// kernel/sched/sched.h
/*
 * This is the priority-queue data structure of the RT scheduling class:
 */
struct rt_prio_array {
	DECLARE_BITMAP(bitmap, MAX_RT_PRIO+1); /* include 1 bit for delimiter */
	struct list_head queue[MAX_RT_PRIO];
};
```

位图bitmap用来快速查找第一个非空队列。数组queue的下标是实时进程的调度优先级，下标越小，优先级越高。

每次调度，先找到优先级最高的第一个非空队列，然后从队列中选择第一个进程。

使用先进先出调度策略的进程没有时间片，如果没有优先级更高的进程，并且它不主动让出处理器，那么它将一直霸占处理器。

2使用轮流调度策略的进程有时间片，用完时间片以后，进程加入队列的尾部。默认的时间片是5毫秒，可以通过文件`/proc/sys/kernel/sched＿rr＿timeslice＿ms`修改时间片。

```c
const struct sched_class rt_sched_class = {
	.next			= &fair_sched_class,
	.enqueue_task		= enqueue_task_rt,
	.dequeue_task		= dequeue_task_rt,
	.yield_task		= yield_task_rt,

	.check_preempt_curr	= check_preempt_curr_rt,

	.pick_next_task		= pick_next_task_rt,
	.put_prev_task		= put_prev_task_rt,

#ifdef CONFIG_SMP
	.select_task_rq		= select_task_rq_rt,

	.set_cpus_allowed       = set_cpus_allowed_common,
	.rq_online              = rq_online_rt,
	.rq_offline             = rq_offline_rt,
	.task_woken		= task_woken_rt,
	.switched_from		= switched_from_rt,
#endif

	.set_curr_task          = set_curr_task_rt,
	.task_tick		= task_tick_rt,

	.get_rr_interval	= get_rr_interval_rt,

	.prio_changed		= prio_changed_rt,
	.switched_to		= switched_to_rt,

	.update_curr		= update_curr_rt,
};
```

实时任务（Real-time Task）对响应时间要求更高，例如编辑器软件，它可能由于等待用户输入长期处于睡眠之中，但一旦用户有输入动作，我们就期望编辑器能够立马响应，而不是等系统完成其它任务之后才开始反应，这一点对用户体验十分重要。

## 公平调度

Fair 调度类用来调度绝大多数用户任务，CFS 实现的就是这种调度类，其核心逻辑是根据任务的优先级公平地分配 CPU 时间。

公平调度类使用完全公平调度算法。完全公平调度算法引入了虚拟运行时间的概念：

```
虚拟运行时间＝实际运行时间 * nice_0对应的权重／进程的权重
```

nice值和权重的对应关系如下：

```c
// kernel/sched/core.c
const int sched_prio_to_weight[40] = {
 /* -20 */     88761,     71755,     56483,     46273,     36291,
 /* -15 */     29154,     23254,     18705,     14949,     11916,
 /* -10 */      9548,      7620,      6100,      4904,      3906,
 /*  -5 */      3121,      2501,      1991,      1586,      1277,
 /*   0 */      1024,       820,       655,       526,       423,
 /*   5 */       335,       272,       215,       172,       137,
 /*  10 */       110,        87,        70,        56,        45,
 /*  15 */        36,        29,        23,        18,        15,
};
```



```c
// kernel/sched/fair.c
/*
 * All the scheduling class methods:
 */
const struct sched_class fair_sched_class = {
	.next			= &idle_sched_class,
	.enqueue_task		= enqueue_task_fair,
	.dequeue_task		= dequeue_task_fair,
	.yield_task		= yield_task_fair,
	.yield_to_task		= yield_to_task_fair,

	.check_preempt_curr	= check_preempt_wakeup,

	.pick_next_task		= pick_next_task_fair,
	.put_prev_task		= put_prev_task_fair,

#ifdef CONFIG_SMP
	.select_task_rq		= select_task_rq_fair,
	.migrate_task_rq	= migrate_task_rq_fair,

	.rq_online		= rq_online_fair,
	.rq_offline		= rq_offline_fair,

	.task_dead		= task_dead_fair,
	.set_cpus_allowed	= set_cpus_allowed_common,
#endif

	.set_curr_task          = set_curr_task_fair,
	.task_tick		= task_tick_fair,
	.task_fork		= task_fork_fair,

	.prio_changed		= prio_changed_fair,
	.switched_from		= switched_from_fair,
	.switched_to		= switched_to_fair,

	.get_rr_interval	= get_rr_interval_fair,

	.update_curr		= update_curr_fair,

#ifdef CONFIG_FAIR_GROUP_SCHED
	.task_change_group	= task_change_group_fair,
#endif
};
```



## 空闲调度

与 Stop 类似，Idle 调度类也是仅供内核使用的特殊调度类，其优先级最低，只有在没有任何用户任务时才会用到。内核会为每个 CPU 绑定一个内核线程`kthread`来完成该任务，该线程会在队列无事可做的情况下启动该任务，并将 CPU 的功耗降到最低。

```c
// kernel/sched/idle.c
/*
 * Simple, special scheduling class for the per-CPU idle tasks:
 */
const struct sched_class idle_sched_class = {
	/* .next is NULL */
	/* no enqueue/yield_task for idle tasks */

	/* dequeue is not valid, we print a debug message there: */
	.dequeue_task		= dequeue_task_idle,

	.check_preempt_curr	= check_preempt_curr_idle,

	.pick_next_task		= pick_next_task_idle,
	.put_prev_task		= put_prev_task_idle,

#ifdef CONFIG_SMP
	.select_task_rq		= select_task_rq_idle,
	.set_cpus_allowed	= set_cpus_allowed_common,
#endif

	.set_curr_task          = set_curr_task_idle,
	.task_tick		= task_tick_idle,

	.get_rr_interval	= get_rr_interval_idle,

	.prio_changed		= prio_changed_idle,
	.switched_to		= switched_to_idle,
	.update_curr		= update_curr_idle,
};
```



## Reference



1. https://s3.shizhz.me/linux-sched/concepts/sched-class
2. https://hv1oua0ifoy.feishu.cn/docx/Vx2MdYlOdo1vWAxAlk3cpxI8nGh