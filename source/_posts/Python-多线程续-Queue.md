title: Python爬虫(六)--多线程续(Queue)
date: 2014-12-15 14:21:18
tags: Python
---

---

本文希望达到的目标:

1. 学习Queue模块
2. 将Queue模块与多线程编程相结合
3. 通过Queue和threading模块, 重构爬虫, 实现多线程爬虫, 
4. 通过以上学习希望总结出一个通用的多线程爬虫小模版


#1. Queue模块
---

`Queue`模块实现了多生产者多消费者队列, 尤其适合多线程编程.`Queue`类中`实现了所有需要的锁原语`(这句话非常重要), Queue模块实现了三种类型队列:

- FIFO(先进先出)队列, 第一加入队列的任务, 被第一个取出
- LIFO(后进先出)队列,最后加入队列的任务, 被第一个取出(操作类似与栈, 总是从栈顶取出, 这个队列还不清楚内部的实现)
- PriorityQueue(优先级)队列, 保持队列数据有序, 最小值被先取出(在C++中我记得优先级队列是可以自己重写排序规则的, Python不知道可以吗)


<!--more-->

##1.1. 类和异常

```python
import Queue

#类
Queue.Queue(maxsize = 0)  #构造一个FIFO队列,maxsize设置队列大小的上界, 如果插入数据时, 达到上界会发生阻塞, 直到队列可以放入数据. 当maxsize小于或者等于0, 表示不限制队列的大小(默认)

Queue.LifoQueue(maxsize = 0)  #构造一LIFO队列,maxsize设置队列大小的上界, 如果插入数据时, 达到上界会发生阻塞, 直到队列可以放入数据. 当maxsize小于或者等于0, 表示不限制队列的大小(默认)

Queue.PriorityQueue(maxsize = 0)  #构造一个优先级队列,,maxsize设置队列大小的上界, 如果插入数据时, 达到上界会发生阻塞, 直到队列可以放入数据. 当maxsize小于或者等于0, 表示不限制队列的大小(默认). 优先级队列中, 最小值被最先取出

#异常
Queue.Empty  #当调用非阻塞的get()获取空队列的元素时, 引发异常
Queue.Full  #当调用非阻塞的put()向满队列中添加元素时, 引发异常
```

##1.2. Queue对象

三种队列对象`提供公共的方法`

```
Queue.empty()  #如果队列为空, 返回True(注意队列为空时, 并不能保证调用put()不会阻塞); 队列不空返回False(不空时, 不能保证调用get()不会阻塞)
Queue.full()  #如果队列为满, 返回True(不能保证调用get()不会阻塞), 如果队列不满, 返回False(并不能保证调用put()不会阻塞)

Queue.put(item[, block[, timeout]])  #向队列中放入元素, 如果可选参数block为True并且timeout参数为None(默认), 为阻塞型put(). 如果timeout是正数, 会阻塞timeout时间并引发Queue.Full异常. 如果block为False为非阻塞put
Queue.put_nowait(item)  #等价于put(itme, False)

Queue.get([block[, timeout]])  #移除列队元素并将元素返回, block = True为阻塞函数, block = False为非阻塞函数. 可能返回Queue.Empty异常
Queue.get_nowait()  #等价于get(False)

Queue.task_done()  #在完成一项工作之后，Queue.task_done()函数向任务已经完成的队列发送一个信号
Queue.join()  #实际上意味着等到队列为空，再执行别的操作
```

下面是官方文档给多出的多线程模型(官方文档果然是个好东西):


```python
def worker():
    while True:
        item = q.get()
        do_work(item)
        q.task_done()

q = Queue()
for i in range(num_worker_threads):
     t = Thread(target=worker)
     t.daemon = True
     t.start()

for item in source():
    q.put(item)

q.join()       # block until all tasks are done
```

#2. Queue模块与线程相结合
---

简单写了一个Queue和线程结合的小程序

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import threading
import time
import Queue

SHARE_Q = Queue.Queue()  #构造一个不限制大小的的队列
_WORKER_THREAD_NUM = 3   #设置线程个数

class MyThread(threading.Thread) :

    def __init__(self, func) :
        super(MyThread, self).__init__()
        self.func = func

    def run(self) :
        self.func()

def worker() :
    global SHARE_Q
    while not SHARE_Q.empty():
        item = SHARE_Q.get() #获得任务
        print "Processing : ", item
        time.sleep(1)

def main() :
    global SHARE_Q
    threads = []
    for task in xrange(5) :  #向队列中放入任务
        SHARE_Q.put(task)
    for i in xrange(_WORKER_THREAD_NUM) :
        thread = MyThread(worker)
        thread.start()
        threads.append(thread)
    for thread in threads :
        thread.join()

if __name__ == '__main__':
    main()
```



#3. 重构爬虫
---

主要针对之间写过的豆瓣爬虫进行重构:
- [Python网络爬虫(二)--豆瓣抓站小计](http://andrewliu.tk/2014/12/05/Python%E7%BD%91%E7%BB%9C%E7%88%AC%E8%99%AB-%E4%BA%8C-%E8%B1%86%E7%93%A3%E6%8A%93%E7%AB%99%E5%B0%8F%E8%AE%A1/)



##3.1. 豆瓣电影爬虫重构

通过对Queue和线程模型进行改写, 可以写出下面的爬虫程序 :

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# 多线程爬取豆瓣Top250的电影名称

import urllib2, re, string
import threading, Queue, time
import sys

reload(sys)
sys.setdefaultencoding('utf8')
_DATA = []
FILE_LOCK = threading.Lock()
SHARE_Q = Queue.Queue()  #构造一个不限制大小的的队列
_WORKER_THREAD_NUM = 3  #设置线程的个数

class MyThread(threading.Thread) :

    def __init__(self, func) :
        super(MyThread, self).__init__()  #调用父类的构造函数
        self.func = func  #传入线程函数逻辑

    def run(self) :
        self.func()

def worker() :
    global SHARE_Q
    while not SHARE_Q.empty():
        url = SHARE_Q.get() #获得任务
        my_page = get_page(url)  #爬取整个网页的HTML代码
        find_title(my_page)  #获得当前页面的电影名
        time.sleep(1)
        SHARE_Q.task_done()
```

完整代码请查看[Github豆瓣多线程爬虫](https://github.com/Andrew-liu/dou_ban_spider/blob/master/threading_douban.py)
完成这个程序后, 又出现了新的问题:

> 无法保证数据的顺序性, 因为线程是并发的, 思考的方法是: 设置一个主线程进行管理, 然后他们的线程工作





#4. 通用的多线程爬虫小模版
---

下面是根据上面的爬虫做了点小改动后形成的模板

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import threading
import time
import Queue

SHARE_Q = Queue.Queue()  #构造一个不限制大小的的队列
_WORKER_THREAD_NUM = 3  #设置线程的个数

class MyThread(threading.Thread) :
    """
    
    doc of class
    
    Attributess:
        func: 线程函数逻辑
    """
    def __init__(self, func) :
        super(MyThread, self).__init__()  #调用父类的构造函数
        self.func = func  #传入线程函数逻辑

    def run(self) :
        """
        重写基类的run方法
        
        """
        self.func()

def do_something(item) :
    """
    运行逻辑, 比如抓站
    """
    print item

def worker() :
    """
    主要用来写工作逻辑, 只要队列不空持续处理
    队列为空时, 检查队列, 由于Queue中已经包含了wait,
    notify和锁, 所以不需要在取任务或者放任务的时候加锁解锁
    """
    global SHARE_Q
    while True : 
        if not SHARE_Q.empty():
            item = SHARE_Q.get() #获得任务
            do_something(item)
            time.sleep(1)
            SHARE_Q.task_done()


def main() :
    global SHARE_Q
    threads = []
    #向队列中放入任务, 真正使用时, 应该设置为可持续的放入任务
    for task in xrange(5) :   
        SHARE_Q.put(task)
    #开启_WORKER_THREAD_NUM个线程
    for i in xrange(_WORKER_THREAD_NUM) :
        thread = MyThread(worker)
        thread.start()  #线程开始处理任务
        threads.append(thread)
    for thread in threads :
        thread.join()
    #等待所有任务完成
    SHARE_Q.join()

if __name__ == '__main__':
    main()
```

#5. 思考更高效的爬虫方法
---

- 使用[twisted](https://twistedmatrix.com/trac/wiki/Documentation)进行异步IO抓取
- 使用`Scrapy`框架(Scrapy 使用了 Twisted 异步网络库来处理网络通讯)

http://krondo.com/blog/?page_id=1327
http://turtlerbender007.appspot.com/twisted/index.html

