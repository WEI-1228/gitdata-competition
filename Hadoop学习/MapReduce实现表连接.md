# MapReduce实现表连接

**表一：**

```
id	city
1   北京
2   天津
3   河北
4   山西
5   内蒙古
6   辽宁
7   吉林
8   黑龙江
```

**表二：**

```
id	year	num
1   2010    1962
1   2011    2019
2   2010    1299
2   2011    1355
4   2011    3574
4   2011    3593
9   2010    2303
9   2011    2347
```



## 需求

根据两张表的id，用mapreduce将两个表进行连接操作（关联）

```
1	北京	2011	2019
1	北京	2010	1962
2	天津	2011	1355
2	天津	2010	1299
4	山西	2011	3593
4	山西	2011	3574
```





## 思路：

map程序每次都是读取一行数据，读取两个表内的数据，可以根据数据来源（文件名称）判断当前读取的数据是来自哪一张表，然后打上标记送入reduce去处理，map输出的key是id值，value是id对应的数据。



reduce接收到的数据是**\<key:{value1,value2,value3,......}>**，key是输入的id的值，value中包含表一的id对应的数据，也包含表二的id对应的数据，我们可以通过map里打的标记进行区分，分别记录到`list1`与`list2`中，然后将两个list中的数据进行笛卡尔积就能得到一个id连接后的数据，将所有id都进行这样的操作，就能把整个表都处理完



## 代码

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;
import java.util.LinkedList;
import java.util.List;


public class ReduceJoinTest {

    //mapper类
    public static class mapper extends Mapper<LongWritable, Text, Text, Text> {
        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            
            //下面两步能获取当前行数据的输入文件名称
            FileSplit fileSplit = (FileSplit) context.getInputSplit();
            String name = fileSplit.getPath().getName();

            //将当前行数据转换为标准的String
            String line = value.toString();
            //若数据无效则丢弃
            if (line == null || line.equals("")) return;
			
            //根据空格进行分割
            String[] split = line.split("\\s+");
            
            if (name.contains("tb_a")) {
                //如果当前行是表一，在city前添加一个标记“#”，以跟表二区分
                String id = split[0];
                String city = split[1];
                //输出key为id，value为city
                context.write(new Text(id), new Text("#" + city));
            } else if (name.contains("tb_b")) {
                //如果当前行是表二，在输出的value字段前添加“$”，以跟表一区分
                String id = split[0];
                String num1 = split[1];
                String num2 = split[2];
                //输出key为id，value为year与num
                context.write(new Text(id), new Text("$" + num1 + "\t" + num2));
            }
        }
    }

    //reducer类
    public static class reducer extends Reducer<Text, Text, Text, Text> {
        //输入的数据为<id,{value1,value2,....}>
        @Override
        protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
            //list1存表一带来的数据
            List<String> list1 = new LinkedList<>();
            //list2存表二带来的数据
            List<String> list2 = new LinkedList<>();
            
            //遍历values
            for (Text text : values) {
                String value = text.toString();
                //如果value数据以#开头，则为表一中的数据，添加至list1中
                if (value.startsWith("#")) {
                    value = value.substring(1);
                    list1.add(value);

                } else if (value.startsWith("$")) {
                    //如果value数据以$开头，则为表二中的数据，添加至list2中
                    value = value.substring(1);
                    list2.add(value);
                }
            }
            
            //将两表id相同的数据进行笛卡尔积，key为id，value为list1与list2的组合
            for (String a : list1) {
                for (String b : list2) {
                    context.write(key, new Text(a + "\t" + b));
                }
            }
        }
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

		//下面都是模板，只需修改输入与输出位置即可
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
		
        job.setJarByClass(ReduceJoinTest.class);

        job.setMapperClass(mapper.class);
        job.setReducerClass(reducer.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        FileInputFormat.setInputPaths(job, new Path("L:\\JAVA\\Hadoop\\src\\xyz\\liujiawei\\join\\src\\input"));
        FileOutputFormat.setOutputPath(job, new Path("L:\\JAVA\\Hadoop\\src\\xyz\\liujiawei\\join\\src\\output"));

        System.exit(job.waitForCompletion(true) ? 0 : -1);

    }
}
```



