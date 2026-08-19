title: Mac下安装Hive及使用
date: 2015-04-25 10:11:22
tags: MapReduce
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

> Hive利用MapReduce编程技术, 实现了部分SQL语句, 提供了类SQL的编程接口.

#1. Hive简介
---

Hive是一个基于`Hadoop文件系统HDFS`上的数据仓库架构, 它为数据仓库管理提供: 数据ETL(抽取, 转换和加载)工具, 数据仓库管理和大型数据集的查询和分析功能. 定义了类SQL语言`Hive QL`. 其优势在于极大的可扩展性, 良好的容错性和低约束的数据输入格式

<!--more-->

## 数据存储
Hive中每个表都有一个对应的存储目录.
Hive有三种模式可以连接到RDBMS中:
1. Single User Mode, 利用此模式连接到一个In-Memory数据库, 一般用于单元测试
2. Multi User Mode, 通过网络连接到一个数据库中(常用)
3. Remote Server Mode, 用于非Java客户端访问元数据库, 在服务器端启动MetaStoreServer, 在客户端利用Thrift协议通过MetaStoreServer访问元数据库




#2. Hive安装
---

```
$ brew install hive
```

配置环境变量

```
$ vim ~/.zshrc

export HADOOP_HOME="/usr/local/Cellar/hadoop/2.6.0"
export JAVA_HOME="$(/usr/libexec/java_home)"
export HIVE_HOME="/usr/local/Cellar/hive/1.0.0/libexec"
```

在hive安装目录下libexec/conf/中

```
$ cp hive-env.sh.template hive-env.sh
$ cp hive-default.xml.template hive-site.xml
```

修改hive-env.sh
```
export HADOOP_HEAPSIZE=1024

# Set HADOOP_HOME to point to a specific hadoop install directory
HADOOP_HOME=/usr/local/Cellar/hadoop/2.6.0

# Hive Configuration Directory can be controlled by:
export HIVE_CONF_DIR=/usr/local/Cellar/hive/1.0.0/libexec/conf

# Folder containing extra ibraries required for hive compilation/execution can be controlled by:
export HIVE_AUX_JARS_PATH=/usr/local/Cellar/hive/1.0.0/libexec/lib
```


修改libexec下conf文件夹中hive-site.xml
```
<property>
    <name>hive.querylog.location</name>
    <value>/usr/local/Cellar/hive/1.0.0/log</value>
    <description>Location of Hive run time structured log file</description>
</property>
<property>
    <name>hive.exec.scratchdir</name>
    <value>/tmp/hive-${user.name}</value>
    <description>HDFS root scratch dir for Hive jobs which gets created with write all (733) permission. For each connecting user, an HDFS scratch dir: $     {hive.exec.scratchdir}/&lt;username&gt; is created, with ${hive.scratch.dir.permission}.</description>
</property>
  <property>
    <name>hive.exec.local.scratchdir</name>
    <value>/tmp/${user.name}</value>
    <description>Local scratch space for Hive jobs</description>
  </property>
<property>
    <name>hive.downloaded.resources.dir</name>
    <value>/tmp/${user.name}_resources</value>
    <description>Temporary local directory for added resources in the remote file system.</description>
  </property>
<property>
    <name>hive.server2.logging.operation.log.location</name>
    <value>/tmp/${user.name}/operation_logs</value>
    <description>Top level directory where operation logs are stored if logging functionality is enabled</description>
  </property>
```

安装mysql
```
$ brew install mysql

$ mysql.server start  #启动mysql
$ mysql -uroot  #root用户进入mysql

mysql> CREATE DATABASE metastore;
mysql> USE metastore;
mysql> CREATE USER 'hive'@'localhost' IDENTIFIED BY 'hive';
mysql> GRANT SELECT,INSERT,UPDATE,DELETE,ALTER,CREATE ON metastore.* TO 'hive'@'localhost';
```
 
启动hadoop服务

```
$ $HODOOP_HOME/sbin/start-dfs.sh  #启动hdfs
```

启动hive

```
$ /usr/local/Cellar/hive/1.0.0/bin/hive

在~/.zshrc 设置快捷方式
alias hive='/usr/local/Cellar/hive/1.0.0/bin/hive'
```

#3. Hive Shell命令
---

```
#显示所有的数据表
hive>show tables; 
#创建一个表,这个表包括两列,分别是整数类型以及字符串 类型,使用文本文件表达,数据域之间的分隔符为\t
hive> create table shakespeare (freq int, word string) row format delimited fields terminated by ‘\t’ stored as textfile;
#显示所创建的数据表的描述,即创建时候对于数据表的定义
hive> describe shakespeare;
数据装入到Hive中
hive> load data inpath “shakespeare_freq” into table shakespeare;
#SELECTS and FILTERS
hive> select * from shakespeare limit 10;
hive> SELECT a.foo FROM invites a WHERE a.ds='2008-08-15';
#Group by
hive> INSERT OVERWRITE TABLE events SELECT a.bar, count(*)
FROM invites a WHERE a.foo > 0 GROUP BY a.bar;
#Join
hive> SELECT t1.bar, t1.foo, t2.foo
FROM pokes t1 JOIN invites t2 ON t1.bar = t2.bar;
```

#4. Hive总结
---

- Hive提供了一种类似于SQL的查询语言, 使得其能够用于用户的交互查询
- 与传统的数据库类似, Hive提供了多个数据表之间的联合查询, 能够完成高效的多个数据表之间的查询
- 通过底层执行引擎的工作, Hive将SQL语言扩展到很大的查询规模

#5. 参考链接
---

- [GettingStartedHive](https://cwiki.apache.org/confluence/display/Hive/GettingStarted)
- [java.net.URISyntaxException when starting HIVE](http://stackoverflow.com/questions/27099898/java-net-urisyntaxexception-when-starting-hive)
- [java.net.ConnectException: Connection refused error when running Hive](http://stackoverflow.com/questions/20171455/java-net-connectexception-connection-refused-error-when-running-hive)
- [MySQL Java Connector 5.1.35](http://mvnrepository.com/artifact/mysql/mysql-connector-java/5.1.35)
- [Simplest way to install and configure Hive for Mac OSX Lion](https://noobergeek.wordpress.com/2013/11/09/simplest-way-to-install-and-configure-hive-for-mac-osx-lion/)
