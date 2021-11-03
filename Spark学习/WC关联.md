# 两表关联Demo1

```java
public class MR2 {

    public static class mapper extends Mapper<LongWritable, Text, Text, Text> {
        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            FileSplit fileSplit = (FileSplit) context.getInputSplit();
            String name = fileSplit.getPath().getName();
            String line = value.toString();
            String[] split = line.split("\t");
            if (name.equals("ip.data")) {
                context.write(new Text(split[0]), new Text("ip" + split[1] + "\t" + split[2]));
            } else {
                context.write(new Text(split[1]), value);
            }
        }
    }

    public static class reducer extends Reducer<Text, Text, Text, NullWritable> {
        @Override
        protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
            String city = null;
            List<String> list = new ArrayList<>();
            for (Text text : values) {
                String line = text.toString();
                if (line.startsWith("ip")) city = line.substring(2);
                else list.add(line);
            }
            for (String s : list) {
                StringBuilder builder = new StringBuilder();
                String[] split = s.split("\t");
                builder.append(split[0]).append("\t");
                builder.append(split[1]).append("\t");
                builder.append(city).append("\t");
                for (int i = 2; i < split.length - 1; i++) {
                    builder.append(split[i]).append("\t");
                }
                builder.append(split[split.length - 1]);
                context.write(new Text(builder.toString()), NullWritable.get());
            }
        }
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(MR2.class);

        job.setMapperClass(mapper.class);
        job.setReducerClass(reducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);
        FileSystem fileSystem = FileSystem.get(conf);
        //若输出路径存在则先删除
        Path outputPath = new Path("/p1/src/output1");
        if (fileSystem.exists(outputPath))
            fileSystem.delete(outputPath, true);
        FileInputFormat.setInputPaths(job, new Path("/p1/src/data"));
        FileOutputFormat.setOutputPath(job, outputPath);
        System.exit(job.waitForCompletion(true) ? 0 : -1);

    }
}
```

# 两表关联Demo2

```java
public class ReduceJoinTest {

    public static class mapper extends Mapper<LongWritable, Text, Text, Text> {
        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            FileSplit fileSplit = (FileSplit) context.getInputSplit();
            String name = fileSplit.getPath().getName();

            String line = value.toString();
            if (line == null || line.equals("")) return;

            String[] split = line.split("\\s+");
            if (name.contains("tb_a")) {
                String id = split[0];
                String city = split[1];
                context.write(new Text(id), new Text("#" + city));
            } else if (name.contains("tb_b")) {
                String id = split[0];
                String num1 = split[1];
                String num2 = split[2];
                context.write(new Text(id), new Text("$" + num1 + "\t" + num2));
            }
        }
    }

    public static class reducer extends Reducer<Text, Text, Text, Text> {
        @Override
        protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
            List<String> list1 = new LinkedList<>();
            List<String> list2 = new LinkedList<>();
            for (Text text : values) {
                String value = text.toString();
                if (value.startsWith("#")) {
                    value = value.substring(1);
                    list1.add(value);

                } else if (value.startsWith("$")) {
                    value = value.substring(1);
                    list2.add(value);
                }
            }
            for (String a : list1) {
                for (String b : list2) {
                    context.write(key, new Text(a + "\t" + b));
                }
            }
        }
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {


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