Tomcat调优
=============

# 外部调优
JAVA虚拟机（JVM）性能优化，可以通过以下两个参数来设置虚拟机使用内存的大小，-Xms<size>（JVM初始化堆的大小）和-Xmx<size>（JVM堆的最大值）。
这两个值的大小一般根据需要进行设置。初始化堆的大小执行了虚拟机在启动时向系统申请的内存的大小。一般而言，这个参数不重要。但是有的应用程序在大负载的情况下会急剧地占用更多的内存，此时这个参数就是显得非常重要，如果虚拟机启动时设置使用的内存比较小而在这种情况下有许多对象进行初始化，虚拟机就必须重复地增加内存来满足使用。由于这种原因，我们一般把-Xms和-Xmx设为一样大，而堆的最大值受限于系统使用的物理内存。一般使用数据量较大的应用程序会使用持久对象，内存使用有可能迅速地增长。当应用程序需要的内存超出堆的最大值时虚拟机就会提示内存溢出，并且导致应用服务崩溃。因此一般建议堆的最大值设置为可用内存的最大值的80%。

Tomcat默认可以使用的内存为128MB，在较大型的应用项目中，这点内存是不够的，需要调大。

Windows下，在文件{tomcat_home}/bin/catalina.bat，Unix下，在文件{tomcat_home}/bin/catalina.sh的前面，增加如下设置：
JAVA_OPTS='-Xms【初始化内存大小】-Xmx【可以使用的最大内存】'
需要把这个两个参数值调大。例如：JAVA_OPTS='-Xms256m -Xmx512m'，表示初始化内存为256MB，可以使用的最大内存为512MB。

另外需要考虑的是Java提供的垃圾回收机制。虚拟机的堆大小决定了虚拟机花费在收集垃圾上的时间和频度。收集垃圾可以接受的速度与应用有关，应该通过分析实际的垃圾收集的时间和频率来调整。如果堆的大小很大，那么完全垃圾收集就会很慢，但是频度会降低。如果你把堆的大小和内存的需要一致，完全收集就很快，但是会更加频繁。调整堆大小的的目的是最小化垃圾收集的时间，以在特定的时间内最大化处理客户的请求。在基准测试的时候，为保证最好的性能，要把堆的大小设大，保证垃圾收集不在整个基准测试的过程中出现，   如果系统花费很多的时间收集垃圾，请减小堆大小。一次完全的垃圾收集应该不超过3-5秒。如果垃圾收集成为瓶颈，那么需要指定代的大小，检查垃圾收集的详细输出，研究垃圾收集参数对性能的影响。一般说来，你应该使用物理内存的80%作为堆大小。当增加处理器时，记得增加内存，因为分配可以并行进行，而垃圾收集不是并行的。

# 内部调优
# NIO
而NIO则是使用单线程(单个CPU)或者只使用少量的多线程(多CPU)来接受Socket，而由线程池来处理堵塞在pipe或者队列里的请求.这样的话，只要OS可以接受TCP的连接，web服务器就可以处理该请求。大大提高了web服务器的可伸缩性。
``` xml
<Connector port="8080"
    protocol="org.apache.coyote.http11.Http11NioProtocol"
    connectionTimeout="20000"
    redirectPort="8443" />
```
启之后，进行测试，被打开nio配置，启动时的信息，如下：

```console
2014-2-1 13:01:01 org.apache.tomcat.util.net.NioSelectorPool getSharedSelector
信息: Using a shared selector for servlet write/read
2014-2-1 13:01:01 org.apache.coyote.http11.Http11NioProtocol init
信息: Initializing Coyote HTTP/1.1 on http-8080
```

## 禁止DNS查找
有时候我们的应用可能要记录客户端的信息，两种方式，一是记录客户端的数字IP地址，另一个是在DNS数据中查找真实的主机名。DNS查找会增加网络通信，以致造成了网络延迟。要消除这个延迟，我们可以禁掉DNS查找。这时我们的应用再调用getRemoteHost( )方法时，它就只会得到数字IP地址。这个配置是在Tomcat的serve.xml文件中，Connector对象的enableLookups属性

``` xml
<Connector
    className="org.apache.coyote.tomcat4.CoyoteConnector"
    port="8080"minProcessors="5" maxProcessors="75"
    enableLookups="true" redirectPort="8443"
    acceptCount="10" debug="0" connectionTimeout="20000"
    useURIValidationHack="false"
/>
```
在生产系统中，除非应用要得到所有客户端真实的主机名，这个通常是建议禁掉的。实在不行这样的工作我们可以在Tomcat的外面做。在一个流量较小的Servr这种修改的效果可能不是十分明显，但是对于某些站点来说，也许突然之间流量暴增也说不定呢，比如，前段时间的奥运票务网站，哈哈！

## 线程调整
另一个影响性能的是Connector所使用的进程数。Tomcat使用一个线程池来提高请求响应的速度。Java中的一个线程单独同操作系统交互，并且有它自己的本地内存，以及同进程内的其他线程分享部分共享内存。
我们可以通过修改Connector对象的minProcessors和maxProcessors来控制线程数。这个数字也许在刚上线的时候进行了适当的设置，可是当用户数变多的时候我们就要增加配置数了。minProcessors值应该设置的足够大来处理最小负载。当用户变多的时候，Tomcat会分配更多的线程，不超过maxProcessors。上限也一定要设置的适当，以免使server的内存超过JVM的内存限制而挂掉。

## 加速JSP的编译
第一次访问JSP的时候，它会被转换成Java servlet源码，然后编译成二进制代码。这个过程中，我们是可以控制所使用的编译器的。默认情况，Tomcat所使用的是和命令行上执行javac时同样的编译器。其实有更快的编译器的，我们可以利用这些来提高JSP的编译速度。

# 附录
## server.xml
在tomcat配置文件server.xml中的配置中，和连接数相关的参数有：
* minProcessors：最小空闲连接线程数，用于提高系统处理性能，默认值为10
* maxProcessors：最大连接线程数，即：并发处理的最大请求数，默认值为75
* acceptCount：允许的最大连接数，即等待队列，指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理。应大于等于maxProcessors，默认值为100，
>和最大连接数相关的参数为maxProcessors和acceptCount。如果要加大并发连接数应同时加大这两个参数。web server允许的最大连接数还受制于操作系统的内核参数设置，通常Windows是2000个左右，Linux是1000个左右。
* enableLookups：是否反查域名，取值为：true或false。为了提高处理能力，应设置为false
* connectionTimeout：网络连接超时，单位：毫秒。设置为0表示永不超时，这样设置有隐患的。通常可设置为30000毫秒。
* maxThreads="500" 表示最多同时处理500个连接
* minSpareThreads="100" 表示即使没有人使用也开这么多空线程等待
* maxSpareThreads="300" 表示如果最多可以空75个线程，例如某时刻有80人访问，之后没有人访问了，则tomcat不会保留80个空线程，而是关闭5个空的。
