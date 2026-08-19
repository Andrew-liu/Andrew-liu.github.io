title: Hive和Java API操作HBase实践
date: 2015-05-03 15:09:40
tags: MapReduce
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


> 由于五一假期, 成文较为简略, 一些细节部分并没有详细介绍, 如有需求, 可以参考之前几篇相当MapRuduce主题的博文.

# HBase实践

- 修改MapReduce阶段倒排索引的信息通过文件输出, 而每个词极其对应的`平均出现次数`信息写入到Hbase的表`Wuxia`中(具体的要求可以查看之前的博文[MapReduce实战之倒排索引](http://andrewliu.tk/2015/04/18/MapReduce%E4%B9%8B%E5%80%92%E6%8E%92%E7%B4%A2%E5%BC%95/#4-_MapReduce实战之倒排索引))
- 编写Java程序, 遍历上一步保存在HBase中的表, 并把表格的内容保存到本地文件中.
- Hive使用Hive Shell命令行创建表(`表名:` Wuxia, (word string, count double)), 导入平均出现次数的数据
    - 查询出现次数大于300的词语
    - 查询前100个出现次数最多的数

<!--more-->


```java
import java.io.IOException;
import java.nio.ByteBuffer;
import java.util.StringTokenizer;
import java.util.Iterator;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.*;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.Reducer.Context;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.mapreduce.TableReducer;
import org.apache.hadoop.hbase.mapreduce.TableMapReduceUtil;
import org.apache.hadoop.hbase.mapreduce.TableOutputFormat;
import org.apache.hadoop.hbase.mapreduce.TableReducer;
import org.apache.hadoop.hbase.io.*;
import org.apache.hadoop.hbase.util.Bytes;



public class InvertedIndexHbase {
    //创建表并进行简单配置
    public static void createHBaseTable(Configuration conf, String tablename) throws IOException {
//      HBaseConfiguration configuration = new HBaseConfiguration();
        HBaseAdmin admin = new HBaseAdmin(conf);
        if (admin.tableExists(tablename)) {  //如果表已经存在
            System.out.println("table exits, Trying recreate table!");
            admin.disableTable(tablename);
            admin.deleteTable(tablename);
        }
        HTableDescriptor htd = new HTableDescriptor(tablename); //row
        HColumnDescriptor col = new HColumnDescriptor("content"); //列族
        htd.addFamily(col); //创建列族
        System.out.println("Create new table: " + tablename);
        admin.createTable(htd); //创建表
    }
    //map函数不变
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
    //combine函数不变
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
            wordDoc.set(key.toString().substring(0, splitIndex));  //key变为单词
            wordCount.set(sum + "");  //value变为doc:词频
            context.write(wordDoc, wordCount);
        }
    }
    //reduce将数据存入HBase
    public static class Reduce
        extends TableReducer<Text, Text, NullWritable> {
        private Text temp = new Text();

        public void reduce(Text key, Iterable<Text> values, 
                Context context) throws IOException, InterruptedException {
            int sum = 0;
            int count = 0;
            Iterator<Text> it = values.iterator();
            //形成最终value
            for(;it.hasNext();) { 
                count++;
                temp.set(it.next());
                sum += Integer.parseInt(temp.toString());
            }
            float averageCount = (float)sum / (float)count;
            FloatWritable average = new FloatWritable(averageCount);
            //加入row为key.toString()
            Put put = new Put(Bytes.toBytes(key.toString()));  //Put实例, 每一词存一行
            //列族为content, 列修饰符为average表示平均出现次数, 列值为平均出现次数
            put.add(Bytes.toBytes("content"), Bytes.toBytes("average"), Bytes.toBytes(average.toString()));
            context.write(NullWritable.get(), put); 
        }
    }

    public static void main(String[] args) throws Exception {
        String tablename = "Wuxia";
        Configuration conf = HBaseConfiguration.create();
        conf.set(TableOutputFormat.OUTPUT_TABLE, tablename);
        createHBaseTable(conf, tablename);
        Job job = Job.getInstance(conf, "Wuxia");  //配置作业名
        //配置作业的各个类
        job.setJarByClass(InvertedIndexHbase.class);
        job.setMapperClass(Map.class);
        job.setCombinerClass(InvertedIndexCombiner.class);
        job.setReducerClass(Reduce.class);
//        TableMapReduceUtil.initTableReducerJob(tablename, Reduce.class, job);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);
        job.setOutputFormatClass(TableOutputFormat.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
      }
}
```

然后在Hadoop执行操作. 

```
$ hdfs dfs -mkdir /user
$ hdfs dfs -mkdir /user/input
$ hdfs dfs -put /Users/andrew_liu/Java/Hadoop/wuxia_novels/*  /user/input
$ hadoop jar WorkSpace/InvertedIndexHbase.jar InvertedIndexHbase  /user/input output1
```


执行成功结束后, 打开HBase Shell的操作

```
$ hbase shell
> scan 'Wuxia'
```

#HBase中数据写入本地文件


```java
import java.io.FileWriter;
import java.io.IOException;
import java.io.FileWriter;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.PrintWriter;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.KeyValue;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.client.ResultScanner;
import org.apache.hadoop.hbase.util.Bytes;


public class Hbase2Local {
    static Configuration conf = HBaseConfiguration.create();
    public static void getResultScan(String tableName, String filePath) throws IOException {
        Scan scan = new Scan();
        ResultScanner rs = null;
        HTable table =  new HTable(conf, Bytes.toBytes(tableName));
        try {
            rs = table.getScanner(scan);
            FileWriter fos = new FileWriter(filePath, true);
            for (Result r : rs) {
//              System.out.println("获得rowkey: " + new String(r.getRow()));
                for (KeyValue kv : r.raw()) {
//                  System.out.println("列: " + new String(kv.getFamily()) + "  值: " + new String(kv.getValue()));
                    String s = new String(r.getRow() + "\t" + kv.getValue() + "\n");
                    fos.write(s);
                }
            }
            fos.close();
        } catch (IOException e) {
            // TODO: handle exception
            e.printStackTrace();
        }
        rs.close();
    }
    public static void main(String[] args) throws Exception {
        String tableName = "Wuxia";
        String filePath = "/Users/andrew_liu/Java/WorkSpace/Hbaes2Local/bin/Wuxia";
        getResultScan(tableName, filePath);
    }
}

```

#Hive实践

将本地数据导入Hive


```
hive> create table Wuxia(word string, count double) row format delimited fields terminated by '\t' stored as textfile;
Time taken: 0.049 seconds
hive> load data local inpath '/Users/andrew_liu/Downloads/Wuxia.txt' into table Wuxia;
Loading data to table default.wuxia
Table default.wuxia stats: [numFiles=1, totalSize=2065188]
OK
Time taken: 0.217 seconds
```


输出出现次数大于300的词语

```
select * from Wuxia order by count desc limit 100;
```
