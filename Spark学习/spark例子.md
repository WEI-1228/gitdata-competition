# Spark例子

**wordcount**

```scala
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}
object ScalaWC {
  def main(args: Array[String]): Unit = {
    //conf 可以设置SparkApplication 的名称，设置Spark 运行的模式
    val conf = new SparkConf()
    conf.setAppName("wordcount")
    conf.setMaster("local")
    //SparkContext 是通往spark集群的唯一通道
    val sc = new SparkContext(conf)
    val lines: RDD[String] = sc.textFile("./data/word.txt")
    val words: RDD[String] = lines.flatMap(line => {
      line.split(" ")
    })
    val pairWords: RDD[(String, Int)] = words.map(word => {
      Tuple2(word, 1)
    })
    val result: RDD[(String, Int)] = pairWords.reduceByKey((v1: Int, v2: Int) => {
      v1 + v2
    })
    result.foreach((one) => {
      println(one)
    })
  }
}
```

**SparkSQL_Demo1**

```scala
import org.apache.spark.sql.{DataFrame, SparkSession}
object SparkSQL_02_SQL {
  def main(args: Array[String]): Unit = {
    //创建sparkSQL的环境对象
    val spark: SparkSession = SparkSession.builder().master("local[*]").appName("demo1").getOrCreate()
    //读取数据，构建DataFrame
    val frame: DataFrame = spark.read.json("data/user.json")
    //展示数据
    //frame.show()
    //将DataFrame转化为一张表
    frame.createOrReplaceTempView("user")
    spark.sql("select * from user").show
    //释放资源
    spark.stop()
  }
}
```

**SparkSQL_Demo2**

```scala
import org.apache.spark.SparkContext
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.{DataFrame, Dataset, Row, SparkSession}
object SparkSQL_03_Transform {
  def main(args: Array[String]): Unit = {
    //创建sparkSQL的环境对象
    val spark: SparkSession = SparkSession.builder().master("local[*]").appName("demo1").getOrCreate()
    //进行隐式转换之前，需要引入隐式转换规则
    //这里的spark不是包名的含义，是SparkSession对象的名字
    import spark.implicits._
    //创建RDD
    val sc: SparkContext = spark.sparkContext
    val rdd: RDD[(Int, String, Int)] = sc.makeRDD(List((1, "zhangsan", 20), (2, "lisi", 30), (3, "wangwu", 40)))
    //转换为DF
    val df: DataFrame = rdd.toDF("id","name","age")
    //转换为DS
    val ds: Dataset[User] = df.as[User]
    //转换为DF
    val df1: DataFrame = ds.toDF()
    //转换为RDD
    val rdd1: RDD[Row] = df1.rdd

    rdd1.foreach(row=>{
      //获取数据时可以通过索引访问数据
      println(row.getInt(0))
    })
    //释放资源
    spark.stop()
  }
}
case class User(id:Int,name:String,age:Int)
```

**sparkSQL_Demo3**

```scala
import org.apache.spark.SparkContext
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.{DataFrame, Dataset, Row, SparkSession}
object SparkSQL_04_Transform {
  def main(args: Array[String]): Unit = {
    //创建sparkSQL的环境对象
    val spark: SparkSession = SparkSession.builder().master("local[*]").appName("demo1").getOrCreate()
    //进行隐式转换之前，需要引入隐式转换规则
    //这里的spark不是包名的含义，是SparkSession对象的名字
    import spark.implicits._
    //创建RDD
    val sc: SparkContext = spark.sparkContext
    val rdd: RDD[(Int, String, Int)] = sc.makeRDD(List((1, "zhangsan", 20), (2, "lisi", 30), (3, "wangwu", 40)))
      
    // RDD - DataSet
    val userRDD: RDD[User] = rdd.map {
      case (id, name, age) =>
        User(id, name, age)
    }
    val userDS: Dataset[User] = userRDD.toDS()
    val rdd1: RDD[User] = userDS.rdd
    rdd1.foreach(println)
    //释放资源
    spark.stop()
  }
}
```

**sparkSQL_Demo4**

```scala
import org.apache.hadoop.fs.{FileSystem, Path}
import org.apache.spark.SparkContext
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.{DataFrame, Row, SparkSession}

object spark_sql {

  def pathIsExist(spark: SparkSession, path: String): Boolean = {
    //取文件系统
    val filePath: Path = new org.apache.hadoop.fs.Path(path)
    val fileSystem: FileSystem = filePath.getFileSystem(spark.sparkContext.hadoopConfiguration)

    // 判断路径是否存在
    fileSystem.exists(filePath)
  }

  def deletePath(spark: SparkSession, path: String): Unit = {
    // 1 获取文件系统
    val file_path: Path = new org.apache.hadoop.fs.Path( path )
    val file_system: FileSystem = file_path.getFileSystem( spark.sparkContext.hadoopConfiguration )

    // 2 判断路径存在时, 则删除
    if (file_system.exists( file_path )) {
      file_system.delete( file_path, true )
    }
  }

  def main(args: Array[String]): Unit = {
    val spark: SparkSession = SparkSession.builder().appName("app").master("local[*]").getOrCreate()
    val sc: SparkContext = spark.sparkContext
    import spark.implicits._
    val courseText: RDD[String] = sc.textFile("data/course.txt")
    val scoreText: RDD[String] = sc.textFile("data/score.txt")
    val studentText: RDD[String] = sc.textFile("data/students.txt")

    val courseRDD: RDD[Courses] = courseText.map(line => {
      val strings: Array[String] = line.split(",")
      Courses(strings(0), strings(1).toInt)
    })

    val courseDF: DataFrame = courseRDD.toDF()
    courseDF.createOrReplaceTempView("course")
    val scoreRDD: RDD[Scores] = scoreText.map(line => {
      val strings: Array[String] = line.split(",")
      Scores(strings(0), strings(1), strings(2).toInt)
    })

    val scoreDF: DataFrame = scoreRDD.toDF()
    scoreDF.createOrReplaceTempView("score")
    val studentRDD: RDD[Students] = studentText.map(line => {
      val strings: Array[String] = line.split(",")
      Students(strings(0), strings(1), strings(2).toInt, strings(3), strings(4))
    })

    val studentDF: DataFrame = studentRDD.toDF()
    studentDF.createOrReplaceTempView("student")

    val frame: DataFrame = spark.sql("select id,name from student limit 10")
    val rdd: RDD[Row] = frame.rdd

    val outputPath: String = "output"
    deletePath(spark, outputPath)

    rdd.saveAsTextFile(outputPath)

    spark.stop()

  }

}

//语文,150
case class Courses(subject: String, fullScore: Int)

//1500100001,施笑槐,22,女,文科六班
case class Students(id: String, name: String, age: Int, sex: String, classNum: String)

//1500100001,语文,98
case class Scores(id: String, subject: String, score: Int)
```

