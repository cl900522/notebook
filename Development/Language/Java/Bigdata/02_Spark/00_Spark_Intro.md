Spark Introduction
====================
>http://spark.apachecn.org/docs/cn/2.2.0/

# Spark Core
## 基本概念
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

## Spark Context:
Prior to Spark 2.0.0 sparkContext was used as a channel to access all spark functionality.
The spark driver program uses spark context to connect to the cluster through a resource manager (YARN orMesos..).
sparkConf is required to create the spark context object, which stores configuration parameter like appName (to identify your spark driver), application, number of core and memory size of executor running on worker node.

In order to use APIs of SQL, HIVE, and Streaming, separate contexts need to be created.

## Spark Session:

SPARK 2.0.0 onwards, SparkSession provides a single point of entry to interact with underlying Spark functionality and
allows programming Spark with DataFrame and Dataset APIs. All the functionality available with sparkContext are also available in sparkSession.

In order to use APIs of SQL, HIVE, and Streaming, no need to create separate contexts as sparkSession includes all the APIs.

Once the SparkSession is instantiated, we can configure Spark’s run-time config properties.

## conf context session https://www.cnblogs.com/Forever-Road/p/7351245.html

## Master 参数介绍
我们在初始化SparkConf时，或者提交Spark任务时，都会有master参数需要设置，如下：
```java
conf = SparkConf().setAppName(appName).setMaster(master)
sc = SparkContext(conf=conf)

```

```shell
/bin/spark-submit \
        --cluster cluster_name \
        --master yarn-cluster \
        ...
```
但是这个master到底是何含义呢？文档说是设定master url，但是啥是master url呢？说到这就必须先要了解下Spark的部署方式了。

我们要部署Spark这套计算框架，有多种方式，可以部署到一台计算机，也可以是多台(cluster)。我们要去计算数据，就必须要有计算机帮我们计算，当然计算机越多(集群规模越大)，我们的计算力就越强。但有时候我们只想在本机做个试验或者小型的计算，因此直接部署在单机上也是可以的。Spark部署方式可以用如下图形展示：

![](/images/2018/06/spark-mode.jpg)

### Local模式
Local模式就是运行在一台计算机上的模式，通常就是用于在本机上练手和测试。它可以通过以下集中方式设置master。

1. local: 所有计算都运行在一个线程当中，没有任何并行计算，通常我们在本机执行一些测试代码，或者练手，就用这种模式。
2. local\[K\]: 指定使用几个线程来运行计算，比如local[4]就是运行4个worker线程。通常我们的cpu有几个core，就指定几个线程，最大化利用cpu的计算能力
3. local\[*\]: 这种模式直接帮你按照cpu最多cores来设置线程数了。

总而言之这几种local模式都是运行在本地的单机版模式，通常用于练手和测试，而实际的大规模计算就需要下面要介绍的cluster模式。

### cluster模式
1. standalone模式
    这种模式下，Spark会自己负责资源的管理调度。它将cluster中的机器分为master机器和worker机器，master通常就一个，可以简单的理解为那个后勤管家，worker就是负责干计算任务活的苦劳力。具体怎么配置可以参考Spark Standalone Mode
    使用standalone模式示例：
    ```shell
    /bin/spark-submit \
            --cluster cluster_name \
            --master spark://host:port \
            ...
    ```
    --master就是指定master那台机器的地址和端口，我想这也正是--master参数名称的由来吧

2. mesos模式
    这里就很好理解了，如果使用mesos来管理资源调度，自然就应该用mesos模式了，示例如下：
    ```shell
    /bin/spark-submit \
            --cluster cluster_name \
            --master mesos://host:port \
            ...
    ```

3. yarn模式
    同样，如果采用yarn来管理资源调度，就应该用yarn模式，由于很多时候我们需要和mapreduce使用同一个集群，所以都采用Yarn来管理资源调度，这也是生产环境大多采用yarn模式的原因。yarn模式又分为yarn cluster模式和yarn client模式：

    yarn cluster: 这个就是生产环境常用的模式，所有的资源调度和计算都在集群环境上运行。
    yarn client: 这个是说Spark Driver和ApplicationMaster进程均在本机运行，而计算任务在cluster上。
    使用示例：

    ```shell
    /bin/spark-submit \
            --cluster cluster_name \
            --master yarn-cluster \
            ...
    ```

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
