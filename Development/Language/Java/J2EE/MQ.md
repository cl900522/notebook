ActiveMQ是一个消息中间件，对于消费者而言有两种方式从消息中间件获取消息：

①Push方式：由消息中间件主动地将消息推送给消费者；②Pull方式：由消费者主动向消息中间件拉取消息。看一段官网对Push方式的解释：

To be able to achieve high performance it is important to stream messages to consumers as fast as possible
so that the consumer always has a buffer of messages, in RAM, ready to process
- rather than have them explicitly pull messages from the server which adds significant latency per message.
采用Push方式，可以尽可能快地将消息发送给消费者(stream messages to consumers as fast as possible)

而采用Pull方式，会增加消息的延迟，即消息到达消费者的时间有点长(adds significant latency per message)。



但是，Push方式会有一个坏处：如果消费者的处理消息的能力很弱(一条消息需要很长的时间处理)，而消息中间件不断地向消费者Push消息，消费者的缓冲区可能会溢出。

那ActiveMQ是怎么解决这个问题的呢？那就是  prefetch limit

prefetch limit 规定了一次可以向消费者Push(推送)多少条消息。

 Once the prefetch limit is reached, no more messages are dispatched to the consumer
until the consumer starts sending back acknowledgements of messages (to indicate that the message has been processed)
当推送消息的数量到达了perfetch limit规定的数值时，消费者还没有向消息中间件返回ACK，消息中间件将不再继续向消费者推送消息。



那prefetch limit的值设置为多少合适？视具体的应用场景而定。

 If you have very few messages and each message takes a very long time to process
you might want to set the prefetch value to 1 so that a consumer is given one message at a time.
如果消息的数量很少(生产者生产消息的速率不快)，但是每条消息 消费者需要很长的时间处理，那么prefetch limit设置为1比较合适。这样，消费者每次只会收到一条消息，当它处理完这条消息之后，向消息中间件发送ACK，此时消息中间件再向消费者推送下一条消息。

prefetch limit 设置成0意味着什么？

Specifying a prefetch limit of zero means the consumer will poll for more messages, one at a time,
instead of the message being pushed to the consumer.
意味着此时，消费者去轮询消息中间件获取消息。不再是Push方式了，而是Pull方式了。即消费者主动去消息中间件拉取消息。



perfetch limit是“消息预取”的值，这是针对消息中间件如何向消费者发消息 而设置的。与之相关的还有针对 消费者以何种方式向消息中间件返回确认ACK(响应)：比如消费者是每次消费一条消息之后就向消息中间件确认呢？还是采用“延迟确认”---即采用批量确认的方式(消费了若干条消息之后，统一再发ACK)。这就是 Optimized Acknowledge

ActiveMQ can acknowledge receipt of messages back to the broker in batches (to improve performance).


引用 一段话：“如果prefetchACK为true，那么prefetch必须大于0；当prefetchACK为false时，你可以指定prefetch为0以及任意大小的正数。
不过，当prefetch=0是，表示consumer将使用PULL(拉取)的方式从broker端获取消息，broker端将不会主动push消息给client端，直到client端发送PullCommand时；
当prefetch>0时，就开启了broker push模式，此后只要当client端消费且ACK了一定的消息之后，会立即push给client端多条消息。”



那么，在程序中如何采用Push方式或者Pull方式呢？

从是否阻塞来看，消费者有两种方式获取消息。同步方式和异步方式。

同步方式使用的是ActiveMQMessageConsumer的receive()方法。而异步方式则是采用消费者实现MessageListener接口，监听消息。



使用同步方式receive()方法获取消息时，prefetch limit即可以设置为0，也可以设置为大于0

prefetch limit为零 意味着：“receive()方法将会首先发送一个PULL指令并阻塞，直到broker端返回消息为止，这也意味着消息只能逐个获取(类似于Request<->Response)”

prefetch limit 大于零 意味着：“broker端将会批量push给client 一定数量的消息(<= prefetch)，client端会把这些消息(unconsumedMessage)放入到本地的队列中，只要此队列有消息，那么receive方法将会立即返回，当一定量的消息ACK之后，broker端会继续批量push消息给client端。”



当使用MessageListener异步获取消息时，prefetch limit必须大于零了。因为，prefetch limit 等于零 意味着消息中间件不会主动给消费者Push消息，而此时消费者又用MessageListener被动获取消息(不会主动去轮询消息)。这二者是矛盾的。



此外，还有一个要注意的地方，即消费者采用同步获取消息(receive方法) 与 异步获取消息的方法(MessageListener) ，对消息的确认时机是不同的。



----------------------------------------------------------


作为一个消息系统，消息的传递是一块核心内容，最近在看amq的consumer消费的时候，突然想分析下推拉模式的优劣。好，闲话不多说，来点干货。
消息系统必不可少的就是消息的 push 和 pull了。事实上，push模式和pull模式各有优劣。比如Facebook的Scribe和Cloudera的Flume，AMQ 采用push模式。
Kafka这种新型的消息队列采用2种模式，选择由Producer向broker push消息并由Consumer从broker pull消息。


push模式
在push方式里，是由consumer把轮询过程封装了，并注册MessageListener监听器，取到消息后，唤醒MessageListener的consumeMessage()来消费，对用户而言，感觉消息是被推送过来的。常见的push模式如storm的消息处理，由spout负责消息的推送。该模式下需要一个中心节点，负责消息的分配情况（哪段消息分配给consumer1，哪段消息分配给consumer2），同时还要监听consumer的ack消息用于判断消息是否处理成功，如果在timeout时间内为收到响应可以认为该consumer挂掉，需要重新分配sonsumer上失败的消息。而且容易冲垮consumer，虽然可以向amq那样使用prefetch limit  和预留缓冲区 ，但是还是不能解决由consumer的选择性消费。
使用场景：
push模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。push模式的目标是尽可能以最快速度传递消息，但是这样很容易造成Consumer来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而pull模式则可以根据Consumer的消费能力以适当的速率消费消息。


pull模式
取消息的过程需要用户自己写，根据offset取得消息，kafka中的offset是有zk管理(具体不叙述细节)。由consumer决定消息的消费情况，这种模式有一个好处是我们不需要返回ack消息，因为当consumer申请消费下一批消息时就可以认为上一批消息已经处理完毕，也不需要处理超时的问题，consumer可以根据自己的消费能力来消费消息。但是pull最大问题是实时性问题，对于现在的MQ系统，对latency的追求越来越高，这个时候纯粹的Pull模式将满足不了实时性方面的需求。这时可以采用动态push/pull方式。
consumer在pull的时候，告诉broker自己buffer中可用的容量，整个流程如下：
1、consumer请求broker，告诉broker本地的可承载量，比如500
2、broker在收到消息后，如果没有消息则进入long polling状态
3、当有消息的时候，broker直接向consumer进行push，总共push的数据量为500
4、在整个push期间，consumer无需重新pull，即可获取数据
5、由于broker知道最大容量，所以无需担心被冲垮。
这样也不用担心consumer 被冲垮。


使用场景：
简化Broker的设计，consuemr根据自己的消费能力以适当的速率消费信息；
