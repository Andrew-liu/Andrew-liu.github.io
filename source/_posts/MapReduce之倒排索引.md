title: MapReduce之倒排索引
date: 2015-04-18 11:05:41
tags: MapReduce
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.



#1. 系统参数配置
---


通过Hadoop的API对各种组件的参数进行配置

- org.apache.hadoop.conf 系统参数的配置文件处理API
- org.apache.hadoop.fs 抽象文件系统API
- org.apache.hadoop.dfs HDFS模块的实现
- org.apache.hadoop.mapred MapReduce模块实现
- org.apache.hadoop.ipc 封装了网络异步I/O的基础模块
- org.apache.hadoop.io 定义了通用的I/O模块

<!--more-->

#2. 配置开发环境
---


> Hadoop分为三种运行方式: 单机模式, 伪分布模式, 完全分步模式

- 单机模式安装配置方便, 便于调试, 大数据下运行慢
- 伪分布式模式在本地文件系统运行, 运行HDFS文件系统
- 完全分布模式在多台机器的HDFS上运行


#3. MapReduce框架
---

- Map接口需要派生自`Mapper<k1, v1, k2, v2>`
- Reduce接口需要派生自`Reducer<k2, v2, k3, v3>`

输入输出的数据类型要与集成时设置的数据类型一致, Map的输出类型要和Reduce的输入类型对应.

- Hadoop集群上的用户作业采用先入先出调度策略
- Map输出会经过shuffle过程交给Reduce处理 shuffle对Map结果`划分(partition), 排序(sort), 分割(spilt)`, 按照不同的划分将结果送给对应的Reduce


```java
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

public class X {
    public static class Map 
    extends Mapper<LongWritable, Text, Text, IntWritable> {
        public void map(LongWritable key, Text value, Context context)
        throws IOException, InterruptedException {
            // Map逻辑
            }
        }
    }

    public static class Reduce
    extends Reducer<Text, IntWritable, Text, IntWritable> {
        public void reduce(Text key, Iterable<IntWritable> values, 
            Context context) throws IOException, InterruptedException {
            // Reduce逻辑
        }
    }

    public int run(String[] args) throws Exception {
        // 运行配置
        Job job = new Job(getConf());
        job.setJarByClass(Score_Process.class);
        job.steJobName("Score_Process");
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        job.setMapperClass(Map.class);
        job.setCombinerClass(Reduce.class);
        job.setReducerClass(Reduce.class);
        job.setInputFormatClass(TextInputFormat.class);
        job.setOutputFormatClass(TextOutputFormat.class);
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        boolean success = job.waitForCompletion(true);
        return success ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {
        int ret = ToolRunner.run(new Score_Process(), args);
        System.exit(ret);
    }
}
    }

    public int run(String[] args) throws Exception {
        Job job = new Job(getConf());
        job.setJarByClass(Score_Process.class);
        job.steJobName("Score_Process");
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        job.setMapperClass(Map.class);
        job.setCombinerClass(Reduce.class);
        job.setReducerClass(Reduce.class);
        job.setInputFormatClass(TextInputFormat.class);
        job.setOutputFormatClass(TextOutputFormat.class);
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        boolean success = job.waitForCompletion(true);
        return success ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {
        int ret = X.run(new Score_Process(), args);
        System.exit(ret);
    }
}
```


**Mapper和Reducer基类中的其他函数:**
1. setup函数(在task启动之后数据处理前调用一次, 对task的全局处理)
2. cleanup函数(task销毁之前执行)
3. run函数

```
    protected void setup(Context context) 
    throws IOException, InterruptedException {
    }
    protected void cleanup(Context context)
    throws IOException, InterruptedException {
    }
    public void run(Context context) throws IOException, InterruptedException {
        setup(context);
        while(context.nextKeyValue()) {
            map(context.getCurrentKey(), context.getCurrentValue(), context);
        }
        cleanup(context);
    }
```


#4. MapReduce实战之倒排索引
---

**倒排索引**是一种索引方法, 被用来存储在全文搜索下某个单词在一个文档或者一组文档中的存储位置的映射. 它是文档检索系统中最常用的数据结构. 通过倒排索引, 可以根据单词快速获取包含这个单词的文档列表。倒排索引主要由两个部分组成：`单词词典`和`倒排文件`。


> 题目: 使用Hadoop集群测试编写的`带词频属性的文档倒排算法`, 在统计词语的倒排索引时, 除了要输出带词频属性的倒排索引,请再计算出每个词语的`平均出现次数`(平均出现次数 = 词语在全部文档中出现的频数总和 / 包含该词语的文档数)并输出.

> 输出格式: 
词语1   平均出现次数,文档名1:词频;文档名2:词频;文档名3:词频;...
词语2   平均出现次数,文档名1:词频;...
.





```java
import java.io.IOException;
import java.util.StringTokenizer;
import java.util.Iterator;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.*;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;


public class InvertedIndex {
    public static class Map 
    extends Mapper<Object, Text, Text, Text> {
        private Text keyWord = new Text();
        private Text valueDocCount = new Text();

        public void map(Object key, Text value, Context context)
        throws IOException, InterruptedException {
            //获取文档
            FileSplit fileSplit = (FileSplit)context.getInputSplit();
            String fileName = fileSplit.getPath().getName();
            StringTokenizer itr = new StringTokenizer(value.toString());
            while(itr.hasMoreTokens()) {
                keyWord.set(itr.nextToken() + ":" + fileName);  // key为key#doc
                valueDocCount.set("1"); // value为词频
                context.write(keyWord, valueDocCount);
            }
        }
    }

    public static class InvertedIndexCombiner
        extends Reducer<Text, Text, Text, Text> {
        private Text wordCount = new Text();
        private Text wordDoc = new Text();
        //将key-value转换为word-doc:词频
        public void reduce(Text key, Iterable<Text> values, 
            Context context) throws IOException, InterruptedException {
            int sum = 0;
            for (Text value : values) {
                sum += Integer.parseInt(value.toString());
            }
            int splitIndex = key.toString().indexOf(":");  // 找到:的位置
            int splitFileName = key.toString().indexOf(".txt.segmented");
            wordDoc.set(key.toString().substring(0, splitIndex));  //key变为单词
            wordCount.set(key.toString().substring(splitIndex + 1, splitFileName) + ":" + sum);  //value变为doc:词频
            context.write(wordDoc, wordCount);
        }
    }

    public static class Reduce
        extends Reducer<Text, Text, Text, Text> {
        private Text temp = new Text();

        public void reduce(Text key, Iterable<Text> values, 
            Context context) throws IOException, InterruptedException {
            int sum = 0;
            int count = 0;
            Iterator<Text> it = values.iterator();
            StringBuilder all = new StringBuilder();
            //形成最终value
            for(;it.hasNext();) { 
                count++;
                temp.set(it.next());
                all.append(temp.toString());  //将一个文档:1添加到总的string value串中
                int splitIndex = temp.toString().indexOf(":");  // 找到:的位置
                temp.set(temp.toString().substring(splitIndex + 1));  //取出词频字符串
                sum += Integer.parseInt(temp.toString());
                if (it.hasNext()) {
                    all.append(";");
                }
            }
            float averageCount = (float)sum / (float)count;
            FloatWritable average = new FloatWritable(averageCount);
            all.insert(0, average.toString() + ",");
            context.write(key, new Text(all.toString()));
        }
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration(); 
        Job job = Job.getInstance(conf, "Inverted Index");  //配置作业名
        //配置作业的各个类
        job.setJarByClass(InvertedIndex.class);
        job.setMapperClass(Map.class);
        job.setCombinerClass(InvertedIndexCombiner.class);
        job.setReducerClass(Reduce.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
      }
}
```

在单机上运行流程可以查看另一篇博文[MapReduce之WordCount](http://andrewliu.tk/2015/03/29/MapReduce%E4%B9%8BWordCount/)


##4.1. 集群运行流程


```
#第一步对文件打包的过程就不详细解释了, 可以参考单机运行流程
#本机文件复制到集群
$ scp -r InvertedIndexer.jar 集群用户名@集群IP地址:集群目的文件夹  #范例: scp -r InvertedIndexer.jar 2015st27@114.212.190.91:WorkSpace

#ssh远程登录集群
$ ssh 集群用户名@集群IP #范例:ssh 2015st27@114.212.190.91

#如果密码正确会登录集群服务器, 集群上运行Jar包
$ hadoop jar WorkSpace/InvertedIndex.jar InvertedIndex /user/input output
```


#5. 参考链接
---

- [MapReduce Tutorial 2.6.0](http://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html)
- `<Hadoop Action>`
