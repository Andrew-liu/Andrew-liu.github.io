title: 在Mac OSX Yosemite上安装Hadoop
date: 2015-03-05 17:30:20
tags: MapReduce
---


本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


#1. 安装Homebrew和Cask
---

打开Mac终端, 安装OS X 不可或缺的套件管理器[homebrew](http://brew.sh/index_zh-cn.html)和[homebrew cask](http://caskroom.io/)

```
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

$ brew install caskroom/cask/brew-cask
```

<!--more-->

#2. 安装Java
---

Hadoop是由Java编写, 所以需要预先安装Java 6或者更高的版本

```
$ brew update && brew upgrade brew-cask && brew cleanup && brew cask cleanup
$ brew cask install java
```

测试是否安装成功

```
$ java -version
```


#3. 配置SSH

> 为了确保在远程管理Hadoop以及Hadoop节点用户共享时的安全性, Hadoop需要配置使用SSH协议

**首先在系统偏好设置->共享->`打开`远程登录服务->`右侧选择`允许所有用户访问**

**生成密钥对,执行如下命令**

```
$ ssh-keygen -t rsa
```

执行这个命令后, 会在当前用户目录中的.ssh文件夹中生成id_rsa文件, 执行如下命令:

```
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

使用下面命令测试是否能够不使用密码登录

```
$ ssh localhost
# Last login: Thu Mar  5 17:30:07 2015
```

#4. 安装Hadoop
---

```
$ brew install hadoop
```

Hadoop会被安装在`/usr/local/Cellar/hadoop`目录下


##4.1. 配置Hadoop

配置hadoop-env.sh

在目录`/usr/local/Cellar/hadoop/2.6.0/libexec/etc/hadoop`下找到`hadoop-env.sh`文件


找到其中一行:

```
export HADOOP_OPTS="$HADOOP_OPTS -Djava.net.preferIPv4Stack=true"
```

修改为:

```
export HADOOP_OPTS="$HADOOP_OPTS -Djava.net.preferIPv4Stack=true -Djava.security.krb5.realm= -Djava.security.krb5.kdc="
export JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk1.8.0_20.jdk/Contents/Home" #改为自己JDK目录
```


在目录`/usr/local/Cellar/hadoop/2.6.0/libexec/etc/hadoop`下找到`core-site.xml`

```
<!-- Put site-specific property overrides in this file. -->
 <configuration>
  <property>
     <name>hadoop.tmp.dir</name>
<value>/usr/local/Cellar/hadoop/hdfs/tmp</value>
    <description>A base for other temporary directories.</description>
  </property>
  <property>
     <name>fs.default.name</name>
     <value>hdfs://localhost:8020</value>
  </property>
</configuration>                                             
```

在目录`/usr/local/Cellar/hadoop/2.6.0/libexec/etc/hadoop`下找到mapred-site.xml, 在其中添加:

```
 <configuration>
       <property>
         <name>mapred.job.tracker</name>
         <value>localhost:8021</value>
       </property>
 </configuration>
```


在目录`/usr/local/Cellar/hadoop/2.6.0/libexec/etc/hadoop`下找到`hdfs-site.xml`


```
 <configuration>
    <property>
      <name>dfs.replication</name>
      <value>1</value>
     </property>
 </configuration>
```




在运行后台程序前, 必须格式化新安装的HDFS, 并通过创建存储目录和初始化元数据创新空的文件系统, 执行下面命令:

```
$ hdfs namenode -format

#生成类似下面的字符串:

...

#此书省略大部分


STARTUP_MSG:   java = 1.6.0_65
************************************************************
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at Andrew-liudeMacBook-Pro.local/192.168.1.100
************************************************************/
```


##4.2. 启动后台程序

在`/usr/local/Cellar/hadoop/2.6.0/sbin`目录下, 执行如下命令

```
$ start-dfs.sh  #启动HDFS
$ stop-dfs.sh  #停止HDFS
```

成功启动服务后, 可以直接在浏览器中输入[http://localhost:50070/](http://localhost:50070/)访问Hadoop页面


Resource Manager: http://localhost:50070
JobTracker: http://localhost:8088
Specific Node Information: http://localhost:8042


#5. 参考链接

[INSTALLING HADOOP ON MAC OSX YOSEMITE TUTORIAL PART 1.](http://amodernstory.com/2014/09/23/installing-hadoop-on-mac-osx-yosemite/)
[如何在MacOSX上安装Hadoop(how to install hadoop on mac os x)](http://www.ifzer.com/2014/10/31/how_to_install_hadoop_on_mac_ox_x/)
