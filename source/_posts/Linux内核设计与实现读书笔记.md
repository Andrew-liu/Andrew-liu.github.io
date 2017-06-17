---
title: Linux内核设计与实现读书笔记
date: 2016-11-15 10:13:27
tags: Linux
desc: Linux Kernel 读书笔记
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

**Unix强大的根本原因:**

1. Unix简洁, 提供几百个系统调用, 设计目的明确
2. Unix中`所有东西都被当做文件对待`
3. Unix内核和相关系统工具是用C语言开发的, 移植能力强大
4. Unix进程创建迅速, 有独特的fork机制
5. Unix提供简单稳定的进程间通信元语

> Linux是类Unix系统, 借鉴了Unix设计并实现了Unix的API.
> 应用程序通常调用库函数(如C库函数)再由库函数通过系统调用界面, 让内核代其完成各种任务.

<!--more-->

- Linux支持动态加载内核模块
- Linux支持对称多处理(SMP)机制
- Linux为`抢占式内核`
- Linux并不区分线程和其他的一般进程
- Linux提供具有设备类的面向对象的设备模型, 热插拔事件, 以及用户控件的设备文件系统


## 中断和中断处理

> 中断是一种解决处理器和速度差异的方案, 只有在硬件需要的时候再向内核发出信号. 中断本质上是一种特殊的电信号.


- 内核响应特定中断, 然后`内核`调用特定的`中断处理程序`, 终端处理程序是设备驱动程序的一部分
- Linux中的终端处理程序是不可重入的, 同一个中断处理程序不会被同时调用
- 中断上下文不可以睡眠(我理解当前被中断的程序再中断处理结束后需要继续执行)
- 中断处理程序不在进程上下文中进行, 他们不能阻塞
- 中断处理分为两部分, 上半部为中断处理程序, 要求尽可能快的执行, 下半部(`用于减少中断处理程序的工作量`)执行与中断处理密切相关但中断处理程序本身不执行的工作
- 下半部的实现方法 `软中断、tasklet、工作队列`,

> **中断机制的实现:** 设置产生中断, 通过电信号给处理器的特定管脚发送一个信号, 处理器听着当前处理工作, `关闭中断系统`, 然后调到内存中预定义的位置(中断处理程序的入口点)开始执行.计算终端号, `do_IRQ()`对接收的中断进行应答, 禁止这条线上的中断传递.


## 内核同步

> 对于共享资源, 如果同时被多个线程访问和操作, 就可能发生各线程之间相互覆盖共享数据, 造成访问数据不一致.

同步实现通过主要`锁机制`对共享资源进行加锁, 只有持有锁的线程才能操作共享资源, 其他线程睡眠(或者轮询). 资源操作完成后, 持有锁的线程释放锁, 由等待线程抢锁.


**内核同步方法:**

1. `原子操作`
2. `自旋锁`, 特性是当线程无法获取锁, 会一直忙循环(`忙等`)等待锁重新可以, 适用于短期轻量级加锁
3. `读/写自旋锁`(共享/排它锁), 一个或多个任务可以并发的持有读者锁, 写者锁只能被一个写任务持有.
4. `信号量`(睡眠锁), 如果一个任务试图获得一个被占用的信用量时, 信号量会将其推进一个等待队列, 然后让其睡眠. 当信号量可用后, 等待队列中的任务会被唤醒. 适用于锁被长期占用的时候.
5. mutex(计数为1的信号量), 这个是编程中最常见的.
6. `顺序锁`
7. `屏障`(barriers), 用于确保指令序列和读写的执行顺序

内核中造成并发的原因:

- 中断, 几乎可以再任何时刻异步发生, 可能随时打断当前正在执行的代码
- 软中断和tasklet, 内核能在任何时刻唤醒或调度软中断或tasklet, 打断当前正在执行的代码
- 内核抢占
- 睡眠及与用户空间的同步
- 对称多处理, 多个处理器同时执行代码


## 内存管理

内核把物理页作为内存管理的基本单位, `内存管理单元(MMU, 管理内存并将虚拟地址转换为物理地址)`通常以页为单位来管理系统中的页表.

内核把也划分为不同的区(`zone`), 使用区对具有相似特性的页进行分组

```cpp
// <linux/gfp.h> 该函数分配2的order次方个连续`物理页`, 返回指针指向第一个页的page结构体
static inline struct page *
alloc_pages(gfp_t gfp_mask, unsigned int order)

// 释放物理页
extern void free_pages(unsigned long addr, unsigned int order);

//<linux/slab.h>以字节为单位分配一块内核内存(物理上连续)
static __always_inline void *kmalloc(size_t size, gfp_t flags)
//释放kmalloc分配的内存块
void kfree(const void *);
```


## 虚拟文件系统

虚拟文件系统为用户控件程序提供了文件和文件系统相关接口.

文件的元数据, 被存储在一个单独的数据结构中, 被称为`inode`(索引节点)

虚拟文件系统(VFS)有四个主要的对象模型:

- 超级块对象, 代表一个具体的已安装文件系统, 存储特定文件系统的信息
- 索引节点对象, 代表一个具体文件, 包含内核在操作文件或目录时需要的全部信息, 一个索引节点代表文件系统中的一个文件, 
- 目录项对象, 代表一个目录项, 是路径的一个组成部分, `VFS把目录当做文件处理`, 目录项对象没有对应的磁盘数据结构
- 文件对象, 代表进程打开的文件, `进程直接处理的是文件`


```cpp
// <linux/fs.h> 文件对象的数据结构
struct file {
	union {
		struct llist_node	fu_llist;
		struct rcu_head 	fu_rcuhead;
	} f_u;
	struct path		f_path;
	struct inode		*f_inode;	/* cached value */
	const struct file_operations	*f_op;

	/*
	 * Protects f_ep_links, f_flags.
	 * Must not be taken from IRQ context.
	 */
	spinlock_t		f_lock;
	atomic_long_t		f_count;
	unsigned int 		f_flags;
	fmode_t			f_mode;
	struct mutex		f_pos_lock;
	loff_t			f_pos;
	struct fown_struct	f_owner;
	const struct cred	*f_cred;
	struct file_ra_state	f_ra;

	u64			f_version;
#ifdef CONFIG_SECURITY
	void			*f_security;
#endif
	/* needed for tty driver, and maybe others */
	void			*private_data;

#ifdef CONFIG_EPOLL
	/* Used by fs/eventpoll.c to link all the hooks to this file */
	struct list_head	f_ep_links;
	struct list_head	f_tfile_llink;
#endif /* #ifdef CONFIG_EPOLL */
	struct address_space	*f_mapping;
} __attribute__((aligned(4)));	/* lest something weird decides that 2 is OK */
```

## 块I/O层

> 系统中能够`随机访问`固定大小数据片(chunks)的硬件设备称作块设备, 如硬盘. 按照字符流的方式被`有序访问`的硬件设备称为字符设备, 如键盘


```
# <linux/bio.h>I/O设备基本容器由bio结构体表示
```

- `I/O调度程序`用于管理块设备的请求队列, 决定队列中的请求排列顺序以及什么时刻派发请求到挂设备. 这样有利于减少磁盘的寻址时间, 从而提高全局的吞吐量
- linux实际使用的I/O调度程序有`linux电梯, 最终期限I/O调度, 预测I/O调度程序, 空操作的I/O调度程序`


## 进程地址空间

内核需要管理用户空间中进程的内存, 这个内存称为`进程地址空间`, 系统中所有进程之间以虚拟方式共享内存.

进程地址空间由进程可寻址的虚拟内存组成, 每个进程有32位或64位地址空间.

虚拟地址空间, 可被访问的合法地址空间称为`内存区域`:

- 可执行文件代码的内存映射, 称为代码段
- 可执行文件的已初始化全局变量的内存映射, 称为数据段
- 包含未初始化全局变量,bss(block started by symbol)段的零页的内存映射
- 用于进程用户空间栈的零页内存映射
- 每一个如C库或动态链接程序等共享库的代码段、数据段和bss会被载入进程的地址空间
- 任何内存映射文件
- 任何共享内存段
- 任何匿名的内存映射, 如malloc分配的内存

内核使用内存描述符结构体表示进程的地址空间, 内存描述符由mm_struct(`<linux/sched.h>`)结构体表示. `内核线程没有进程地址空间, 也没有相关的内存描述符, 所有内核线程没有用户上下文`


> 应用程序操作的对象是`映射到物理内存上的虚拟内存`, 而处理器操作的是物理内存, Linux使用三级页表完成地址转换, 每个虚拟地址作为索引指向页表, 页表项则指向下一级的页表. 在多级页表中通过TLB(translate lookaside buffer)作为一个虚拟地址映射到物理地址的缓存

## 参考链接

`<内核设计与实现>`
