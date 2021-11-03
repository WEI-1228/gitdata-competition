# 自定义Partitioner

**Partitioner**

```java
package xyz.liujiawei.areapartition;

import org.apache.hadoop.mapreduce.Partitioner;

import java.util.HashMap;

public class AreaPartitioner<KEY, VALUE> extends Partitioner<KEY, VALUE> {

    private static HashMap<String, Integer> areaMap = new HashMap<>();

    static {
        areaMap.put("135", 0);
        areaMap.put("136", 1);
        areaMap.put("137", 2);
        areaMap.put("138", 3);
        areaMap.put("139", 4);
    }

    @Override
    public int getPartition(KEY key, VALUE value, int i) {
        //从key中拿到手机号，查询手机归属地字典，不同省份返回不同组号

        return areaMap.get(key.toString().substring(0, 3)) == null ? 5 : areaMap.get(key.toString().substring(0, 3));
    }
}
```

**WR**

```java
/**
 * 对流量原始日志进行流量统计，将不同省份的用户统计结果输出到不同文件
 * 需要自定义改造两个机制：
 * 1、改造分区的逻辑，自定义一个partitioner
 * 2、自定义reducer task的并发任务数
 */
public class FlowSumArea {

    static public class mapper extends Mapper<LongWritable, Text, Text, FlowBean> {
        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            String line = value.toString();
            String[] field = StringUtils.split(line, "\t");
            String phoneNum = field[0];
            long up_flow = Long.parseLong(field[7]);
            long down_flow = Long.parseLong(field[8]);

            //封装kv对并输出
            context.write(new Text(phoneNum), new FlowBean(phoneNum, up_flow, down_flow));
        }
    }

    static public class reducer extends Reducer<Text, FlowBean, Text, FlowBean> {

        @Override
        protected void reduce(Text key, Iterable<FlowBean> values, Context context) throws IOException, InterruptedException {

            long up_flow_counter = 0;
            long down_flow_counter = 0;

            for (FlowBean bean : values) {
                up_flow_counter += bean.getUp_flow();
                down_flow_counter += bean.getDown_flow();
            }
            context.write(key, new FlowBean(key.toString(), up_flow_counter, down_flow_counter));
        }
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(FlowSumArea.class);

        job.setMapperClass(mapper.class);
        job.setReducerClass(reducer.class);

        //设置自定义的逻辑分组
        job.setPartitionerClass(AreaPartitioner.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowBean.class);

        //设置reduce的任务并发数，应该跟分组的数量保持一致
        job.setNumReduceTasks(6);

        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        System.exit(job.waitForCompletion(true) ? 0 : -1);
    }
}
```