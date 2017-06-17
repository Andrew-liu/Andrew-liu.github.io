title: Mac下安装HBase及详解
date: 2015-04-19 12:57:09
tags: MapReduce
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.



#1. 千篇一律的HBase简介
---

HBase是Hadoop的数据库, 而Hive数据库的管理工具, HBase具有`分布式, 可扩展及面向列存储`的特点(**基于谷歌BigTable**). HBase可以使用本地文件系统和HDFS文件存储系统, 存储的是松散的数据(**key-value的映射关系**).

> HBase位于HDFS的上层, 向下提供存储, 向上提供运算

<!--more-->

#2. HBase安装
---

> HBase有单机, 伪分布式, 全分布式运行模式


依赖:

- 匹配HBase的Hadoop版本
- Java JDK 1.6+
- SSH

**安装**

```
$ brew install hbase
# 安装在/usr/local/Cellar/hbase/1.0.0
```

**配置HBase**

在`conf/hbase-env.sh`设置JAVA_HOME

```
$ cd /usr/local/Cellar/hbase/1.0.0/libexec/conf
$ vim hbase-env.sh

export JAVA_HOME="/usr/bin/java"
```

在`conf/hbase-site.xml`设置HBase的核心配置

```
$ vim hbase-site.xml

<configuration>
  <property>
    <name>hbase.rootdir</name>
    //这里设置让HBase存储文件的地方
    <value>file:///Users/andrew_liu/Downloads/hbase</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    //这里设置让HBase存储内建zookeeper文件的地方
    <value>/Users/andrew_liu/Downloads/zookeeper</value>
  </property>
</configuration>
```


`/usr/local/Cellar/hbase/1.0.0/bin/start-hbase.sh`提供HBase的启动

```
$ ./start-hbase.sh                                          
starting master, logging to /usr/local/Cellar/hbase/1.0.0/libexec/bin/../logs/hbase-andrew_liu-master-Andrew-liudeMacBook-Pro.local.out
```

**验证是否安装成功**

```
$ jps

3440 Jps
3362 HMaster # 有HMaster则说明安装成功
1885
```

**启动HBase Shell**

```
$ ./bin/hbase shell

HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.0.0, r6c98bff7b719efdb16f71606f3b7d8229445eb81, Sat Feb 14 19:49:22 PST 2015

1.8.7-p357 :001 >
1.8.7-p357 :001 > exit  #退出shell
```

**停止HBase运行**

```
$ ./bin/stop-hbase.sh
stopping hbase....................
```


#3. 伪分布式模式

> 必须关闭HBase


修改hbase-env.sh

```
HBASE_MANAGE_ZK = true
```

修改hbase-site.xml, 设置HBase使用分布式模式运行

```
<configuration>
  <property>
    <name>hbase.rootdir</name>
    //Here you have to set the path where you want HBase to store its files.
    <value>hdfs://localhost:8020/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
  <value>true</value>
</property>
</configuration>
```

> `hbase.rootdir`路径一定要跟hadoop中`core-site.xml`中fs.default.name相同

>  **change the hbase.rootdir from the local filesystem to the address of your HDFS instance**  ---offical quick start


> 在启动HBase之前, 请先启动Hadoop, 使之运行

启动HBase


```
$ ./start-hbase.sh
$ jps  #验证是否启动成功, 包含HMaster和HRegionServer说明启动成功

6400 DataNode
7872 Jps
7702 HMaster  
7624 HQuorumPeer
6315 NameNode
6508 SecondaryNameNode
6716 NodeManager
7804 HRegionServerHBase 
6623 ResourceManager
```

如果在启动HBase后, 提示如下

```
regionserver running as process 4667. Stop it first.
#请执行以下操作
$ kill -9 4667  #这里4667是要杀掉的进程号
```


启动成功HBase会在HDFS下创建/hbase目录

```
$ hdfs dfs -ls /hbase
```

#4. HBase Shell
---

```
$ hbase shell  #启动HBase Shell

#创建表
> create 'student', 'description', 'course'  #创建表名为student的表, 指明两个列名, 分别为description和course

#信息明细
> list 'student'  #列出list表信息

#插入数据
> put 'student', 'row1', 'description:age', '18'  #意思为在student表row1处插入description:age的数据为18
> put 'student', 'row1', 'description:name', 'liu'
put 'student', 'row1', 'course:chinese', '100'

#一次扫描所有数据
> scan 'student

#使表失效 / 有效
> disable 'student'
> enable 'student'

#删除表(要先disable)
>  drop 'student'

#退出shell
> quit

```

#5. HBase与HDFS
---


> HBase是一个稀疏的长期存储的, 多维度的, 排序的映射表, 通过行键, 行键 + 时间戳 或 行键 + 列(列族: 列修饰符)就可以定位特殊的数据

##5.1. HBase体系结构


HBase的服务器体系遵从简单的主从服务器架构, 由`HRegion服务器群`和`HBase服务器构成`, Master服务器负责管理所有的HRegion服务器, 而HBase中所有的服务器通过ZooKeeper来进行协调并处理HBase服务器运行期间可能遇到的错误.

HBase逻辑上的表可能会被划分为多个HRegion, 然后存储在HRegion服务器上.

- HBase不涉及数据的直接删除和更新, 当Store中的Storefile数量超出阈值会触发合并操作
- HMaster的主要任务是告诉每个HRegion服务器它要维护那些HRegion
- ZooKeeper存储的是HBase中ROOT表和META表的位置, ZooKeeper还负责监控各个机器的状态

##5.2. Java API

- HBaseConfiguration, 通过此类对HBase进行配置
- HBaseAdmin, 提供一个接口来管理HBase数据库的表信息, 提供创建, 删除表, 列出表项, `使表有效或无效`, 以及添加或删除列族成员
- HTableDescriptor, 包含了表的名字及对应表的列族
- HColumnDescriptor, 维护关于列族的信息
- HTable, 用来与HBase表进行通信
- Put, 用来对单个行执行添加操作
- Get, 用来获取单个行的相关信息
- Result, 存储Get或者Scan操作后获取的表的单行值


#6. 参考链接
---

- [HBase Quick Start](http://hbase.apache.org/book.html#quickstart)
- [Hadoop & Hbase on OSX 10.8 Mountain Lion](http://freddy.cellcore.org/post/52568231952/hadoop-hbase-on-osx-10-8-mountain-lion)
- [HBase - Installation](http://www.tutorialspoint.com/hbase/hbase_installation.htm)
- `<Hadoop Action>`
- [Hadoop+HBase 伪分布式安装配置](http://www.cnblogs.com/Dreama/articles/2219190.html)

