JMS概述
======
# 什么是JMS

JMS即Java消息服务（Java Message Service）应用程序接口，是一个Java平台中关于面向消息中间件（MOM）的API，用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。Java消息服务是一个与具体平台无关的API，绝大多数MOM提供商都对JMS提供支持（百度百科给出的概述）。我们可以简单的理解：两个应用程序之间需要进行通信，我们使用一个JMS服务，进行中间的转发，通过JMS 的使用，我们可以解除两个程序之间的耦合。

# JMS的优势
1. Asynchronous（异步）
JMS is asynchronous by default. So to receive a message, the client is not required to send the request. The message will arrive automatically to the client as they become available.
JMS 原本就是一个异步的消息服务，客户端获取消息的时候，不需要主动发送请求，消息会自动发送给可用的客户端

2. Reliable（可靠）
JMS provides the facility of assurance that the message will delivered once and only once. You know that duplicate messages create problems. JMS helps you avoiding such problems.
JMS保证消息只会递送一次。大家都遇到过重复创建消息问题，而JMS能帮你避免该问题。

# JMS的消息模型

JMS具有两种通信模式：
1. Point-to-Point Messaging Domain （点对点，不可重复消费）
2. Publish/Subscribe Messaging Domain （发布/订阅模式，可以重复消费）

在JMS API出现之前，大部分产品使用“点对点”和“发布/订阅”中的任一方式来进行消息通讯。JMS定义了这两种消息发送模型的规范，它们相互独立。任何JMS的提供者可以实现其中的一种或两种模型，这是它们自己的选择。JMS规范提供了通用接口保证我们基于JMS API编写的程序适用于任何一种模型。

## Point-to-Point Messaging Domain（点对点通信模型）

![](/images/2017/09/point-to-point.png)

### 涉及到的概念
在点对点通信模式中，应用程序由消息队列，发送方，接收方组成。每个消息都被发送到一个特定的队列，接收者从队列中获取消息。队列保留着消息，直到他们被消费或超时。

### 特点
每个消息只要一个消费者发送者和接收者在时间上是没有时间的约束，也就是说发送者在发送完消息之后，不管接收者有没有接受消息，都不会影响发送方发送消息到消息队列中。
发送方不管是否在发送消息，接收方都可以从消息队列中去到消息（The receiver can fetch message whether it is running or not when the sender sends the message）
接收方在接收完消息之后，需要向消息队列应答成功

## Publish/Subscribe Messaging Domain（发布/订阅通信模型）


![](/images/2017/09/publish-subscribe.png)

### 涉及到的概念
在发布/订阅消息模型中，发布者发布一个消息，该消息通过topic传递给所有的客户端。该模式下，发布者与订阅者都是匿名的，即发布者与订阅者都不知道对方是谁。并且可以动态的发布与订阅Topic。Topic主要用于保存和传递消息，且会一直保存消息直到消息被传递给客户端。

### 特点
一个消息可以传递个多个订阅者（即：一个消息可以有多个接受方）
发布者与订阅者具**有时间约束**，针对某个主题（Topic）的订阅者，它必须创建一个订阅者之后，才能消费发布者的消息，而且为了消费消息，订阅者必须保持运行的状态。
为了缓和这样严格的时间相关性，JMS允许订阅者创建一个可持久化的订阅。这样，即使订阅者没有被激活（运行），它也能接收到发布者的消息。

# JMS接收消息
在JMS中，消息的产生和消息是异步的。对于消费来说，JMS的消息者可以通过两种方式来消费消息。

1. 同步（Synchronous）
在同步消费信息模式模式中，订阅者/接收方通过调用 receive（）方法来接收消息。在receive（）方法中，线程会阻塞直到消息到达或者到指定时间后消息仍未到达。

2. 异步（Asynchronous）
使用异步方式接收消息的话，消息订阅者需注册一个消息监听者，类似于事件监听器，只要消息到达，JMS服务提供者会通过调用监听器的onMessage()递送消息。

# push/pull
消息系统必不可少的就是消息的 push 和 pull了。事实上，push模式和pull模式各有优劣。比如Facebook的Scribe和Cloudera的Flume，AMQ 采用push模式。
Kafka这种新型的消息队列采用2种模式，选择由Producer向broker push消息并由Consumer从broker pull消息。

## push模式
在push方式里，是由consumer把轮询过程封装了，并注册MessageListener监听器，取到消息后，唤醒MessageListener的consumeMessage()来消费，对用户而言，感觉消息是被推送过来的。常见的push模式如storm的消息处理，由spout负责消息的推送。该模式下需要一个中心节点，负责消息的分配情况（哪段消息分配给consumer1，哪段消息分配给consumer2），同时还要监听consumer的ack消息用于判断消息是否处理成功，如果在timeout时间内为收到响应可以认为该consumer挂掉，需要重新分配sonsumer上失败的消息。而且容易冲垮consumer，虽然可以向amq那样使用prefetch limit  和预留缓冲区 ，但是还是不能解决由consumer的选择性消费。

### 使用场景
push模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。push模式的目标是尽可能以最快速度传递消息，但是这样很容易造成Consumer来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而pull模式则可以根据Consumer的消费能力以适当的速率消费消息。

## pull模式
取消息的过程需要用户自己写，根据offset取得消息，kafka中的offset是有zk管理(具体不叙述细节)。由consumer决定消息的消费情况，这种模式有一个好处是我们不需要返回ack消息，因为当consumer申请消费下一批消息时就可以认为上一批消息已经处理完毕，也不需要处理超时的问题，consumer可以根据自己的消费能力来消费消息。但是pull最大问题是实时性问题，对于现在的MQ系统，对latency的追求越来越高，这个时候纯粹的Pull模式将满足不了实时性方面的需求。这时可以采用动态push/pull方式。consumer在pull的时候，告诉broker自己buffer中可用的容量。

整个流程如下：
1、consumer请求broker，告诉broker本地的可承载量，比如500
2、broker在收到消息后，如果没有消息则进入long polling状态
3、当有消息的时候，broker直接向consumer进行push，总共push的数据量为500
4、在整个push期间，consumer无需重新pull，即可获取数据
5、由于broker知道最大容量，所以无需担心被冲垮。这样也不用担心consumer 被冲垮。

### 使用场景
简化Broker的设计，consuemr根据自己的消费能力以适当的速率消费信息；

## 对比
无论是消息系统，还是配置管理中心，甚至存储系统，你都要面临这样一个选择，push模型 or pull模型?是服务端主动给客户端推送数据，还是客户端去服务器拉数据，一张图表对比如下：
   |push模型   |pull模型   |
--|---|---|--
描述  |服务端主动发送数据给客户端   |客户端主动从服务端拉取数据，通常客户端会定时拉取   |
实时性  |较好，收到数据后可立即发送给客户端   |一般，取决于pull的间隔时间   |
服务端状态 |需要保存push状态，哪些客户端已经发送成功，哪些发送失败   |服务端无状态   |
客户端状态  |无需额外保存状态   |需保存当前拉取的信息的状态，以便在故障或者重启的时候恢复   |
状态保存  |集中式，集中在服务端   |分布式，分散在各个客户端   |
负载均衡  |服务端统一处理和控制   | 客户端之间做分配，需要协调机制，如使用zookeeper  |
其他  |服务端需要做流量控制，无法最大化客户端的处理能力。 其次，在客户端故障情况下，无效的push对服务端有一定负载。 |客户端的请求可能很多无效或者没有数据可供传输，浪费带宽和服务器处理能力   |
缺点方案  |服务器端的状态存储是个难点，可以将这些状态转移到DB或者key-value存储，来减轻server压力。   |针对实时性的问题，可以将push加入进来，push小数据的通知信息，让客户端再来主动pull。
    针对无效请求的问题，可以设置逐渐延长间隔时间的策略，以及合理设计协议尽量缩小请求数据包来节省带宽。   |
流转机制    |服务端需要依据订阅者消费能力做流控 |消费端可以根据自身消费能力决定是否pull

## 两种模式结合
将信息推送与拉取两种模式结合能做到取长补短，使二者优势互补。根据推、拉结合顺序及结合方式的差异，又分以下四种不同推拉模式：
* 先推后拉——先由信源及时推送公共信息，再由用户有针对性地拉取个性化信息；
* 先拉后推——根据用户拉取的信息，信源进一步主动提供(推送)与之相关的信息；
* 推中有拉——在信息推送过程中，允许用户随时中断并定格在感兴趣的网页上，以拉取更有针对性的信息；
* 拉中有推——根据用户搜索(即拉取)过程中所用的关键字，信源主动推送相关的最新信息。

# JMS编程模型
1. 管理对象（Administered objects）-连接工厂（Connection -
2. Factories）和目的地（Destination）
3. 连接对象（Connections）
4. 会话（Sessions）
5. 消息生产者（Message Producers）
6. 消息消费者（Message Consumers）
7. 消息监听者（Message Listeners）

![](/images/2017/09/mq-struct.png)

## 连接工厂

创建Connection对象的工厂，针对两种不同的jms消息模型，分别有QueueConnectionFactory和TopicConnectionFactory两种。可以通过JNDI来查找ConnectionFactory对象。客户端使用一个连接工厂对象连接到JMS服务提供者，它创建了JMS服务提供者和客户端之间的连接。JMS客户端（如发送者或接受者）会在JNDI名字空间中搜索并获取该连接。使用该连接，客户端能够与目的地通讯，往队列或话题发送/接收消息。

```java
QueueConnectionFactory queueConnFactory = (QueueConnectionFactory) initialCtx.lookup ("primaryQCF");
Queue purchaseQueue = (Queue) initialCtx.lookup ("Purchase_Queue");
Queue returnQueue = (Queue) initialCtx.lookup ("Return_Queue");
```

## 目的地

目的地指明消息被发送的目的地以及客户端接收消息的来源。JMS使用两种目的地，队列和话题。如下代码指定了一个队列和话题：
创建一个队列Session：
```java
QueueSession ses = con.createQueueSession (false, Session.AUTO_ACKNOWLEDGE);  //get the Queue object
Queue t = (Queue) ctx.lookup ("myQueue");  //create QueueReceiver
QueueReceiver receiver = ses.createReceiver(t);
```

创建一个Topic Session：
```java
QueueSession ses = con.createQueueSession (false, Session.AUTO_ACKNOWLEDGE);  //get the Queue object
Queue t = (Queue) ctx.lookup ("myQueue");  //create QueueReceiver
QueueReceiver receiver = ses.createReceiver(t);
```

## JMS连接
Connection表示在客户端和JMS系统之间建立的链接（对TCP/IP socket的包装）。Connection可以产生一个或多个Session。跟ConnectionFactory一样，Connection也有两种类型：QueueConnection和TopicConnection。

连接对象封装了与JMS提供者之间的虚拟连接，如果我们有一个ConnectionFactory对象，可以使用它来创建一个连接。
```java
Connection connection = connectionFactory.createConnection();
```

创建完连接后，需要在程序使用结束后关闭它：
```java
connection.close();
```

## JMS 会话

Session 是我们对消息进行操作的接口，可以通过session创建生产者、消费者、消息等。Session 提供了事务的功能，如果需要使用session发送/接收多个消息时，可以将这些发送/接收动作放到一个事务中。

我们可以在连接创建完成之后创建session：
```java
Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
```

这里面提供了参数两个参数，第一个参数是是否支持事务，第二个是事务的类型

## JMS消息生产者

消息生产者由Session创建，用于往目的地发送消息。生产者实现MessageProducer接口，我们可以为目的地、队列或话题创建生产者；
```java
MessageProducer producer = session.createProducer(dest);
MessageProducer producer = session.createProducer(queue);
MessageProducer producer = session.createProducer(topic);
```

创建完消息生产者后，可以使用send方法发送消息：
```java
producer.send(message);
```

## JMS消息消费者

消息消费者由Session创建，用于接收被发送到Destination的消息。

```java
MessageConsumer consumer = session.createConsumer(dest);
MessageConsumer consumer = session.createConsumer(queue);
MessageConsumer consumer = session.createConsumer(topic);
```

## JMS消息监听器

消息监听器。如果注册了消息监听器，一旦消息到达，将自动调用监听器的onMessage方法。EJB中的MDB（Message-Driven Bean）就是一种MessageListener。
JMS消息监听器是消息的默认事件处理者，他实现了MessageListener接口，该接口包含一个onMessage方法，在该方法中需要定义消息达到后的具体动作。通过调用setMessageListener方法我们给指定消费者定义了消息监听器。
```java
Listener myListener =newListener();
consumer.setMessageListener(myListener);
```

# JMS消息结构

JMS客户端使用JMS消息与系统通讯，JMS消息虽然格式简单但是非常灵活， JMS消息由三部分组成：

## 消息头

JMS消息头预定义了若干字段用于客户端与JMS提供者之间识别和发送消息，预编译头如下：
* JMSDestination
* JMSDeliveryMode
* JMSMessageID
* JMSTimestamp
* JMSCorrelationID
* JMSReplyTo
* JMSRedelivered
* JMSType
* JMSExpiration
* JMSPriority

## 消息属性

我们可以给消息设置自定义属性，这些属性主要是提供给应用程序的。对于实现消息过滤功能，消息属性非常有用，JMS API定义了一些标准属性，JMS服务提供者可以选择性的提供部分标准属性。

## 消息体

在消息体中，JMS API定义了五种类型的消息格式，让我们可以以不同的形式发送和接受消息，并提供了对已有消息格式的兼容。不同的消息类型如下：
* Text message: javax.jms.TextMessage，表示一个文本对象。
* Object message: javax.jms.ObjectMessage，表示一个JAVA对象。
* Bytes message: javax.jms.BytesMessage，表示字节数据。
* Stream message:javax.jms.StreamMessage，表示java原始值数据流。
* Map message: javax.jms.MapMessage，表示键值对。
