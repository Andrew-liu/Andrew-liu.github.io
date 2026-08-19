title: Python爬虫(五)--多线程
date: 2014-12-14 13:59:46
tags: Python
---


---



#1. thread模块
---

- python是支持多线程的, 主要是通过thread和threading这两个模块来实现的。
- python的thread模块是比较底层的模块(或者说轻量级)，python的threading模块是对thread做了一些包装的，可以更加方便的被使用。




简要的看一下thread模块中含函数和常量

```python
import thread

thread.LockType  #锁对象的一种, 用于线程的同步
thread.error  #线程的异常

thread.start_new_thread(function, args[, kwargs])  #创建一个新的线程
function : 线程执行函数
args : 线程执行函数的参数, 类似为tuple,
kwargs : 是一个字典
返回值: 返回线程的标识符

thread.exit()  #线程退出函数
thread.allocate_lock()  #生成一个未锁状态的锁对象
返回值: 返回一个锁对象
```

`锁对象`的方法

```python
lock.acquire([waitflag]) #获取锁
无参数时, 无条件获取锁, 无法获取时, 会被阻塞, 知道可以锁被释放
有参数时, waitflag = 0 时,表示只有在不需要等待的情况下才获取锁, 非零情况与上面相同
返回值 :　获得锁成功返回True, 获得锁失败返回False

lock.release() #释放锁

lock.locked() #获取当前锁的状态
返回值 : 如果锁已经被某个线程获取,返回True, 否则为False
```

<!--more-->


#1.1. thread多线程

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import thread
import time

def print_time(thread_name, delay) :
    count = 0
    while count < 5 :
        time.sleep(delay)
        count += 1
        print "%s : %s" % (thread_name, time.ctime(time.time()))

try :
    thread.start_new_thread(print_time, ("Thread-1", 2, ))
    thread.start_new_thread(print_time, ("Thread-2", 4, ))
except : 
    print "Error: unable to start the thread"

while True :
    pass
```


#2. threading模块
---

> python的threading模块是对thread做了一些包装的，可以更加方便的被使用。经常和[Queue](https://docs.python.org/2/library/queue.html)结合使用,Queue模块中提供了同步的、线程安全的队列类，包括`FIFO（先入先出)队列Queue`，`LIFO（后入先出）队列LifoQueue`，和`优先级队列PriorityQueue`。这些队列都实现了`锁原语`，能够在多线程中直接使用。可以使用队列来实现线程间的同步


##2.1. 常用函数和对象

```python
#函数
threading.active_count()  #返回当前线程对象Thread的个数
threading.enumerate()  #返回当前运行的线程对象Thread(包括后台的)的list
threading.Condition()  #返回条件变量对象的工厂函数, 主要用户线程的并发
threading.current_thread()  #返回当前的线程对象Thread, 文档后面解释没看懂
threading.Lock()  #返回一个新的锁对象, 是在thread模块的基础上实现的 与acquire()和release()结合使用

#类
threading.Thread  #一个表示线程控制的类, 这个类常被继承
thraeding.Timer  #定时器,线程在一定时间后执行
threading.ThreadError  #引发中各种线程相关异常
```

###2.1.1. Thread对象


> 一般来说，使用线程有两种模式, 一种是创建线程要执行的函数, 把这个函数传递进Thread对象里，让它来执行. 另一种是直接从Thread继承，创建一个新的class，把线程执行的代码放到这个新的class里

常用两种方式运行线程(线程中包含name属性) :

- 在构造函数中传入用于线程运行的函数(这种方式更加灵活)
- 在子类中重写threading.Thread基类中run()方法(`只重写__init__()和run()方法`)

创建线程对象后, 通过调用start()函数运行线程,  然后会自动调用`run()`方法.

>　通过设置｀daemon｀属性, 可以将线程设置为守护线程

```python
threading.Thread(group = None, target = None, name = None, args = () kwars = {})
group : 应该为None
target : 可以传入一个函数用于run()方法调用,
name : 线程名 默认使用"Thread-N"
args : 元组, 表示传入target函数的参数
kwargs : 字典, 传入target函数中关键字参数

属性:
name  #线程表示, 没有任何语义
doemon  #布尔值, 如果是守护线程为True, 不是为False, 主线程不是守护线程, 默认threading.Thread.damon = False

类方法: 
run()  #用以表示线程活动的方法。
start()  #启动线程活动。
join([time])  #等待至线程中止。这阻塞调用线程直至线程的join() 方法被调用中止-正常退出或者抛出未处理的异常-或者是可选的超时发生。
isAlive(): 返回线程是否活动的。
getName(): 返回线程名。
setName(): 设置线程名。
```

范例: 

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import threading
import time

def test_thread(count) :
    while count > 0 :
        print "count = %d" % count
        count = count - 1
        time.sleep(1)

def main() :
    my_thread = threading.Thread(target = test_thread, args = (10, ))
    my_thread.start()
    my_thread.join()

if __name__ == '__main__':
    main()
```

##2.2. 常用多线程写法

- 固定线程运行的函数

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import threading, thread
import time


class MyThread(threading.Thread):
    """docstring for MyThread"""

    def __init__(self, thread_id, name, counter) :
        super(MyThread, self).__init__()  #调用父类的构造函数 
        self.thread_id = thread_id
        self.name = name
        self.counter = counter

    def run(self) :
        print "Starting " + self.name
        print_time(self.name, self.counter, 5)
        print "Exiting " + self.name

def print_time(thread_name, delay, counter) :
    while counter :
        time.sleep(delay)
        print "%s %s" % (thread_name, time.ctime(time.time()))
        counter -= 1

def main():
    #创建新的线程
    thread1 = MyThread(1, "Thread-1", 1)
    thread2 = MyThread(2, "Thread-2", 2)

    #开启线程
    thread1.start()
    thread2.start()


    thread1.join()
    thread2.join()
    print "Exiting Main Thread"

if __name__ == '__main__':
    main()
```

- 外部传入线程运行的函数

```python
#/usr/bin/env python
# -*- coding: utf-8 -*-
import threading
import time

class MyThread(threading.Thread):
    """
    属性:
    target: 传入外部函数, 用户线程调用
    args: 函数参数
    """
    def __init__(self, target, args):
        super(MyThread, self).__init__()  #调用父类的构造函数 
        self.target = target
        self.args = args

    def run(self) :
        self.target(self.args)

def print_time(counter) :
    while counter :
        print "counter = %d" % counter
        counter -= 1
        time.sleep(1)

def main() :
    my_thread = MyThread(print_time, 10)
    my_thread.start()
    my_thread.join()

if __name__ == '__main__':
    main()
```

##2.3. 生产者消费者问题


> 试着用python写了一个生产者消费者问题(伪生产者消费者), 只是使用简单的锁, 感觉有点不太对, 下面另一个程序会写出正确的生产者消费者问题

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import thread, threading
import urllib2
import time, random
import Queue

share_queue = Queue.Queue()  #共享队列
my_lock = thread.allocate_lock()
class Producer(threading.Thread) :

    def run(self) :
        products = range(5)
        global share_queue
        while True :
            num = random.choice(products)
            my_lock.acquire()
            share_queue.put(num)
            print  "Produce : ", num
            my_lock.release()
            time.sleep(random.random())

class Consumer(threading.Thread) :

    def run(self) :
        global share_queue
        while True:
            my_lock.acquire()
            if share_queue.empty() : #这里没有使用信号量机制进行阻塞等待, 
                print "Queue is Empty..."  
                my_lock.release()
                time.sleep(random.random())
                continue
            num = share_queue.get()
            print "Consumer : ", num
            my_lock.release()
            time.sleep(random.random())

def main() :
    producer = Producer()
    consumer = Consumer()
    producer.start()
    consumer.start()

if __name__ == '__main__':
    main()
```

杀死多线程程序方法: 使用`control + z`挂起程序(程序依然在后台, 可以使用`ps aux`查看), 获得程序的进程号, 然后使用`kill -9 进程号`杀死进程


> 参考一篇帖子解决了上述问题,重写了生产者消费者问题程序, 参考链接惯例放在最后.

使用了wait()和notify()解决

> 当然最简答的方法是直接使用`Queue`,Queue封装了Condition的行为, 如wait(), notify(), acquire(), 没看文档就这样, 使用了Queue竟然不知道封装了这些函数, 继续滚去看文档了

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import threading
import random, time, Queue

MAX_SIZE = 5
SHARE_Q = []  #模拟共享队列
CONDITION = threading.Condition()

class Producer(threading.Thread) :

    def run(self) :
        products = range(5)
        global SHARE_Q
        while True :
            CONDITION.acquire()
            if len(SHARE_Q) == 5 :
                print "Queue is full.."
                CONDITION.wait()
                print "Consumer have comsumed something"
            product = random.choice(products)
            SHARE_Q.append(product)
            print "Producer : ", product
            CONDITION.notify()
            CONDITION.release()
            time.sleep(random.random())

class Consumer(threading.Thread) :

    def run(self) :
        global SHARE_Q
        while True:
            CONDITION.acquire()
            if not SHARE_Q :
                print "Queue is Empty..."
                CONDITION.wait()
                print "Producer have producted something"
            product = SHARE_Q.pop(0)
            print "Consumer :", product
            CONDITION.notify()
            CONDITION.release()
            time.sleep(random.random())

def main() :
    producer = Producer()
    consumer = Consumer()
    producer.start()
    consumer.start()

if __name__ == '__main__':
    main()
```

##2.4.简单锁

> 如果只是简单的加锁解锁可以直接使用threading.Lock()生成锁对象, 然后使用acquire()和release()方法

例如:

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*- 

import threading
import time

class MyThread(threading.Thread) :
    
    def __init__(self, thread_id, name, counter) :
        threading.Thread.__init__(self)
        self.thread_id = thread_id
        self.name = name
        self.counter = counter

    def run(self) :
        #重写run方法, 添加线程执行逻辑, start函数运行会自动执行
        print  "Starting " + self.name
        threadLock.acquire() #获取所
        print_time(self.name, self.counter, 3)
        threadLock.release() #释放锁

def print_time(thread_name, delay, counter) :
    while counter :
        time.sleep(delay)
        print "%s %s" % (thread_name, time.ctime(time.time()))
        counter -= 1

threadLock = threading.Lock()
threads = [] #存放线程对象

thread1 = MyThread(1, "Thread-1", 1)
thread2 = MyThread(2, "Thread-2", 2)

#开启线程
thread1.start()
thread2.start()

for t in threads :
    t.join()  #等待线程直到终止
print "Exiting Main Thread"
```

##2.5. Condition

> 如果是向生产者消费者类似的情形, 使用Condition类 或者直接使用`Queue`模块

**Condition**

条件变量中有`acquire()和release方法用来调用锁的方法`, 有`wait(), notify(), notifyAll()方法`, 后面是三个方法必须在获取锁的情况下调用, 否则产生`RuntimeError`错误.

- 当一个线程获得锁后, 发现没有期望的资源或者状态, 就会调用wait()阻塞, 并释放已经获得锁, 知道期望的资源或者状态发生改变
- 当一个线程获得锁, 改变了资源或者状态, 就会调用notify()和notifyAll()去通知其他线程, 

```python
#官方文档中提供的生产者消费者模型
# Consume one item
cv.acquire()
while not an_item_is_available():
    cv.wait()
get_an_available_item()
cv.release()

# Produce one item
cv.acquire()
make_an_item_available()
cv.notify()
cv.release()
```

```
#threading.Condition类
thread.Condition([lock])
可选参数lock: 必须是Lock或者RLock对象, 并被作为underlying锁(悲观锁?), 否则, 会创建一个新的RLock对象作为underlying锁

类方法:
acquire()  #获得锁
release()  #释放锁
wait([timeout])  #持续等待直到被notify()或者notifyAll()通知或者超时(必须先获得锁),
#wait()所做操作, 先释放获得的锁, 然后阻塞, 知道被notify或者notifyAll唤醒或者超时, 一旦被唤醒或者超时, 会重新获取锁(应该说抢锁), 然后返回
notify()  #唤醒一个wait()阻塞的线程.
notify_all()或者notifyAll()  #唤醒所有阻塞的线程
```

> `参考程序可以查看上面的生产者消费者程序`


#3. 参考链接
---

- [Python中的生产者消费者问题](http://blog.jobbole.com/52412/)
- [Python多线程](http://www.w3cschool.cc/python/python-multithreading.html)
- [threading官方文档](https://docs.python.org/2/library/threading.html)
- [thread官方文档](https://docs.python.org/2/library/thread.html#module-thread)
