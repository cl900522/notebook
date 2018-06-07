Spark Introduction
====================
>http://spark.apachecn.org/docs/cn/2.2.0/
# Spark Core

基于Spark通用计算平台，可以很好地扩展各种计算类型的应用，尤其是Spark提供了内建的计算库支持，像Spark Streaming、Spark SQL、MLlib、GraphX，这些内建库都提供了高级抽象，可以用非常简洁的代码实现复杂的计算逻辑、这也得益于Scala编程语言的简洁性。这里，我们基于1.3.0版本的Spark搭建了计算平台，实现基于Spark Streaming的实时计算。
我们的应用场景是分析用户使用手机App的行为，描述如下所示：

手机客户端会收集用户的行为事件（我们以点击事件为例），将数据发送到数据服务器，我们假设这里直接进入到Kafka消息队列
后端的实时服务会从Kafka消费数据，将数据读出来并进行实时分析，这里选择Spark Streaming，因为Spark Streaming提供了与Kafka整合的内置支持
经过Spark Streaming实时计算程序分析，将结果写入Redis，可以实时获取用户的行为数据，并可以导出进行离线综合统计分析

Spark支持两种类型操作：Transformations和Actions。
1. Transformation从一个已知的RDD数据集经过转换得到一个新的RDD数据集，这些Transformation操作包括map、filter、flatMap、union、join等，而且Transformation具有lazy的特性，调用这些操作并没有立刻执行对已知RDD数据集的计算操作，而是在调用了另一类型的Action操作才会真正地执行。
2. Action执行，会真正地对RDD数据集进行操作，返回一个计算结果给Driver程序，或者没有返回结果，如将计算结果数据进行持久化，Action操作包括reduceByKey、count、foreach、collect等。关于Transformations和Actions更详细内容，可以查看官网文档。

同样、Spark Streaming提供了类似Spark的两种操作类型，分别为Transformations和Output操作，它们的操作对象是DStream，作用也和Spark类似：
1. Transformation从一个已知的DStream经过转换得到一个新的DStream，而且Spark Streaming还额外增加了一类针对Window的操作，当然它也是Transformation，但是可以更灵活地控制DStream的大小（时间间隔大小、数据元素个数），例如window(windowLength, slideInterval)、countByWindow(windowLength, slideInterval)、reduceByWindow(func, windowLength, slideInterval)等。
2. Spark Streaming的Output操作允许我们将DStream数据输出到一个外部的存储系统，如数据库或文件系统等，执行Output操作类似执行Spark的Action操作，使得该操作之前lazy的Transformation操作序列真正地执行。

# Spark Stream
为了初始化Spark Streaming程序，一个StreamingContext对象必需被创建，它是Spark Streaming所有流操作的主要入口。一个StreamingContext 对象可以用SparkConf对象创建。 可以使用SparkConf对象创建JavaStreamingContext对象：

```java
SparkConf conf = new SparkConf().setAppName(appName).setMaster(master);
JavaStreamingContext jsc = new JavaStreamingContext(conf, Durations.seconds(seconds));
```

appName参数是应用程序在集群UI上显示的名称。 master是Spark，Mesos或YARN集群URL，或者是以本地模式运行的特殊字符串"local \[*\]"。

实际上，当在集群上运行时，您不想在程序中硬编码master(即在程序中写死)，而是希望使用spark-submit启动应用程序时得到master的值。 但是，对于本地测试和单元测试，您可以传递"local \[*\]"来运行Spark Streaming进程。 注意，这里内部创建的JavaSparkContext（所有Spark功能的起始点），可以通过jsc.sparkContext访问。

JavaStreamingContext对象也可以从现有的JavaSparkContext创建：

```java
SparkConf conf = new SparkConf().setAppName("socket-spark-stream").setMaster("local[2]");
JavaSparkContext sparkContext = new JavaSparkContext(conf);
JavaStreamingContext jsc = new JavaStreamingContext(sparkContext, Durations.seconds(seconds));
```

批处理间隔必须根据应用程序和可用群集资源的延迟要求进行设置。 有关更多详细信息，请参阅“性能调优”部分。

定义上下文后，您必须执行以下操作：
* 通过创建输入DStreams定义输入源
* 通过对DStreams应用转换操作（transformation）和输出操作（output）来定义流计算
* 可以使用streamingContext.start()方法接收和处理数据
* 可以使用streamingContext.awaitTermination()方法等待流计算完成（手动或由于任何错误），来防止应用退出
* 可以使用streamingContext.stop（）手动停止处理。

注意点:
* 一旦上下文已经开始，则不能设置或添加新的流计算。
* 上下文停止后，无法重新启动。
* 在同一时间只有一个StreamingContext可以在JVM中处于活动状态。
* 在StreamingContext上调用stop()方法，也会关闭SparkContext对象。如果只想关闭StreamingContext对象，设置stop()的可选参数为false。
* 一个SparkContext可以重复利用创建多个StreamingContext，只要在创建下一个StreamingContext之前停止前一个StreamingContext（而不停止SparkContext）即可。

# Spark SQL
