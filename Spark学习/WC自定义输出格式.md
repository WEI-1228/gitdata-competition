# 自定义输出格式

```java
public class MR2 {

    public static class mapper extends Mapper<LongWritable, Text, Text, Text> {
        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            String line = value.toString();
            String[] split = line.split(",");
            context.write(new Text(split[0]), value);
        }
    }

    public static class reducer extends Reducer<Text, Text, Text, Text> {
        @Override
        protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
            StringBuilder builder = new StringBuilder();
            for (Text value : values) {
                String line = value.toString();
                String[] split = line.split(",");
                String name = split[1];
                double sum = 0;
                for (int i = 2; i < split.length; i++)
                    sum += Integer.parseInt(split[i]);
                sum = sum / (split.length - 2);
                builder.append(name).append("%").append(String.format("%.2f", sum)).append("#");
            }
            context.write(key, new Text(builder.toString()));
        }
    }

    /**
     * 自定义输出格式
     */
    public static class myFileOutPutClass extends FileOutputFormat<Text, Text> {

        @Override
        public RecordWriter<Text, Text> getRecordWriter(TaskAttemptContext taskAttemptContext) throws IOException, InterruptedException {
            return new myRecordWriter(taskAttemptContext);
        }
    }

    /**
     * 输出格式控制类，两个泛型对应的是reduce输出的内容，相当于reduce输出之后再到这里进行进一步处理
     */
    public static class myRecordWriter extends RecordWriter<Text, Text> {

        FileSystem fs;

        public myRecordWriter(TaskAttemptContext taskAttemptContext) throws IOException {
            fs = FileSystem.get(taskAttemptContext.getConfiguration());
        }

        @Override
        public void write(Text key, Text value) throws IOException, InterruptedException {
            //定义输出路径
            String out = "L:\\JAVA\\Hadoop\\src\\xyz\\practice\\p3\\src\\result\\" + key.toString() + ".txt";
            Path outPath = new Path(out);
            //创建一个输出流对象
            FSDataOutputStream fos = fs.create(outPath);
            String data = value.toString();

            String[] split = data.split("#");
            List<String[]> list = new LinkedList<>();

            for (String s : split) {
                String[] strings = s.split("%");
                list.add(strings);
            }
            list.sort(((o1, o2) -> (int) (Double.parseDouble(o2[1]) - Double.parseDouble(o1[1]))));
            for (String[] strings : list) {
                String res = strings[0] + "\t" + strings[1] + "\n";
                //当作IO流直接输出内容到一个文件
                fos.write(res.getBytes());
            }
            IOUtils.closeStream(fos);
        }

        @Override
        public void close(TaskAttemptContext taskAttemptContext) throws IOException, InterruptedException {
            fs.close();
        }
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(MR2.class);
        job.setMapperClass(mapper.class);
        job.setReducerClass(reducer.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);
        job.setOutputFormatClass(myFileOutPutClass.class);
        FileSystem fileSystem = FileSystem.get(conf);

        Path outputPath = new Path("L:\\JAVA\\Hadoop\\src\\xyz\\practice\\p3\\src\\output1");
        if (fileSystem.exists(outputPath))
            fileSystem.delete(outputPath, true);

        FileInputFormat.setInputPaths(job, new Path("L:\\JAVA\\Hadoop\\src\\xyz\\practice\\p3\\src\\input"));
        FileOutputFormat.setOutputPath(job, outputPath);

        System.exit(job.waitForCompletion(true) ? 0 : -1);

    }
}
```

