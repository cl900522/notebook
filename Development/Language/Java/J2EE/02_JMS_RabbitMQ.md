消息队列产品介绍与对比
===========

# MQ对比
  |ActiveMq   |RabbitMQ   |Kafka
--|---|---|--
所属社区  |Apache   |Mozilla Public License   |Apache/LinkedIn
开发语言  |Java   |Erlang   |Java
支持协议|OpenWiare,STOMP,REST,XMPP,AMQP   |AMQP   |仿AMQP
事务  |支持   |支持   |不支持
集群  |支持   |支持   |支持
负载均衡  |支持   |支持   |支持
动态扩展 |不支持   |不支持   |支持(ZK)

# RabbitMq
## 使用场景
RabbitMQ他是一个消息中间件，说道消息中间件【最主要的作用：信息的缓冲区】还是的从应用场景来看下:

1. 系统集成与分布式系统的设计
    各种子系统通过消息来对接，这种解决方案也逐步发展成一种架构风格，即“通过消息传递的架构”。

    举个例子：现在医院有两个科“看病科”和“住院科”在之前他们之间是没有任何关系的，如果你在“看病课”看完病后注册的信息和资料，到住院科后还得重新注册一遍？那现在要改革，你看完病后可以直接去住院科那两个系统之间需要打通怎么办？这里就可以使用我们的消息中间件了。

2. 异步任务处理结果回调的设计
    举个例子：记录日志，假如需要记录系统中所有的用户行为日志，如果通过同步的方式记录日志势必会影响系统的响应速度，当我们将日志消息发送到消息队列，记录日志的子系统就会通过异步的方式去消费日志消息。这样不需要同步的写入日志了NICE

3. 并发请求的压力高可用性设计
    举个例子：比如电商的秒杀场景。当某一时刻应用服务器或数据库服务器收到大量请求，将会出现系统宕机。如果能够将请求转发到消息队列，再由服务器去消费这些消息将会使得请求变得平稳，提高系统的可用性。

## 概念汇总
* ConnectionFactory、Connection、Channel
    ConnectionFactory、Connection、Channel都是RabbitMQ对外提供的API中最基本的对象。Connection是RabbitMQ的socket链接，它封装了socket协议相关部分逻辑。

    ConnectionFactory为Connection的制造工厂。

    Channel是我们与RabbitMQ打交道的最重要的一个接口，我们大部分的业务操作是在Channel这个接口中完成的，包括定义Queue、定义Exchange、绑定Queue与Exchange、发布消息等。

* exchange
    实际上生产者不会直接把消息放到消息队列里，实际的情况是，生产者将消息发送到Exchange（交换器，下图中的X），由Exchange将消息路由到一个或多个Queue中（或者丢弃）

* queue
    Queue（队列）是RabbitMQ的内部对象，用于存储消息，用下图表示。

    RabbitMQ中的消息都只能存储在Queue中，生产者（下图中的P）生产消息并最终投递到Queue中，消费者（下图中的C）可以从Queue中获取消息并消费。

* Message acknowledgment
    在实际应用中，可能会发生消费者收到Queue中的消息，但没有处理完成就宕机（或出现其他意外）的情况，这种情况下就可能会导致消息丢失。为了避免这种情况发生，我们可以要求消费者在消费完消息后发送一个回执给RabbitMQ，RabbitMQ收到消息回执（Message acknowledgment）后才将该消息从Queue中移除；如果RabbitMQ没有收到回执并检测到消费者的RabbitMQ连接断开，则RabbitMQ会将该消息发送给其他消费者（如果存在多个消费者）进行处理。 这里不存在timeout概念，一个消费者处理消息时间再长也不会导致该消息被发送给其他消费者，除非它的RabbitMQ连接断开。

    **这里会产生另外一个问题，如果我们的开发人员在处理完业务逻辑后，忘记发送回执给RabbitMQ，这将会导致严重的bug——Queue中堆积的消息会越来越多；消费者重启后会重复消费这些消息并重复执行业务逻辑…**

* Message durability
    如果我们希望即使在RabbitMQ服务重启的情况下，也不会丢失消息，我们可以将Queue与Message都设置为可持久化的（durable），这样可以保证绝大部分情况下我们的RabbitMQ消息不会丢失。但依然解决不了小概率丢失事件的发生（比如RabbitMQ服务器已经接收到生产者的消息，但还没来得及持久化该消息时RabbitMQ服务器就断电了），如果我们需要对这种小概率事件也要管理起来，那么我们要用到事务。由于这里仅为RabbitMQ的简单介绍，所以这里将不讲解RabbitMQ相关的事务。

* Prefetch count
    生产者在将消息发送给Exchange的时候，一般会指定一个routing key，来指定这个消息的路由规则，而这个routing key需要与Exchange Type及binding key联合使用才能最终生效。 在Exchange Type与binding key固定的情况下（在正常使用时一般这些内容都是固定配置好的），我们的生产者就可以在发送消息给Exchange时，通过指定routing key来决定消息流向哪里。 RabbitMQ为routing key设定的长度限制为255 bytes。

* Binding
    RabbitMQ中通过Binding将Exchange与Queue关联起来，这样RabbitMQ就知道如何正确地将消息路由到指定的Queue了。 在绑定（Binding）Exchange与Queue的同时，一般会指定一个binding key；生产者将消息发送给Exchange时，一般会指定一个routing key；当binding key与routing key相匹配时，消息将会被路由到对应的Queue中。这个将在Exchange Types章节会列举实际的例子加以说明。

    在绑定多个Queue到同一个Exchange的时候，这些Binding允许使用相同的binding key。 **binding key 并不是在所有情况下都生效，它依赖于Exchange Type，比如fanout类型的Exchange就会无视binding key，而是将消息路由到所有绑定到该Exchange的Queue。**

## 消息传输模式
RabbitMQ和一般的消息传输模式(队列模式&主题模式区别)

### 队列模式
![](/images/2018/09/queue_mode.png)
一个发布者发布消息，下面的接收者按队列顺序接收，比如发布了10个消息，两个接收者A,B那就是A,B总共会收到10条消息，不重复。

### 主题模式
![](/images/2018/09/topic_mode.png)
对于Topic模式，一个发布者发布消息，有两个接收者A,B来订阅，那么发布了10条消息，A,B各收到10条消息。

### RabbitMQ的模式(RabbitMQ独有)
![](/images/2018/09/rabbitmq_mode.png)
生产者生产消息后不直接直接发到队列中，而是发到一个交换空间：Exchange，Exchange会根据Exchange类型和Routing Key来决定发到哪个队列中，这个讲到发布订阅在详细来看

## 生产者消费模型

## 简单 - 单发单接收

```python
channel = connection.channel()
# 声明队列hello,RabbitMQ的消息队列机制如果队列不存在那么数据将会被丢掉,下面我们声明一个队列如果不存在创建
channel.queue_declare(queue='hello')
# 给队列中添加消息
channel.publish(exchange="",
                routing_key="hello",
                body="Hello World")
```

### 工作队列 - 单发送多接收
一个发送端，多个接收端，如分布式的任务派发。为了保证消息发送的可靠性，不丢失消息，使消息持久化了。同时为了防止接收端在处理消息时down掉，只有在消息处理完成后才发送ack消息。

场景描述：
1. 使用“task_queue”声明了另一个Queue，因为RabbitMQ不容许声明2个相同名称、配置不同的Queue
2. 使"task_queue"的Queue的durable的属性为true，即使消息队列durable
3. 使用MessageProperties.PERSISTENT_TEXT_PLAIN使消息durable

```java
//生产者
channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);

//消费者
channel.basicQos(1);
channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
```

### 发布-订阅
发布、订阅模式，发送端发送广播消息，多个接收端接收。

场景描述：
1. 发送消息到一个名为“logs”的exchange上，使用“fanout”方式发送，即广播消息，不需要使用queue，发送端不需要关心谁接收。
2. channel.queueDeclare().getQueue();该语句得到一个随机名称的Queue，该queue的类型为non-durable、exclusive、auto-delete的，将该queue绑定到上面的exchange上接收消息。
3. 注意binding queue的时候，channel.queueBind()的第三个参数Routing key为空，即所有的消息都接收。如果这个值不为空，在exchange type为“fanout”方式下该值被忽略！


```java
//生产者
channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes());

//消费者
String queueName = channel.queueDeclare().getQueue();
channel.queueBind(queueName, EXCHANGE_NAME, "");

QueueingConsumer consumer = new QueueingConsumer(channel);
channel.basicConsume(queueName, true, consumer);
```

### Router
场景描述：
1. exchange的type为direct
2. 发送消息的时候加入了routing key
在绑定queue和exchange的时候使用了routing key，即从该exchange上只接收routing key指定的消息。

```java
//生产者
channel.exchangeDeclare(EXCHANGE_NAME, "direct");
channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes());

//消费者
String queueName = channel.queueDeclare().getQueue();
channel.queueBind(queueName, EXCHANGE_NAME, severity);
QueueingConsumer consumer = new QueueingConsumer(channel);
channel.basicConsume(queueName, true, consumer);
```


### Topic
场景描述：
1. exchange的type为topic
2. 发送消息的routing key不是固定的单词，而是匹配字符串，如"*.lu.#"，*匹配一个单词，#匹配0个或多个单词。

```java
//生产者
channel.exchangeDeclare(EXCHANGE_NAME, "topic");
channel.basicPublish(EXCHANGE_NAME, routingKey, null, message.getBytes());

//消费者
String queueName = channel.queueDeclare().getQueue();
channel.queueBind(queueName, EXCHANGE_NAME, routingKey);
QueueingConsumer consumer = new QueueingConsumer(channel);
channel.basicConsume(queueName, true, consumer);
```

### 总结

模式  |说明   |图片介绍
--|---|--
简单模式 |消息产生者产生消息，消息的消费者进行消费      |![](/images/2018/09/mq_pc_basic.png)
工作模式 |消息消费产生消息，将消息发送到消息队列中，这是竞争，消费者1和消费者2都监听消息队列，当队列中有消息，一起来抢消息。谁抢到谁处理。      |![](/images/2018/09/mq_pc_worker.png)
发布和订阅|消息产生者产生消息，将消息发送到交换机中。多个消息队列绑定到交换机上。交换机将消息发送到多个队列中。消费者1监听自己的队列，如果有消息就进行消费。消费者2监听自己的队列，如果有消息进行消费。      |![](/images/2018/09/mq_pc_pub_sub.png)
路由模式  | 比发布订阅模式多了一个路由选择，称为路由key。路由key指定一个名称。队列在绑定到交换机时，还要设置这个路由key。消息的队列中不是所有的消息了，交换机会根据消息的路由key，选择性将消息传递给消息队列。    |![](/images/2018/09/mq_pc_router.png)
主题模式  | 在路由模式基础上，让路由key可以使用通配符。相当于进行分类。灵活程度更高些。**隐患：容易误伤。**    |![](/images/2018/09/mq_pc_topic.png)

## 持久化和公平分发
### 消息持久化
在实际应用中，可能会发生消费者收到Queue中的消息，但没有处理完成就宕机（或出现其他意外）的情况，这种情况下就可能会导致消息丢失。为了避免这种情况发生，我们可以要求消费者在消费完消息后发送一个回执给RabbitMQ，RabbitMQ收到消息回执（Message acknowledgment）后才将该消息从Queue中移除；如果RabbitMQ没有收到回执并检测到消费者的RabbitMQ连接断开，则RabbitMQ会将该消息发送给其他消费者（如果存在多个消费者）进行处理。这里不存在timeout概念，一个消费者处理消息时间再长也不会导致该消息被发送给其他消费者，除非它的RabbitMQ连接断开。 这里会产生另外一个问题，如果我们的开发人员在处理完业务逻辑后，忘记发送回执给RabbitMQ，这将会导致严重的bug——Queue中堆积的消息会越来越多；消费者重启后会重复消费这些消息并重复执行业务逻辑…千面是因为我们在消费者端标记了ACK=True关闭了它们，如果你没有增加ACK=True或者没有回执就会出现这个问题

### 队列持久化

如果我们希望即使在RabbitMQ服务重启的情况下，也不会丢失消息，我们可以将Queue与Message都设置为可持久化的（durable），这样可以保证绝大部分情况下我们的RabbitMQ消息不会丢失。但依然解决不了小概率丢失事件的发生（比如RabbitMQ服务器已经接收到生产者的消息，但还没来得及持久化该消息时RabbitMQ服务器就断电了），如果我们需要对这种小概率事件也要管理起来，那么我们要用到事务。由于这里仅为RabbitMQ的简单介绍，所以这里将不讲解RabbitMQ相关的事务。 这里我们需要修改下生产者和消费者设置RabbitMQ消息的持久化**[生产者/消费者]都需要配置**

### 公平分发
默认情况下RabitMQ会把队列里面的消息立即发送到消费者，无论该消费者有多少消息没有应答，也就是说即使发现消费者来不及处理，新的消费者加入进来也没有办法处理已经堆积的消息，因为那些消息已经被发送给老消费者了。类似下面的

在消费者中增加:
```python
channel.basic_qos(prefetch_count=1)
```

prefetchCount：会告诉RabbitMQ不要同时给一个消费者推送多于N个消息，即一旦有N个消息还没有ack，则该consumer将block掉，直到有消息ack。 这样做的好处是，如果系统处于高峰期，消费者来不及处理，消息会堆积在队列中，新启动的消费者可以马上从队列中取到消息开始工作。
![](/images/2018/09/send_fairly.png)

工作过程如下:
1. 消费者1接收到消息后处理完毕发送了ack并接收新的消息并处理
2. 消费者2接收到消息后处理完毕发送了ack并接收新的消息并处理
3. 消费者3接收到消息后一直处于消息中并没有发送ack不在接收消息一直等到消费者3处理完毕后发送ACK后再接收新消息

## 头交换机
### 概述

header exchange(头交换机)和主题交换机有点相似，但是不同于主题交换机的路由是基于路由键，头交换机的路由值基于消息的header数据。主题交换机路由键只有是字符串,而头交换机可以是整型和哈希值，header Exchange类型用的比较少。

---------------------

本文来自 hry2015 的CSDN 博客 ，全文地址请点击：https://blog.csdn.net/hry2015/article/details/79188615?utm_source=copy

## 资源
RabbitMQ核心概念篇
https://www.cnblogs.com/luotianshuai/p/7469365.html

RabbitMQ的几种典型使用场景
http://www.cnblogs.com/luxiaoxun/p/3918054.html

RabbitMq的整理 exchange、route、queue关系
http://blog.csdn.net/samxx8/article/details/47417133

官方Turtorial
http://www.rabbitmq.com/tutorials/tutorial-two-java.html

# ActiveMq

# Kafka

http://www.cnblogs.com/luotianshuai/p/5206662.html
