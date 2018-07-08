FAQ
=====
1. 任务提交master运行
描述： 代码从本地运行模式修改为提交到集群master运行，会产生类的序列化问题
```java
SparkConf sparkConf = new SparkConf().setAppName("JavaWordCount").setMaster("local[2]");﻿​
```
修改为
```java
SparkConf sparkConf = new SparkConf().setAppName("JavaWordCount").setMaster("spark://192.168.168.200:7077");
```
会产生以下错误
```log
Exception in thread "main" org.apache.spark.SparkException: Job aborted due to stage failure: Task 1 in stage 0.0 failed 4 times, most recent failure: Lost task 1.3 in stage 0.0 (TID 6, 192.168.168.200): java.lang.ClassCastException: cannot assign instance of scala.collection.immutable.List$SerializationProxy to field org.apache.spark.rdd.RDD.org$apache$spark$rdd$RDD$$dependencies_ of type scala.collection.Seq in instance of org.apache.spark.rdd.MapPartitionsRDD
	at java.io.ObjectStreamClass$FieldReflector.setObjFieldValues(ObjectStreamClass.java:2083)
```

修复方案： 增加指定jar包文件都中
```java
SparkConf sparkConf = new SparkConf().setAppName("JavaWordCount").setMaster("spark://192.168.168.200:7077");
String [] jars = {"I:\\TestSpark\\target\\TestSpark-0.0.1-jar-with-dependencies.jar"};
sparkConf.setJars(jars);
```
http://blog.leanote.com/post/2630794313@qq.com/Eclipse%E6%8F%90%E4%BA%A4%E4%BB%A3%E7%A0%81%E5%88%B0
