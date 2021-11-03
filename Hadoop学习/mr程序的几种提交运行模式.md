# MR程序的提交运行模式



1、在Windows的eclipse里面直接运行main方法，就会将job提交给本地执行器localjobrunner执行

​		----输入输出数据可以放在本地路径下(L:/.../src)，也可以放在hdfs中(hdfs://192.168.79.129:9000/wc/src)



2、在Linux的eclipse里面直接运行main方法，但是不要添加yurn相关配置，也会提交给localjobrunner执行

​		----输入输出数据可以放在本地路径下(L:/.../src)，也可以放在hdfs中(hdfs://192.168.79.129:9000/wc/src)



集群模式运行

1、将工程打成jar包，上传到服务器，然后用Hadoop命令提交 hadoop jar wc.jar xyz.liujiawei.hadoop.mr.wordcount.WCRunner

2、在Linux的eclipse中直接运行main方法，也可以提交到集群中去运行，但是必须采取以下措施：

​		----在工程src目录下加入mapred-site.xml 和 yarn-site.xml

​		----将工程打成jar包(wc.jar)，同时在main方法中添加一个conf配置参数  conf.set("mapreduce.job.jar","wc.jar")