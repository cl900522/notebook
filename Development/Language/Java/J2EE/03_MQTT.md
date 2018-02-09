MQTT简介
=======================

>MQTT,mosquitto,Eclipse Paho这三个单词陌生而又神秘。那么这三个单词究竟是什么意思，代表了什么技术，他们之间有关联吗？

# MQTT
MQTT（英语全称，Message Queue Telemetry Transport）,中文翻译过来就是遥测传输协议：其主要提供订阅/发布模式，更为简约、轻量，易于使用，针对受限环境（带宽低、网络延迟高、网络通信不稳定），属于物联网（Internet of Thing）的一个传输协议。设计思想是开放、简单、轻量、易于实现。这些特点使它适用于受限环境。例如，但不仅限于此：

1. 网络代价昂贵，带宽低、不可靠。
2. 在嵌入设备中运行，处理器和内存资源有限。

该协议的特点有：
1. 使用发布/订阅消息模式，提供一对多的消息发布，解除应用程序耦合。
2. 对负载内容屏蔽的消息传输。
3. 使用 TCP/IP 提供网络连接。
4. 小型传输，开销很小（固定长度的头部是 2 字节），协议交换最小化，以降低网络流量。
5. 使用 Last Will 和 Testament 特性通知有关各方客户端异常中断的机制。
6. 有三种消息发布服务质量(QoS)：
    1. “至多一次”(QoS==0)，消息发布完全依赖底层 TCP/IP 网络。会发生消息丢失或重复。这一级别可用于如下情况，环境传感器数据，丢失一次读记录无所谓，因为不久后还会有第二次发送。
    2. “至少一次”(QoS==1)，确保消息到达，但消息重复可能会发生。
    3.  “只有一次”(QoS==\)，确保消息到达一次。这一级别可用于如下情况，在计费系统中，消息重复或丢失会导致不正确的结果。小型传输，开销很小（固定长度的头部是 2 字节），协议交换最小化，以降低网络流量。

因为MQTT是轻量级的发布/订阅的消息传输协议，因此很多应用都可以借用MQTT的思想， 传说中的Facebook的Messager据说就是按照MQTT的协议编写的。具体协议内容，请参考：http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html。

# mosquitto
mosquitto 是MQTT协议标准的一种开源实现，其具体的操作方式，请参考 http://mosquitto.org/man/mosquitto-8.html，其具体的安装和使用方式，网上有很多的资料，咱们就不在重复说明了。对于这种协议，其实有很多的服务器的实现，如下，都是MQTT协议的服务器端的实现。

* IBM Websphere MQ Telemetry
* IBM Integration Bus
* IBM MessageSight
* Mosquitto
* Eclipse Paho
* Eurotech Everywhere Device Cloud
* emqttd
* m2m.io
* Xively
* webMethods Nirvana Messaging
* Apache ActiveMQ
* Apache Apollo
* RabbitMQ
* HiveMQ
* Moquette
* Mosca
* Litmus Automation Loop
* JoramMQ
* ThingMQ
* VerneMQ
更多实现请参考 http://mqtt.org/tag/rabbitmq

# Paho
Eclipse Paho是Eclipse 提供的一个访问MQTT服务器的一种开源客户端库。其提供了7种不同语言平台的客户端类库。
和MQTT服务器进行交互的开源框架还有很多，比如，对于Java语言和平台来说，有下面的框架。

* Eclipse Paho Java

* Xenqtt Includes a client library, mock broker for unit/integration testing, and applications to support enterprise needs like using a cluster of servers as a single client, an HTTP gateway, etc.

* MeQanTT

* Fusesource mqtt-client

* moquette

* "MA9B" zip of 1/2 dozen mobile clients source code. Includes Android-optimized Java source that works with Android notifications, based on Paho

* IA92 - deprecated IBM IA92 support pack, use Eclipse Paho GUI client instead. A useful MQTT Java swing GUI for publishing & subscribing. The Eclipse Paho GUI is identical but uses newer client code

但是，根据目前的流行和使用的次数，应该首推Eclipse Paho
