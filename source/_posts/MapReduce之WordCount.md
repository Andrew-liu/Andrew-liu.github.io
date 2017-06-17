title: MapReduce之WordCount
date: 2015-03-29 09:26:05
tags: MapReduce
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


#1. 再述MapReduce计算模型
---


- JobTracker用于管理和调度工作(`一个集群只有一个JobTracker`)
- TaskTracker用于执行工作
- 每个MapReduce任务被初始化为一个Job, 每个Job分为Map(接收键值对)和Reduce阶段

<!--more-->

InputSplit(存储分片长度和记录数据位置的数组)把输入数据传送给单独的Map, 数据传给Map后, Map将输入分片传送到InputFormat()上, InputFormat()(`用来生成可供Map处理的键值对`)调用getRecordReader()方法生成RecordReader, RecordReader再通过createKey(), createValue()方法创建可供Map处理的键值对.`TextInputFormat`是Hadoop默认的输入方法, 每个文件都读作为Map的输入, 每行数组生成一条键值对(key在数据分片中的字节偏移量LongWritable, value是每行内容Text)


#2. 编译打包运行WordCount
---

总结一下通过Eclipse来编译打包运行自己写的MapReduce程序(`基于Hadoop2.6.0`)


##2.1. Hadoop库

在编写Hadoop程序会用到Hadoop库, 所以需要一些Hadoop库文件, 用于编译

- hadoop-common-2.6.0.jar
- hadoop-mapreduce-client-core-2.6.0.jar
- hadoop-test-1.2.1.jar

下载地址[Group: org.apache.hadoop](http://mvnrepository.com/artifact/org.apache.hadoop)下载对应版本的库文件



##2.2. 创建工程

1. 使用Eclipse创建名为WordCount的工程
2. 在`Project Properties -> Java Build Path -> Libraries -> Add External Jars` 添加第一步所下载Jar包, 点击OK
3. 创建WordCount.java源文件

```
#WorkCount.java

import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {

  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();
    /*
     * LongWritable 为输入的key的类型
     * Text 为输入value的类型
     * Text-IntWritable 为输出key-value键值对的类型
     */
    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());  // 将TextInputFormat生成的键值对转换成字符串类型
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }

  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();
    /*
     * Text-IntWritable 来自map的输入key-value键值对的类型
     * Text-IntWritable 输出key-value 单词-词频键值对
     */
    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();  // job的配置
    Job job = Job.getInstance(conf, "word count");  // 初始化Job
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);  
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[0]));  // 设置输入路径
    FileOutputFormat.setOutputPath(job, new Path(args[1]));  // 设置输出路径
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}

```

##2.3. 打包源文件

1. 在`File -> Export -> Java -> JAR File`, 然后点击next
2. 选中WordCount源文件, 设置输出路径和文件名WordCount.jar, 选择Finish则打包成功
3. 在输出路径生成Wordcount.jar




##2.4. 启动HDFS服务

打开目录/usr/local/Cellar/hadoop/2.6.0/sbin

```
$ start-dfs.sh  #启动HDFS
$ jps  #验证是否启动成功
8324 Jps
8069 DataNode
7078 NodeManager
6696 NameNode
6987 ResourceManager
3453 SecondaryNameNode
$ stop-dfs.sh  #停止HDFS
```

成功启动服务后, 可以直接在浏览器中输入[http://localhost:50070/](http://localhost:50070/)访问Hadoop页面

##2.5. 将输入文件上传到HDFS

打开目录/usr/local/Cellar/hadoop/2.6.0/bin

```
#在HDFS上创建输入/输出文件夹
$ hdfs dfs -mkdir /user
$ hdfs dfs -mkdir /user/input
$ hdfs dfs -ls /user
#上传本地file中文件到集群的input目录下
$ hdfs dfs -put /Users/andrew_liu/Java/Hadoop/input/* /user/input
#查看上传到HDFS输入文件夹中到文件
$ hdfs dfs -ls /user/input

#输出结果
-rw-r--r--   1 andrew_liu supergroup    1808033 2015-04-05 12:37 /user/input/rural.txt
-rw-r--r--   1 andrew_liu supergroup    2246756 2015-04-05 12:37 /user/input/science.txt
```


##2.6. 运行Jar文件

```
#在当前文件夹创建一个工作目录
$ mkdir WorkSpace 
#将打包号的Jar包复制到当前工作目录
$cp /usr/local/Cellar/hadoop/2.6.0/bin/WorkSpace/WordCount.jar ./WorkSpace
#运行Jar文件, 各字段的意义(Hadoop打包命令, 指定Jar文件, 指定Jar文件入口类, 指定job的HDFS上的输入文件目录, 指定job的HDFS输出文件目录)
$ hadoop jar WorkSpace/WordCount.jar WordCount /user/input output
```


```
... 省略部分
    File System Counters
        FILE: Number of bytes read=2025025
        FILE: Number of bytes written=4443318
        FILE: Number of read operations=0
        FILE: Number of large read operations=0
        FILE: Number of write operations=0
        HDFS: Number of bytes read=10356334
        HDFS: Number of bytes written=616286
        HDFS: Number of read operations=25
        HDFS: Number of large read operations=0
        HDFS: Number of write operations=5
    Map-Reduce Framework
        Map input records=33907
        Map output records=663964
        Map output bytes=6687108
        Map output materialized bytes=1005779
        Input split bytes=216
        Combine input records=663964
        Combine output records=68147
        Reduce input groups=55800
        Reduce shuffle bytes=1005779
        Reduce input records=68147
        Reduce output records=55800
        Spilled Records=136294
        Shuffled Maps =2
        Failed Shuffles=0
        Merged Map outputs=2
        GC time elapsed (ms)=187
        Total committed heap usage (bytes)=1323827200
    Shuffle Errors
        BAD_ID=0
        CONNECTION=0
        IO_ERROR=0
        WRONG_LENGTH=0
        WRONG_MAP=0
        WRONG_REDUCE=0
    File Input Format Counters
        Bytes Read=4054789
    File Output Format Counters
        Bytes Written=616286
```


##2.7. 查看运行结果

```
#查看FS上output目录内容
$ hdfs dfs -ls output
-rw-r--r--   1 andrew_liu supergroup          0 2015-04-05 13:20 output/_SUCCESS
-rw-r--r--   1 andrew_liu supergroup     616286 2015-04-05 13:20 output/part-r-00000 # 存放结果文件
#查看结果输出文件内容
hdfs dfs -cat output/part-r-00000
```


##2.8. MapReduce运行流程


- JobTracker调度任务个TaskTracker, TaskTracker执行任务时, 返回进度报告, 如果执行失败, JobTracker将任务分配给另一个TaskTracker, 知道任务完成
- 数据按照TextInputFormat被处理成InputSplit, 输入到Map中, Map读取InputSplit指定位置的数据, `按照设定的方式处理数据`, 最后写入本地磁盘
- Reduce读取Map输出数据, 合并value, 然后输出到HDFS上


#3. MapReduce任务优化
---

- 计算性能优化
- I/O操作优化

1. 任务调度(就近原则, 选用空闲原则)
2. 数据预处理应合理设置block快大小及Map和Reduce任务数量
3. combine函数用于本地合并数据的函数, 运行用户combine用于本地合并, 可减少网络I/O的消耗
4. 对Map输出和最终结果压缩
5. 自定义comparator实现数据的二进制比较, 省去数据序列化和反序列化时间

#4. Hadoop流
---

当一个可执行未见作为Mapper时, 每个Map任务以一个独立的进程启动可执行未见, 任务执行时, 会把输入划分成行提供给可执行文件, 并作为Map的标准输入, Map从标准输出中收集数据, 并转换为`<key, value>`输出

Reduce任务启动可执行文件, 将键值对转化为标准输入, Reduce从标准输出中收集数据, 并转换为`<key, value>`输出



#5. 参考链接
---

- [MapReduce Tutorial 2.6.0](http://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html)
- [Group: org.apache.hadoop](http://mvnrepository.com/artifact/org.apache.hadoop)
- [HADOOP TUTORIAL: CREATING MAPREDUCE JOBS IN JAVA](http://glebche.appspot.com/static/hadoop-ecosystem/mapreduce-job-java.html)
- [BIG DATA AND HADOOP](http://glebche.appspot.com/static/hadoop-ecosystem/hadoop-hive-tutorial.html)
- `<Hadoop Action>`
- [【Hadoop基础教程】5、Hadoop之单词计数](http://blog.csdn.net/andie_guo/article/details/44055863)
