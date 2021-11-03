# 本地Hadoop安装部署

1. 将winutils.exe拷贝到windows下hadoop的bin目录
2. 将hadoop.dll与winutils.exe拷贝到目录`C:\Windows\System32` 下
3. 设置环境变量`HADOOP_HOME`为`.../hadoop2.XXX/`
4. 打开eclipse/idea导入jar包(mapreduce,yarn,hdfs,common)，编写程序运行wordcount



# WordCount程序编写



1、Map程序的编写

```java
import java.io.IOException;
import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

//四个泛型中，前两个指定mapper输入数据类型，KEYIN是输入的key的类型，VALUEIN是输入的value的类型
//map 和 reduce 的数据输入输出都是以key-value对的形式封装
//默认情况下，框架传递给我们的mapper的数据中，key是要处理的文本中一行的起始偏移量，这一行的内容作为value
public class WCMapper extends Mapper<LongWritable, Text, Text, LongWritable> {

    //mapreduce框架每读一行数据就调用一次该方法
    @Override
    protected void map(LongWritable key, Text value,
                       Mapper<LongWritable, Text, Text, LongWritable>.Context context)
            throws IOException, InterruptedException {
        //具体业务逻辑就写在这个方法体中，而且我们业务要处理的数据已经被框架传进来了，在方法的参数中key-value
        //key是这一行的起始偏移量，这一行的内容作为value

        //将这行内容转换为String类型
        String line = value.toString();

        //这一行文本按特定分隔符切分
        String[] words = StringUtils.split(line," ");

        //遍历这个单词数组输出为k-v形式，k：单词，v：1
        for(String word:words){
            context.write(new Text(word), new LongWritable(1));
        }

    }

}
```



2、reduce程序编写

```java
import java.io.IOException;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class WCReducer extends Reducer<Text, LongWritable, Text, LongWritable> {
    //框架在map处理完成后，将所有的kv对缓存起来，进行分组，然后传递一个组<key,values{}>,调用一次reduce方法
    @Override
    protected void reduce(Text key, Iterable<LongWritable> values,
                          Reducer<Text, LongWritable, Text, LongWritable>.Context context)
            throws IOException, InterruptedException {

        long count = 0;
        for(LongWritable value:values){
            count+=value.get();
        }
        //输出这个单词的统计结果
        context.write(key, new LongWritable(count));
    }
}
```



3、总控程序编写

- 集群版（需要打成jar包放到Linux中用yarn去运行）

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

/**
 * 用来描述一个特定作业
 * 比如，改作业使用哪个类作为逻辑处理中的map，哪个作为reduce
 * 还可以指定改作业要处理的数据所在的路径
 * 还可以指定改作业输出的结果放到哪个路径
 *
 * @author WEI
 */
public class WCRunner {
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();

        Job wcjob = Job.getInstance(conf);

        //设置整个job所用的那些类在哪个jar包
        wcjob.setJarByClass(WCRunner.class);

        //本job使用的mapper和reducer类
        wcjob.setMapperClass(WCMapper.class);
        wcjob.setReducerClass(WCReducer.class);

        //指定reduce的输出数据类型
        wcjob.setOutputKeyClass(Text.class);
        wcjob.setOutputValueClass(LongWritable.class);

        //指定mapper的输出数据类型
        wcjob.setMapOutputKeyClass(Text.class);
        wcjob.setMapOutputValueClass(LongWritable.class);

        //指定要处理的输入数据存放路径
        FileInputFormat.setInputPaths(wcjob, new Path("/wc/src"));

        //指定处理结果的输出数据存放路径
        FileOutputFormat.setOutputPath(wcjob, new Path("/wc/output"));

        //将job提交给集群运行
        wcjob.waitForCompletion(true);

    }
}
```

- 单机版（本地运行，不用mapreduce框架）

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WCRunnerLocal {
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();

        Job wcjob = Job.getInstance(conf);

        //设置整个job所用的那些类在哪个jar包
        wcjob.setJarByClass(WCRunnerLocal.class);

        //本job使用的mapper和reducer类
        wcjob.setMapperClass(WCMapper.class);
        wcjob.setReducerClass(WCReducer.class);

        //指定reduce的输出数据类型
        wcjob.setOutputKeyClass(Text.class);
        wcjob.setOutputValueClass(LongWritable.class);

        //指定mapper的输出数据类型
        wcjob.setMapOutputKeyClass(Text.class);
        wcjob.setMapOutputValueClass(LongWritable.class);

        //指定要处理的输入数据存放路径
        FileInputFormat.setInputPaths(wcjob, new Path("L:\\documents\\hadoopTest\\src"));

        //指定处理结果的输出数据存放路径
        FileOutputFormat.setOutputPath(wcjob, new Path("L:\\documents\\hadoopTest\\output"));

        //将job提交给集群运行
        wcjob.waitForCompletion(true);

    }
}
```