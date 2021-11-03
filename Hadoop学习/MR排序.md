bean

```java
package xyz.liujiawei.learn.flowsum;

import org.apache.hadoop.io.WritableComparable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

public class FlowBean implements WritableComparable<FlowBean> {

    private String phoneNumber;
    private long up_flow;
    private long down_flow;

    //反序列化需要一个空参的构造函数，所以显示定义一个
    public FlowBean() {
    }

    //为了对象初始化方便，加入一个带参的构造函数
    public FlowBean(String phoneNumber, long up_flow, long down_flow) {
        this.phoneNumber = phoneNumber;
        this.up_flow = up_flow;
        this.down_flow = down_flow;
        this.sum_flow = up_flow + down_flow;
    }

    public long getUp_flow() {
        return up_flow;
    }

    public void setUp_flow(long up_flow) {
        this.up_flow = up_flow;
    }

    public long getDown_flow() {
        return down_flow;
    }

    public void setDown_flow(long down_flow) {
        this.down_flow = down_flow;
    }

    public long getSum_flow() {
        return sum_flow;
    }

    public void setSum_flow(long sum_flow) {
        this.sum_flow = sum_flow;
    }

    private long sum_flow;

    public String getPhoneNumber() {
        return phoneNumber;
    }

    public void setPhoneNumber(String phoneNumber) {
        this.phoneNumber = phoneNumber;
    }


    //将对象数据序列化到流中
    @Override
    public void write(DataOutput dataOutput) throws IOException {
        dataOutput.writeUTF(this.phoneNumber);
        dataOutput.writeLong(up_flow);
        dataOutput.writeLong(down_flow);
        dataOutput.writeLong(sum_flow);
    }


    //从数据流中反序列化出对象数据
    //从数据流中读出对象字段时，必须跟序列化时的顺序保持一致
    @Override
    public void readFields(DataInput dataInput) throws IOException {
        phoneNumber = dataInput.readUTF();
        up_flow = dataInput.readLong();
        down_flow = dataInput.readLong();
        sum_flow = dataInput.readLong();
    }

    @Override
    public String toString() {
        return up_flow + "\t" + down_flow + "\t" + sum_flow;
    }

    @Override
    public int compareTo(FlowBean o) {
        return this.sum_flow>o.getSum_flow()?-1:1;
    }
}
```



```java
package xyz.liujiawei.learn.flowsort;

import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import xyz.liujiawei.learn.flowsum.FlowBean;

import java.io.IOException;

public class SortMR {

    public static class SortMapper extends Mapper<LongWritable, Text, FlowBean, NullWritable>{

        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            String line = value.toString();
            String[] fields = StringUtils.split(line, "\t");

            String phoneNum = fields[0];
            long u_flow = Long.parseLong(fields[1]);
            long d_flow = Long.parseLong(fields[2]);

            context.write(new FlowBean(phoneNum,u_flow,d_flow), NullWritable.get());
        }
    }

    public static class SortReducer extends Reducer<FlowBean,NullWritable,FlowBean,NullWritable>{
        @Override
        protected void reduce(FlowBean key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {
            context.write(key,NullWritable.get());
        }
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(SortMR.class);

        job.setMapperClass(SortMapper.class);
        job.setReducerClass(SortReducer.class);

//        job.setMapOutputKeyClass(Text.class);
//        job.setMapOutputValueClass(FlowBean.class);

        job.setOutputKeyClass(FlowBean.class);
        job.setOutputValueClass(NullWritable.class);

        FileInputFormat.setInputPaths(job,new Path(args[0]));
        FileOutputFormat.setOutputPath(job,new Path(args[1]));

        System.exit(job.waitForCompletion(true)?0:-1);
    }
}
```

