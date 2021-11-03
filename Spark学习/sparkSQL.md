## SQL风格

```scala
> var df = spark.read.json("file:///home/liujiawei/2.json")

> df.show

> df.printSchema

> df.createTempView("student")

> spark.sql("select * from student").show()
```

## DSL风格

```scala
> df.select("name").show()

> df.select($"age"+1).show

> df.filter($"age">=21).show

> df.groupBy("age").count.show
```

## RDD转DF

```scala
> val rdd = sc.makeRDD( List((1,"zhngsan",20),(2,"lisi",30),(3,"wangwu",40) ))

> df = rdd.toDF("id","name","age")

> df.show
```

## 对象反射创建RDD

```scala
> val rdd = sc.makeRDD(List(("zhangsan",20),("lisi",30),("wangwu",40)))

> val peopleRDD = rdd.map(t=>{People(t._1,t._2)})

> df = peopleRDD.toDF
```

## 创建DATASET

```scala
> val caseClassDS = Seq(People("Andy",32)).toDS
```

## RDD转换为DATASET

```scala
> val rdd = sc.makeRDD(List(("zhangsan",20),("lisi",30),("wangwu",40)))

> val peopleRDD = rdd.map(t=>{People(t._1,t._2)})

> peopleRDD.toDS
```

## DATASET转换为RDD

```scala
> val ds = mapRDD.toDS

> ds.rdd
```

## DATAFRAME转DATASET

```scala
> df.as[Person]
```

## DATASET转DATAFRAME

```scala
> ds.toDF
```

## 保存数据

```scala
> df.write.format("json").mode("append").save("file:///home/liujiawei/output")
```

