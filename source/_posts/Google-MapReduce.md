title: Google MapReduce
date: 2016-04-02 00:27:34
tags: MapReduce
---


## 简介

- 开始喜欢看基于工程实践总结出来的paper, 而不是功利性的paper, 没有冗长难懂的数据公式和概率统计的paper.
-  并行计算，分布式数据，处理故障的问题与与大量复杂的业务代码掩盖了原来的简单的大数据处理计算。
-  MapReduce通过函数式风格代码, 自动和并行的运行在集群中. 系统的运行考虑输入数据的划分, 机器间的程序调度, 处理其中出现的错误(容错), 管理机器间通信.
- MapReduce抽象模型灵感来自于`Lisp和其他函数式编程语言`
- MapReduce的主要贡献: 通过简单又不失强大的接口, 在大规模商用集群中自动并行和分布式的处理大规模计算. MapReduce简单易用, 隐藏了底层的并行, 容错, 局部优化和负载均衡. 很多大数据处理逻辑都可以通过MapReduce来表达.

<!--more-->

## 编程模型


Map-Reduce主要思想: 
1. Map输入key-value键值对(Key-Value相当于参数), 产生中间键值对
2. Partitioning function (如 hash(key) mod R). 用于将中间结果分配到R个并行的Reduce任务中
3. Reduce输入中间键值对, 对value值做合并工作, 输出键值对

```
 map(String key, String value):
    // key: document name
    // value: document contents
    for each word w in value:
      EmitIntermediate(w, "1");  
    /*
    key-value对被发射出去, 相同Key的元素产生中间结果, 如(w, ["1", "2", "1])
     */
  reduce(String key, Iterator values):
    // key: a word
    // values: a list of counts
    int result = 0;
    for each v in values: // 对values集合进行合并
      result += ParseInt(v);
    Emit(AsString(result));
```

## 实现

![](http://ww1.sinaimg.cn/large/ab508d3djw1f2czip6x3ej21k80z67aa.jpg)



1. MapReduce库将数据划分为M片(16M~64MB一个分片, GFS), 然后在集群中做拷贝工作.
2. Master执行一个特殊的拷贝程序, 其他Woekers由Master分配任务(M个Map任务和R个Reduce任务). Master寻找空闲的Worker然后分配Map或Reduce任务
3. Map Worker读取响应的输入split. 从输入数据中解析Key-Value对然后传入用户自定义的Map函数中, Map产生的中间Key-Value对缓存在内存中.
4. 缓存的键值对被周期的写入本地磁盘, 通过划分函数(partitioning function)写入R个区域. 缓存键值对的本地磁盘位置会返回到Master中, 并由Master将这些位置传入Reduce Worker.
5. 当Master通知Reduce中间键值对的位置(传参), Reduce使用RPC(emote procedure call)从磁盘中读取Map的缓存数据. Reduce Worker读取所有中间数据后, 通过排序中间数据使用相同的Key聚集在一起(不能的Key Map可能会进入同一个Reduce Worker), 当数据过大(超出内存), 会转而使用外部排序
6. Reduce迭代已排序的Key-Value数据, 对每个唯一的key, 将key和`value集合`传入用户定义的Reduce function. Reduce函数的输出被添加到这个reduce分割的最终的输出文件中
7. 当所有Map和Reduce任务完成, Master唤醒用户程序. 在这个时候,在用户程序里的MapReduce调用返回到用户代码.


### 容错

- Worker Failure, Master周期性的进行健康检查, 当在一定时间后发现某台机器回应后(Ack机制), 重置所有在该机器上完成的Map任务为`idle state`(因为Map中间结果存储在机器本地磁盘, 检查失败, 可能导致中间结果无法访问, Reduce无需重新执行, 结果存储在全局文件系统), 此时这些任务可以被重新调度.
- Master Failure, 周期写入Master数据结构的checkpoint, 如果Master任务失效了,可以从上次最后一个checkpoint开始启动另一个master进程.


## 常用拓展组件

> 这些组件都是可以进行自定义的

1. `Partitioning Function`:  在Map产生的中间key上使用分割函数,使数据分割后进入R个Reduce任务. 默认分割函数使用hash方法(例如, hash(key) mod R).
2. `Ordering Guarantees`: 保证在一个分片中, 中间key递增有序.
3. `Combiner Function`: 用户指定一个可选的combiner函数,先在本地进行合并一下,然后再通过网络发送给Reduce. 常见于词频统计中, 单个Map任务可能产生大量重复的Key(单词)需要进行合并. 一般的, combiner和reduce函数使用相同代码. 在combiner和reduce函数之间唯一的区别是MapReduce库怎样控制函数的输出. reduce函数的输出被保存最终输出文件里.combiner函数的输出被写到中间文件里, 然后被发送给reduce任务.
4. `Input and Output Types`: MapReduce库支持以几种不同的格式读取输入数据.



> 谷歌通过MapReduce来改善大规模索引系统


- 目前比较著名的MapReduce框架有Hadoop 和 Spark

## 参考链接

- [Google Mapduce: : Simplified Data Processing on Large Clusters](http://static.googleusercontent.com/media/research.google.com/en//archive/mapreduce-osdi04.pdf)
