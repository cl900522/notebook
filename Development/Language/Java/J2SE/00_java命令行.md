常用Java命令行
=========

# Jstack
>jstack是java虚拟机自带的一种堆栈跟踪工具。jstack用于打印出给定的java进程ID或core file或远程调试服务的Java堆栈信息，如果是在64位机器上，需要指定选项"-J-d64"，Windows的jstack使用方式只支持以下的这种方式：
```shell
jstack [-l] pid
```

主要分为两个功能： 
* 针对活着的进程做本地的或远程的线程dump； 
* 针对core文件做线程dump。

jstack用于生成java虚拟机当前时刻的线程快照。线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。 线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。 如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。另外，jstack工具还可以附属到正在运行的java程序中，看到当时运行的java程序的java stack和native stack的信息, 如果现在运行的java程序呈现hung的状态，jstack是非常有用的。

想要通过jstack命令来分析线程的情况的话，首先要知道线程都有哪些状态，下面这些状态是我们使用jstack命令查看线程堆栈信息时可能会看到的线程的几种状态：

* NEW,未启动的。不会出现在Dump中。
* RUNNABLE,在虚拟机内执行的。运行中状态，可能里面还能看到locked字样，表明它获得了某把锁。
* BLOCKED,受阻塞并等待监视器锁。被某个锁(synchronizers)給block住了。
* WAITING,无限期等待另一个线程执行特定操作。等待某个condition或monitor发生，一般停留在park(), wait(), sleep(),join() 等语句里。
* TIMED_WAITING,有时限的等待另一个线程的特定操作。和WAITING的区别是wait() 等语句加上了时间限制 wait(timeout)。
* TERMINATED,已退出的。

## Monitor
在多线程的 JAVA程序中，实现线程之间的同步，就要说说 Monitor。 Monitor是 Java中用以实现线程之间的互斥与协作的主要手段，它可以看成是对象或者 Class的锁。每一个对象都有，也仅有一个 monitor。下 面这个图，描述了线程和 Monitor之间关系，以 及线程的状态转换图：

![线程状态](/images/2018/12/thread.bmp)

* 进入区(Entrt Set):表示线程通过synchronized要求获取对象的锁。如果对象未被锁住,则迚入拥有者;否则则在进入区等待。一旦对象锁被其他线程释放,立即参与竞争。
* 拥有者(The Owner):表示某一线程成功竞争到对象锁。
* 等待区(Wait Set):表示线程通过对象的wait方法,释放对象的锁,并在等待区等待被唤醒。

从图中可以看出，一个 Monitor在某个时刻，只能被一个线程拥有，该线程就是 “Active Thread”，而其它线程都是 “Waiting Thread”，分别在两个队列 “ Entry Set”和 “Wait Set”里面等候。在 “Entry Set”中等待的线程状态是 “Waiting for monitor entry”，而在“Wait Set”中等待的线程状态是 “in Object.wait()”。 先看 “Entry Set”里面的线程。我们称被 synchronized保护起来的代码段为临界区。当一个线程申请进入临界区时，它就进入了 “Entry Set”队列。对应的 code就像：
```java
synchronized(obj) {
.........
}
```

## 调用修饰
表示线程在方法调用时,额外的重要的操作。线程Dump分析的重要信息。修饰上方的方法调用。
* locked <地址> 目标：使用synchronized申请对象锁成功,监视器的拥有者。
* waiting to lock <地址> 目标：使用synchronized申请对象锁未成功,在迚入区等待。
* waiting on <地址> 目标：使用synchronized申请对象锁成功后,释放锁幵在等待区等待。
* parking to wait for <地址> 目标


## 线程动作
线程状态产生的原因
* runnable:状态一般为RUNNABLE。
* in Object.wait():等待区等待,状态为WAITING或TIMED_WAITING。
* waiting for monitor entry:进入区等待,状态为BLOCKED。
* waiting on condition:等待区等待、被park。
* sleeping:休眠的线程,调用了Thread.sleep()。

Wait on condition 该状态出现在线程等待某个条件的发生。具体是什么原因，可以结合 stacktrace来分析。 最常见的情况就是线程处于sleep状态，等待被唤醒。 常见的情况还有等待网络IO：在java引入nio之前，对于每个网络连接，都有一个对应的线程来处理网络的读写操作，即使没有可读写的数据，线程仍然阻塞在读写操作上，这样有可能造成资源浪费，而且给操作系统的线程调度也带来压力。在 NewIO里采用了新的机制，编写的服务器程序的性能和可扩展性都得到提高。 正等待网络读写，这可能是一个网络瓶颈的征兆。因为网络阻塞导致线程无法执行。一种情况是网络非常忙，几 乎消耗了所有的带宽，仍然有大量数据等待网络读 写；另一种情况也可能是网络空闲，但由于路由等问题，导致包无法正常的到达。所以要结合系统的一些性能观察工具来综合分析，比如 netstat统计单位时间的发送包的数目，如果很明显超过了所在网络带宽的限制 ; 观察 cpu的利用率，如果系统态的 CPU时间，相对于用户态的 CPU时间比例较高；如果程序运行在 Solaris 10平台上，可以用 dtrace工具看系统调用的情况，如果观察到 read/write的系统调用的次数或者运行时间遥遥领先；这些都指向由于网络带宽所限导致的网络瓶颈。

[Java线程死锁查看分析方法](https://www.cnblogs.com/flyingeagle/articles/6853167.html)

## 命令格式
jstack [ option ] pid
jstack [ option ] executable core
jstack [ option ] [server-id@]remote-hostname-or-IP

### 基本参数：
* -F当’jstack [-l] pid’没有相应的时候强制打印栈信息,如果直接jstack无响应时，用于强制jstack），一般情况不需要使用
* -l长列表. 打印关于锁的附加信息,例如属于java.util.concurrent的ownable synchronizers列表，会使得JVM停顿得长久得多（可能会差很多倍，比如普通的jstack可能几毫秒和一次GC没区别，加了-l 就是近一秒的时间），**-l 建议不要用。一般情况不需要使用**
* -m打印java和native c/c++框架的所有栈信息.可以打印JVM的堆栈,显示上Native的栈帧，一般应用排查不需要使用
* -h | -help打印帮助信息
* pid 需要被打印配置信息的java进程id,可以用jps查询.


# Jstate
jstat命令可以查看堆内存各部分的使用量，以及加载类的数量。命令的格式如下：
```shell
jstat [-命令选项] [vmid] [间隔时间/毫秒] [查询次数]
```

## 类加载统计
jstat -class 2060
- Loaded:加载class的数量
- Bytes：所占用空间大小
- Unloaded：未加载数量
- Bytes:未加载占用空间
- Time：时间

## 编译统计
jstat -compiler 2060
- Compiled：编译数量。
- Failed：失败数量
- Invalid：不可用数量
- Time：时间
- FailedType：失败类型
- FailedMethod：失败的方法

## 垃圾回收统计
jstat -gc 2060

- S0C：第一个幸存区的大小
- S1C：第二个幸存区的大小
- S0U：第一个幸存区的使用大小
- S1U：第二个幸存区的使用大小
- EC：伊甸园区的大小
- EU：伊甸园区的使用大小
- OC：老年代大小
- OU：老年代使用大小
- MC：方法区大小
- MU：方法区使用大小
- CCSC:压缩类空间大小
- CCSU:压缩类空间使用大小
- YGC：年轻代垃圾回收次数
- YGCT：年轻代垃圾回收消耗时间
- FGC：老年代垃圾回收次数
- FGCT：老年代垃圾回收消耗时间
- GCT：垃圾回收消耗总时间

## 堆内存统计
jstat -gccapacity 2060

- NGCMN：新生代最小容量
- NGCMX：新生代最大容量
- NGC：当前新生代容量
- S0C：第一个幸存区大小
- S1C：第二个幸存区的大小
- EC：伊甸园区的大小
- OGCMN：老年代最小容量
- OGCMX：老年代最大容量
- OGC：当前老年代大小
- OC:当前老年代大小
- MCMN:最小元数据容量
- MCMX：最大元数据容量
- MC：当前元数据空间大小
- CCSMN：最小压缩类空间大小
- CCSMX：最大压缩类空间大小
- CCSC：当前压缩类空间大小
- YGC：年轻代gc次数
- FGC：老年代GC次数

## 新生代垃圾回收统计
jstat -gcnew 7172
- S0C：第一个幸存区大小
- S1C：第二个幸存区的大小
- S0U：第一个幸存区的使用大小
- S1U：第二个幸存区的使用大小
- TT:对象在新生代存活的次数
- MTT:对象在新生代存活的最大次数
- DSS:期望的幸存区大小
- EC：伊甸园区的大小
- EU：伊甸园区的使用大小
- YGC：年轻代垃圾回收次数
- YGCT：年轻代垃圾回收消耗时间

## 新生代内存统计
jstat -gcnewcapacity 7172

- NGCMN：新生代最小容量
- NGCMX：新生代最大容量
- NGC：当前新生代容量
- S0CMX：最大幸存1区大小
- S0C：当前幸存1区大小
- S1CMX：最大幸存2区大小
- S1C：当前幸存2区大小
- ECMX：最大伊甸园区大小
- EC：当前伊甸园区大小
- YGC：年轻代垃圾回收次数
- FGC：老年代回收次数

## 老年代垃圾回收统计
jstat -gcold 7172

- MC：方法区大小
- MU：方法区使用大小
- CCSC:压缩类空间大小
- CCSU:压缩类空间使用大小
- OC：老年代大小
- OU：老年代使用大小
- YGC：年轻代垃圾回收次数
- FGC：老年代垃圾回收次数
- FGCT：老年代垃圾回收消耗时间
- GCT：垃圾回收消耗总时间

## 老年代内存统计
jstat -gcoldcapacity 7172

- OGCMN：老年代最小容量
- OGCMX：老年代最大容量
- OGC：当前老年代大小
- OC：老年代大小
- YGC：年轻代垃圾回收次数
- FGC：老年代垃圾回收次数
- FGCT：老年代垃圾回收消耗时间
- GCT：垃圾回收消耗总时间

## 元数据空间统计
jstat -gcmetacapacity 7172
- MCMN:最小元数据容量
- MCMX：最大元数据容量
- MC：当前元数据空间大小
- CCSMN：最小压缩类空间大小
- CCSMX：最大压缩类空间大小
- CCSC：当前压缩类空间大小
- YGC：年轻代垃圾回收次数
- FGC：老年代垃圾回收次数
- FGCT：老年代垃圾回收消耗时间
- GCT：垃圾回收消耗总时间

## 总结垃圾回收统计
jstat -gcutil 7172

- S0：幸存1区当前使用比例
- S1：幸存2区当前使用比例
- E：伊甸园区使用比例
- O：老年代使用比例
- M：元数据区使用比例
- CCS：压缩使用比例
- YGC：年轻代垃圾回收次数
- FGC：老年代垃圾回收次数
- FGCT：老年代垃圾回收消耗时间
- GCT：垃圾回收消耗总时间

## JVM编译方法统计
jstat -printcompilation 7172

- Compiled：最近编译方法的数量
- Size：最近编译方法的字节码数量
- Type：最近编译方法的编译类型。
- Method：方法名标识。

# Jmap
JVM Memory Map命令用于生成heap dump文件，如果不使用这个命令，还可以使用-XX:+HeapDumpOnOutOfMemoryError参数来让虚拟机出现OOM的时候自动生成dump文件。 jmap不仅能生成dump文件，还可以查询finalize执行队列、Java堆和永久代的详细信息，如当前使用率、当前使用的是哪种收集器等。【内存分析】
## 命令格式
```shell
jmap [ option ] pid
jmap [ option ] executable core
jmap [ option ] [server-id@]remote-hostname-or-IP
```

## 参数
- option：选项参数，不可同时使用多个选项参数
- pid：java进程id，命令ps -ef | grep java获取
- executable：产生核心dump的java可执行文件
- core：需要打印配置信息的核心文件
- remote-hostname-or-ip：远程调试的主机名或ip
- server-id：可选的唯一id，如果相同的远程主机上运行了多台调试服务器，用此选项参数标识服务器


## options参数
- heap : 显示Java堆详细信息
- histo : 显示堆中对象的统计信息
- clstatss :类加载器的统计信息
- finalizerinfo : 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
- dump : 生成堆转储快照, 例如：jmap -dump:live,format=b,file=dump.hprof 24971
- F : 当-dump没有响应时，强制生成dump快照

# Java


# Javac
javac命令用与编译java源码文件，其语法格式如下：
javac [ options ] [ sourcefiles ] [ @files ]

参数可按任意次序排列：
* options                       命令行选项。
* sourcefiles                 一个或多个要编译的源文件（例如 MyClass.java）。
* @files                           一个或多个对源文件进行列表的文件。

有两种方法可将源代码文件名传递给 javac：
## sourcefiles
直接给出要编译的源文件。如果源文件数量少，可以用这种方式，在命令行上列出文件名即可。文件与文件之间用空格非分开就可以了。
```shell
javac -d classes src\com\robin\Hello.java src\com\robin\People.java src\com\hubin\Util.java
```

其实这里的源码文件每个都是单独的参数，如果文件路径包括有空格，可以用双引号把该文件名括起来。
比如上面的命令可以写成下面的样子：
```shell
javac -d classes "src\com\robin\Hello.java" "src\com\robin\People.java" "src\com\hubin\Util.java"
```

在没使用分号的情况下，对相同路径下的Java源码文件可以使用统配符，比如示例1可以写成：
```shell
javac -d classes src\com\robin\*.java src\com\hubin\Util.java
```

## @files
为缩短或简化javac命令，可以把要编译的java源文件名列在一个文件，文件名之间用空格或回车进行分割。然后在javac命令行中，可以用'@' 字符加上包含有要编译java源文件名的文件名来指定要编译的java源文件。因为javac当遇到以 `@' 字符，它就会对该字符后的文件所列出的所有java源文件进行编译。这种形式适用于java源文件很多的情况。比如，我们把示例1要编译的源文件名包含在src.txt文件中。

src.txt文件
```
src\com\robin\Hello.java src\com\robin\People.java
src\com\hubin\Util.java
```

然后运行如下的javac命令：
```shell
javac -d classes @src.txt
```

当然我们可以在src.txt中用双引号把单个要编译的java源码文件括起来，但是这时路径直接的分隔符“\”就要写成"\\"的形式了。

src.txt文件
```
"src\\com\\robin\\Hello.java" "src\\com\\robin\\People.java"
"src\\com\\hubin\\Util.java"
```
这时javac命令仍然同上。

## 命令选项options

### -d
该选项用于指定生成的class目标文件的目录。如果某个类是一个包的组成部分，则 javac 将把该类文件放入反映包名的子目录中，必要时创建目录。比如，示例1中的class目标目录就放在classes下，Hello.class和People.class位于classes\com\robin下，Util.class位于classes\com\hubin目录下。

>若未指定 -d 选项，则 javac 将把类文件放到与源文件相同的目录中。
>注意： -d 选项指定的目录不会被自动添加到用户类路径中。

### -bootclasspath，-extdirs，-classpath和-cp

JDK在编译一个java源文件时，搜索类文件的方式和顺序如下：
* Bootstrap classes，
    Bootstrap默认的是JDK自带的jar或zip文件，它包括jre\lib下rt.jar等文件，JDK首先搜索这些文件.可以通过-bootclasspath来设置它。文件之间用分号";"进行分割。
* Extension classes，
    Extension默认的是位于jre"lib"ext目录下的jar文件，JDK在搜索完Bootstrap后就搜索该目录下的jar文件.可以通过-extdirs来设置。文件之间用分号";"来进行分割
* User classes
    User classes搜索顺序为当前目录、环境变量 CLASSPATH、-classpath。

### -cp
-cp 和 -classpath 是同义词，参数意义是一样的。classpath参数太长了，所以提供cp作为缩写形式它们用于告知JDK搜索目录名、jar文档名、zip文档名，用分号";"进行分隔。

例如当你自己开发了公共类并包装成一个common.jar包，在使用 common.jar中的类时，就需要用-classpath common.jar 告诉JDK从common.jar中查找该类，否则JDK就会抛出java.lang.NoClassDefFoundError异常，表明未找到类定义。

使用-classpath后JDK将不再使用CLASSPATH中的类搜索路径，如果-classpath和CLASSPATH都没有设置，则JDK使用当前路径(.)作为类搜索路径。

推荐使用-classpath来定义JDK要搜索的类路径，而不要使用环境变量 CLASSPATH的搜索路径，以减少多个项目同时使用CLASSPATH时存在的潜在冲突。例如应用1要使用a1.0.jar中的类G，应用2要使用 a2.0.jar中的类G,a2.0.jar是a1.0.jar的升级包，当a1.0.jar，a2.0.jar都在CLASSPATH中，JDK搜索到第一个包中的类G时就停止搜索，如果应用1应用2的虚拟机都从CLASSPATH中搜索，就会有一个应用得不到正确版本的类G。
```shell
javac -classpath lib\Util.zip -d classes src\com\robin\*.java
或
javac -cp lib\Util.zip -d classes src\com\robin\*.java
```

```shell
javac -classpath classes -d classes src\com\robin\*.java
或
javac -cp classes -d classes src\com\robin\*.java
```
如果需要指定各个JAR文件具体的存放路径，相同路径有多个可使用通配符。

### -sourcepath
-sourcepath java源码文件路径
在编译时，JDK需要两方面的路径，一个是查找java源码文件的路径，一个是查找class（类）文件的路径。关于class文件的路径上文已经已经介绍过，可以通过-bootclasspath，-extdirs，-classpath和-cp来设定。java源码文件的路径则可以通过

-sourcepath来设定，默认情况下-sourcepath和-classpath的路径一样。在编译的过程中，若需要相关java类的则首先在sourcefiles或@files列出的java源码文件中查找并编译，如果没找到，就在-sourcepath指定的路径中查找java源码文件，这时无论找没找到都会继续在类路径中进行查找。如果在sourcepath中找到了java源码文件，但是在类路径中没有找到了相关的类，或找的类位于包文件（jar或zip）中,或找的类并不是在包文件中，但源码文件比该类文件新，这时会对源码文件进行编译，而且编译生成的类文件将会和你指定要进行编译的java源码所生成的类文件位于同一根目录。否则，除了即没找到java源码文件也没找到相关类就编译失败外，直接载入相关类就可以了。因此你得至少要指定一个要编译的java源文件。它并不是指定sourecfiles或@files中指定的要编译的java源码文件的根目录。与类路径一样，java源码路径项用分号 (;) 进行分隔，它们可以是class文件的根目录、JAR 归档文件或 ZIP 归档文件。
```shell
javac -sourcepath src -d classes src\com\robin\Hello.java
javac -cp lib\Util.zip -sourcepath src -d classes src\com\robin\*.java
```

### -source和-target
* -source 版本
当你从sun安装了某个版本的JDK，而其实该JDK却包含多个版本的编译器。-source参数就是指定用哪个版本的编译器对java源码进行编译。如果你的java源码不符合该版本编译器的规范的话，当然就不能编译通过。

* -target 版本
该命令用于指定生成的class文件将保证和哪个版本的虚拟机进行兼容。我们可以通过-target 1.2来保证生成的class文件能在1.2虚拟机上进行运行，但是1.1的虚拟机就不能保证了。因为java虚拟机的向前兼容行，1.5的虚拟机当然也可以运行通过-target 1.2让生成的class文件。

每个版本编译器的默认-target版本是不太一样的，比如：
JDK1.2版本编译器支持-target 1.1,-target 1.2,-target 1.3,-target 1.4,-target 1.5,-target 1.6,它默认的就是1.1
JDK1.4版本编译器只支持-target 1.4 和-target 1.5
JDK1.5版本编译器就只支持-target 1.5

```shell
javac -cp lib\Util.zip -sourcepath src -source 1.2 -target 1.1 -d classes src\com\robin\*.java
javac -cp lib\Util.zip -sourcepath src -source 1.2 -target 1.3 -d classes src\com\robin\*.java
javac -cp lib\Util.zip -sourcepath src -source 1.2 -target 1.4 -d classes src\com\robin\*.java
javac -cp lib\Util.zip -sourcepath src -source 1.2 -target 1.5 -d classes src\com\robin\*.java
javac -cp lib\Util.zip -sourcepath src -source 1.2 -target 1.6 -d classes src\com\robin\*.java
```

```shell
javac -cp lib\Util.zip -sourcepath src -source 1.4 -target 1.4 -d classes src\com\robin\*.java
javac -cp lib\Util.zip -sourcepath src -source 1.4 -target 1.5 -d classes src\com\robin\*.java
```

### -deprecation
如果java源码中使用了的不鼓励使用的类或类的Field或类的方法或类的方法覆盖，那么如果使用了该参数，将显示关于此的的详细信息,否则只有个简单的Note.
```
D:\project\test>javac -cp lib\Util.zip -sourcepath src -source 1.2 -target 1.6 -
d classes src\com\robin\*.java
Note: src\com\robin\Hello.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.

D:\project\test>javac -cp lib\Util.zip -sourcepath src -deprecation -d classes s
rc\com\robin\*.java
src\com\robin\Hello.java:11: warning: [deprecation] destroy() in java.lang.Threa
d has been deprecated
t.destroy();
               
1 warning
```

### -encoding
设置源文件编码名称，例如UTF-8。若未指定 -encoding 选项，则使用平台缺省的编码方式。

### -g
生成所有的调试信息，包括局部变量。缺省情况下，只生成行号和源文件信息。
    -g:none
    不生成任何调试信息。
    -g:{关键字列表}
    只生成某些类型的调试信息，这些类型由逗号分隔的关键字列表所指定。有效的关键字有：
    source
    源文件调试信息
    lines
    行号调试信息
    vars
    局部变量调试信息

### -nowarn
禁用警告信息。
