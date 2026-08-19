title: "6.824 Lab 1: MapReduce(2016)"
date: 2016-04-16 13:22:51
tags: Go
---


本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


## MapReduce框架简介

1. 应用提供大量输入文件, 一个Map函数, 一个Reduce函数, nReduce个`reduce tasks`.
2. RPC服务器启动, 等该worker注册(Register)RPC中. 任务状态为可执行时, 调度器(Schedule)决定如何把task分配给worker和如何容错.
3. Master把每个输入文件作为一个Map任务, 每个任务至少调用一次doMap. 这些任务或者直接执行或者由DoTask RPC发射, 每个对`doMap()`的调用读取合适的文件, 对每个文件调用map, 对每个map文件最后产生nReduce个文件. 
4. Master对每个reduce人物至少调用一次`doReduce()`. doReduce()收集由`map`产生的nReduce文件(f-*-), 然后对这些文件运行reduce函数. 最终产生nReduce个结果文件.
5. Master调用`mr.merge()`, 合并所有的结果文件为一个输出文件.
6. Master发送`Shutdown RPC`关闭workers, 然后关闭RPC服务器.

<!--more-->

> **Lab1总共分为五部分**

## Part I: Map/Reduce input and output

作业: 实现DoMap()(`mapreduce/common_map.go`)和DoReduce()(`mapreduce/common_reduce.go`)两个函数

DoMap功能简述:

1. 通过`inFile`文件名读取文件中的内容, 将内容传入Map函数中, 返回得到`Key-value对数组`
2. 将Key-value对数组通过Split函数(此处Split函数为`ihash`), 平均分配到`nReduce`个中间文件中, nReduce名字可以通过`reduceName`构造出来. 其中, Key-Value写入文件使用了Json进行序列化(将Key-value数据结构序列化为字符串).

DoReduce功能简述:

1. Map生成`nReduce`个中间文件后, DoReduce遍历读取这些中间文件, 通过序列化器拿出所有的Key-Value对, 然后将Key-value放入新的数据结构(该数据结构使key相同的value值合并, 自然想到使用Map数据结构)
2. 对所有的key进行排序, 生成有序的key数组. 然后对`key-values`(注意此处key对应的值为一个数组)进行Reduce操作, 并将结果写入新的文件(文件名由`mergeName`获得).

> 运行`mapreduce/test_test.go`时需要注意, 只运行`TestSequentialMany, TestSequentialSingle`两个测试用例, 将其他三个测试用例注释掉.


## Part II: Single-worker word count


`Part I` 完成后, `Part II` 没什么难度了, 一个简单的Map-Reduce词频统计的任务, 对输入文件内容进行分词, 然后将词发射出去(词频默认为1), Reduce将values进行加和就完成了.


## Part III: Distributing MapReduce tasks

- 实现分布式MapReduce的调度模块.

整体流程: 启动一个Master RPC服务器, 服务器调用`schedule`来调用Map/Reduce任务; 启动多个Worker RPC服务器, 并将Worker的端口信息注册到Master服务器的数据结构(`channel`)中

- `schedule`: 负责整个MapReduce任务的调度, 查找当前可用Worker, 然后通过worker来执行Map/Reduce任务

注意事项: 应该保证`schedule`中所有的goroutine全部完成后才能返回. 所以应该使函数阻塞直到所有的goroutine完成. 

**Go中有两种方法可以保证进行同步程序**:
参考文章: [Go goroutine同步](http://andrewliu.in/2016/04/08/Go-goroutine%E5%90%8C%E6%AD%A5/)


## Part IV: Handling worker failures

Master来处理失败的`workers`, 当某worker上的Map/Reduce任务失败后, 需要将这个任务转移给其他worker来执行.

在设计调度任务函数`schedule()`的时候考虑`容错性`, 判断在Worker上调用RPC是否成功, 若失败则重新分配一个新的worker服务器来处理task. 整个容错逻辑可以放到一个for循环中, 只有当任务成功调用才`break`跳出循环.


## Part V: Inverted index generation

**构建一个倒排索引Map/Reduce任务**

- **Map任务**:

拿到一个网页url和url对应网页文本, 对网页文本进行分词, 将每个词作为key, 网页url作为value发射出去

- **Reduce任务**

拿到一个关键词key, 和关键词对应的url集合, 首先对url进行去重(可能一个url中出现多次关键词), 然后对url进行排序(可以不排序), 最后根据需要的结构对整个url集合作拼接(`url集合的长度即为url中出现关键词的url个数`), 最后将关键词和拼接字符串发射出去


## Lab1源码

个人完成Lab1的代码托管在Github上, 仅供参考, 第一次使用Go语言, 有些不太熟悉的地方, 若有更好的方法欢迎指导.

[MIT 6.824 2016 Lab1 using go language](https://github.com/Andrew-liu/distributed-system)


## 参考链接

- [6.824 Lab 1: MapReduce](https://pdos.csail.mit.edu/6.824/labs/lab-1.html)
