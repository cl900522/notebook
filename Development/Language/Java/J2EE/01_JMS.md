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
1. Point-to-Point Messaging Domain （点对点）
2. Publish/Subscribe Messaging Domain （发布/订阅模式）

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

## 特点
一个消息可以传递个多个订阅者（即：一个消息可以有多个接受方）
发布者与订阅者具**有时间约束**，针对某个主题（Topic）的订阅者，它必须创建一个订阅者之后，才能消费发布者的消息，而且为了消费消息，订阅者必须保持运行的状态。
为了缓和这样严格的时间相关性，JMS允许订阅者创建一个可持久化的订阅。这样，即使订阅者没有被激活（运行），它也能接收到发布者的消息。

# JMS接收消息
在JMS中，消息的产生和消息是异步的。对于消费来说，JMS的消息者可以通过两种方式来消费消息。

1. 同步（Synchronous）
在同步消费信息模式模式中，订阅者/接收方通过调用 receive（）方法来接收消息。在receive（）方法中，线程会阻塞直到消息到达或者到指定时间后消息仍未到达。

2. 异步（Asynchronous）
使用异步方式接收消息的话，消息订阅者需注册一个消息监听者，类似于事件监听器，只要消息到达，JMS服务提供者会通过调用监听器的onMessage()递送消息。

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
