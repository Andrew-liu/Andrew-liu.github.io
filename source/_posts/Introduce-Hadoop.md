title: Introduce Hadoop
date: 2015-03-08 09:21:32
tags: MapReduce
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


#1. Hadoop简介
---

> 由于几乎所有的书中都会提到Hadoop的发展史, 这里就不说Hadoop的历史时间线了.

`Hadoop是由Apache软件基金会开发的开源分布式计算平台`, `通过Hadoop分布式文件系统和MapReduce为核心为用户提供分布式基础框架`


- HBFS是以一种分布式文件系统, 具有搞容错性的特点, 可以设计部署到低廉的硬件上, 适用于超大规模数据集的应用程序.
- MapReduce是一种编程模型, 用于大规模数据集的并行运算, `Map(映射)将输入键值对映射成一组新的键值对,Reduce(规约)对相同key下所有value进行处理后输出最终键值对`

<!--more-->

Hadoop生态包括MapReduce, HDFS, ZooKeeper, Common, Avro, Chukwa, Hive, HBase等项目



#2. HDFS体系结构
---

HDFS采用Master/Slave结构模型, 一个HDFS集群由`一个`NameNode(主服务器, 管理文件系统的命名空间和客户端对文件的访问操作)和`若干`DataNode(管理存储的数据, 处理文件系统客户端的文件读写请求)组成

#3. MapReduce编程模型
---

MapReduce框架是由`一个`单独运行在主节点的JobTracker和运行在每个集群从节点的TaskTracker共同组成, 主节点`负责调度`构成一个作业的所有任务,  监控任务执行情况, 从节点负责处理主节点分配的任务.



---

**整个系统:**

- HDFS使数据分布式存储(文件在HDFS底层被分割成一个个Block, 这些Block分散地存储在不同的DataNode上, 每个Block还可以复制数据存储在不同的DataNode上来实现容错性)
- MapReduce编程模型进行分布式并行计算
- 对N个Block, 启动N个Map任务
- Map任务的中间结果进行一些中间操作后, JobTracker通知Reduce到某个TaskTracker去中间结果, 获得最终结果.

> 初学Hadoop, 如有任何错误, 请指出, 非常感谢
