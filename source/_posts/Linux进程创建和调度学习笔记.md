---
title: Linux进程创建和调度学习笔记
date: 2016-10-08 20:45:56
tags: Linux
desc: Linux 进程创建 进程调度
---

> 读书笔记

## 进程管理



> 进程是处于执行期的程序, 包含代码段, 打开描述符, 挂起信号, 内核内部数据, 处理器状态, 一个或多个具有内存映射的内存地址空间及一个或多个执行线程.
> 线程是进程中活动对象, 包含`独立`的程序计数器, 栈和一组进程寄存器. `线程间可共享虚拟内存, 但每个都拥有各自的虚拟处理器`

- 内核将进程的列表放在一个双向循环链表中, 每项为`task_struct`
- 进程执行系统调用或者异常处理才会陷入内核空间(内核态)
- Linux所有进程都是PID为1的init进程的后代

<!--more-->

每个进程或线程都有三个数据结构，分别是 `struct thread_info`, `struct task_struct` 和 `内核栈`

**task_struct 结构体中的主要元素**:

- struct thread_info *thread_info。thread_info 指向该进程/线程的基本信息。
- struct mm_struct *mm。mm_struct 对象用来管理该进程/线程的页表以及虚拟内存区。
- struct mm_struct *active_mm。主要用于内核线程访问主内核页全局目录。
- struct fs_struct *fs。fs_struct 是关于文件系统的对象。
- struct files_struct *files。files_struct 是关于打开的文件的对象。
- struct signal_struct *signal。signal_struct 是关于信号的对象。

### 进程创建

通过`fork和exec`来实现, fork() 拷贝当前进程创建一个子进程, exec() 负责读取可执行文件并将其载入地址空间开始执行.

- Linux的 fork() 使用写时复制(copy-on-write)页实现, 资源的复制只有在需要写入的时候才进行. `fork() 的实际开销是复制父进程的页表以及给子进程创建唯一的进程描述符`
- Linux通过 `Clone()` 实现 `fork()`. `fork() -> clone(SIGCHLD) -> do_fork() -> copy_process()`

copy_process()的工作:

1. 调用dup_task_struct为新进程创建一个内核栈, thread_info结果和task_struct, 值保持与当前进程相同
2. 检查当前用户进程数未超出给定分配资源的限制
3. 子进程部分信息被清0或重设为初始值
4. 子进程状态被设置为 TASK_UNINTERRUPTIBLE, 保证他不会投入运行
5. 更新 task_struct的flags成员
6. 调用 alloc_pid为新进程分配有效的`PID`(此时才分配新pid)
7. 根据传递给clone的参数, 设置拷贝或共享的资源.
8. 返回一个指向子进程的指针.

> vfork 与 fork 的区别是, vfork不拷贝父进程的页表项


### 线程

- 线程在Linux被视为一个与其他进程共享某些资源的进程

```c
# 线程同样通过clone创建, 只是指定共享的资源不同
clone(CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGHAND, 0);
```

### 内核线程

- 内核线程用于完成内核需要在后台执行的一些操作
- 内核线程和普通进程的区别在于内核线程`没有独立的地址空间`, 只在内核空间运行, **所有的内核线程共享内核地址空间**
- 内核线程可以被调度, 可以被抢占
- 内核线程只能由其他内核线程创建

```c
# <linux/kthread.h>, 内核线程创建
struct task_struct *kthread_create_on_node(int (*threadfn)(void *data),
					   void *data,
					   int node,
					   const char namefmt[], ...);

#define kthread_create(threadfn, data, namefmt, arg...) \
	kthread_create_on_node(threadfn, data, NUMA_NO_NODE, namefmt, ##arg)

# 创建并运行, kthread_create创建, wake_up_process唤醒
#define kthread_run(threadfn, data, namefmt, ...)			   \
({									   \
	struct task_struct *__k						   \
		= kthread_create(threadfn, data, namefmt, ## __VA_ARGS__); \
	if (!IS_ERR(__k))						   \
		wake_up_process(__k);					   \
	__k;								   \
})
```

### 进程终止

进程终止一般是调用 `exit()`, 最终基本是通过 `do_exit()`完成.

1. task_struct的标志成员设置为 PF_EXITING
2. 调用del_timer_sync()删除任一内核定时器
3. 调用exit_mm()函数释放进程占用的 mm_struct
4. 调用sem_exit()函数
5. 调用exit_file()和exit_fs() 分别递减文件描述符, 文件系统数据的引用计数
6. 执行一些其他的退出动作
7. 调用exit_notify()向父进程发送信号, 为其子进程找继父(线程组其他线程或init进程). 进程状态设置为`EXIT_ZOMBIE`
8. 调度schedule()切换到新的进程. do_exit() 永不返回.

> 进程终止的清理工作和进程描述符的删除是分开执行的. 进程描述符的删除由`wait4()系统调用`完成. 两步完成所有资源才释放完成.

## 进程调度

> 进程调度程序可以看做在可运行态进程之间分配有限的处理器时间资源的内核子系统. Linux在2.6.23内核版本中使用了`完全公平调度算法(CFS)`

- 进程分为I/O消耗型和处理器消耗型
- Linux调度器以模块方式提供, 允许多种不同的可动态添加的调度算法并存.
- Linux使用`完全公平调度(CFS)`, 允许每个进程运行一段时间, 循环轮转, 选择运行最少的进程作为下一个运行进程(不采用通过nice计算并分配给进程时间片做法), 分配给进程的是一个处理的使用比重. nice值在CFS中被作为进程获得处理器运行比的权重. 高nice值获得更低的处理器权重. 为防止可运行进程过趋于无限时,  进程各自获得处理器使用比趋于0, CFS为每个进程设置一个`最小粒度`
- CFS使用红黑树组织可运行进程队列, 选择可运行进程中最小`vruntime(进程虚拟运行时间)`的任务. 进程可运行或被fork()后被加入红黑树
- Linux还提供两种实时调度策略(均为静态优先级) `SCHED_FIFO和SCHED_RR`由特殊的实时调度器管理. FIFO就是先入先出的调度策略. RR是一种带时间片的先入先出的调度策略

```c
# <linux/sched.h> struct sched_entity对进程运行时间做记录
struct sched_entity {
	struct load_weight	load;		/* for load-balancing */
	struct rb_node		run_node;
	struct list_head	group_node;
	unsigned int		on_rq;

	u64			exec_start;
	u64			sum_exec_runtime;
	u64			vruntime;
	u64			prev_sum_exec_runtime;

	u64			nr_migrations;

#ifdef CONFIG_SCHEDSTATS
	struct sched_statistics statistics;
#endif

#ifdef CONFIG_FAIR_GROUP_SCHED
	int			depth;
	struct sched_entity	*parent;
	/* rq on which this entity is (to be) queued: */
	struct cfs_rq		*cfs_rq;
	/* rq "owned" by this entity/group: */
	struct cfs_rq		*my_q;
#endif

#ifdef CONFIG_SMP
	/*
	 * Per entity load average tracking.
	 *
	 * Put into separate cache line so it does not
	 * collide with read-mostly values above.
	 */
	struct sched_avg	avg ____cacheline_aligned_in_smp;
#endif
};
```

> Linux优先级分两种. nice值(取值-20到+19, nice值最大优先级越低); 实时优先级(取值0到99, 实时优先级数值月大优先级越高)


### 上下文切换

`schedule()`执行进程调度时, 调用kernel/sched.c中的`context_switch()`函数执行:

- 调用声明`asm/mmu_context.h`中的switch_mm(), 该函数把虚拟内存从上一个进程切换到新的进程中.
- 调用`<asm/system.h>`中的`switch_to()`, 该函数负责从上一个进程的处理器状态切换到新进程的处理器状态. 包括保存、回复栈信息和寄存器信息

## 参考链接

- Linux内核设计与实现 - 原书第3版

