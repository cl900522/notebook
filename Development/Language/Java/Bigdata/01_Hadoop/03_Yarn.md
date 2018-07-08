Hadoop Yarn
===================

# Yarn原理及运作机制

## YARN基本架构
### 简介
YARN是Hadoop 2.0中的资源管理系统，它的基本设计思想是将MRv1中的JobTracker拆分成了两个独立的服务：一个全局的资源管理器ResourceManager和每个应用程序特有的ApplicationMaster。其中ResourceManager负责整个系统的资源管理和分配，而ApplicationMaster负责单个应用程序的管理。

Yarn是Hadoop集群的资源管理系统。Hadoop2.0对MapReduce框架做了彻底的设计重构，我们称Hadoop2.0中的MapReduce为MRv2或者Yarn。在介绍Yarn之前，我们先回头看一下Hadoop1.x对MapReduce job的调度管理方式（可参考：Hadoop核心之MapReduce架构设计），它主要包括两部分功能：

1. ResourceManagement 资源管理
2. JobScheduling/JobMonitoring 任务调度监控

到了Hadoop2.x也就是Yarn，它的目标是将这两部分功能分开，也就是分别用两个进程来管理这两个任务：

1. ResourceManger
2. ApplicationMaster

需要注意的是，在Yarn中我们把job的概念换成了application，因为在新的Hadoop2.x中，运行的应用不只是MapReduce了，还有可能是其它应用如一个DAG（有向无环图Directed Acyclic Graph，例如storm应用）。Yarn的另一个目标就是拓展Hadoop，使得它不仅仅可以支持MapReduce计算，还能很方便的管理诸如Hive、Hbase、Pig、Spark/Shark等应用。这种新的架构设计能够使得各种类型的应用运行在Hadoop上面，并通过Yarn从系统层面进行统一的管理，也就是说，有了Yarn，各种应用就可以互不干扰的运行在同一个Hadoop系统中，共享整个集群资源，如下图所示：
![](/images/2018/06/yanr-sum.jpg)

### Yarn 框架相对于老的 MapReduce 框架什么优势呢？
- 这个设计大大减小了 ResourceManager 的资源消耗，并且让监测每一个 Job 子任务 (tasks) 状态的程序分布式化了，更安全、更优美。
- 在新的 Yarn 中，ApplicationMaster 是一个可变更的部分，用户可以对不同的编程模型写自己的 AppMst，让更多类型的编程模型能够跑在 Hadoop 集群中，可以参考 hadoop Yarn 官方配置模板中的 ``mapred-site.xml`` 配置。
- 对于资源的表示以内存为单位 ( 在目前版本的 Yarn 中，没有考虑 cpu 的占用 )，比之前以剩余 slot 数目更合理。
- 老的框架中，JobTracker 一个很大的负担就是监控 job 下的 tasks 的运行状况，现在，这个部分就扔给 ApplicationMaster 做了，而 ResourceManager 中有一个模块叫做 ApplicationsManager，它是监测 ApplicationMaster 的运行状况，如果出问题，会将其在其他机器上重启。
- Container 是 Yarn 为了将来作资源隔离而提出的一个框架。这一点应该借鉴了 Mesos 的工作，目前是一个框架，仅仅提供 java 虚拟机内存的隔离 ,hadoop 团队的设计思路应该后续能支持更多的资源调度和控制 , 既然资源表示成内存量，那就没有了之前的 map slot/reduce slot 分开造成集群资源闲置的尴尬情况。

## YARN组成结构
YARN 总体上仍然是Master/Slave结构，在整个资源管理框架中，ResourceManager为Master，NodeManager为 Slave，ResourceManager负责对各个NodeManager上的资源进行统一管理和调度。当用户提交一个应用程序时，需要提供一个用以 跟踪和管理这个程序的ApplicationMaster，它负责向ResourceManager申请资源，并要求NodeManger启动可以占用一 定资源的任务。由于不同的ApplicationMaster被分布到不同的节点上，因此它们之间不会相互影响。在本小节中，我们将对YARN的基本组成 结构进行绍。

Yarn主要由以下几个组件组成：
1. ResourceManager：Global（全局）的进程
2. NodeManager：运行在每个节点上的进程
3. ApplicationMaster：Application-specific（应用级别）的进程
 3.1 Scheduler：是ResourceManager的一个组件*
 3.2 Container：节点上一组CPU和内存资源*

Container是Yarn对计算机计算资源的抽象，它其实就是一组CPU和内存资源，所有的应用都会运行在Container中。ApplicationMaster是对运行在Yarn中某个应用的抽象，它其实就是某个类型应用的实例，ApplicationMaster是应用级别的，它的主要功能就是向ResourceManager（全局的）申请计算资源（Containers）并且和NodeManager交互来执行和监控具体的task。Scheduler是ResourceManager专门进行资源管理的一个组件，负责分配NodeManager上的Container资源，NodeManager也会不断发送自己Container使用情况给ResourceManager。

ResourceManager和NodeManager两个进程主要负责系统管理方面的任务。ResourceManager有一个Scheduler，负责各个集群中应用的资源分配。对于每种类型的每个应用，都会对应一个ApplicationMaster实例，ApplicationMaster通过和ResourceManager沟通获得Container资源来运行具体的job，并跟踪这个job的运行状态、监控运行进度。

图描述了YARN的基本组成结构，YARN主要由ResourceManager、NodeManager、ApplicationMaster（图中给出了MapReduce和MPI两种计算框架的ApplicationMaster，分别为MR AppMstr和MPI AppMstr）和Container等几个组件构成。
![](/images/2018/06/yarn-struct.png)

### ResourceManager(RM)
RM是一个全局的资源管理器，负责整个系统的资源管理和分配。它主要由两个组件构成：调度器（Scheduler）和应用程序管理器（Applications Manager，ASM）。

1. 调度器
    调度器根据容量、队列等限制条件（如每个队列分配一定的资源，最多执行一定数量的作业等），将系统中的资源分配给各个正在运行的应用程序。需要注意的是，该调度器是一个“纯调度器”，它不再从事任何与具体应用程序相关的工作，比如不负责监控或者跟踪应用的执行状态等，也不负责重新启动因应用执 行失败或者硬件故障而产生的失败任务，这些均交由应用程序相关的ApplicationMaster完成。调度器仅根据各个应用程序的资源需求进行资源分 配，而资源分配单位用一个抽象概念“资源容器”（Resource Container，简称Container）表示，Container是一个动态资源分配单位，它将内存、 CPU、磁盘、网络等资源封装在一起，从而限定每个任务使用的资源量。此外，该调度器是一个可插拔的组件，用户可根据自己的需要设计新的调度器，YARN 提供了多种直接可用的调度器，比如Fair Scheduler和Capacity Scheduler等。

    Scheduler是一个资源调度器，它主要负责协调集群中各个应用的资源分配，保障整个集群的运行效率。Scheduler的角色是一个纯调度器，它只负责调度Containers，不会关心应用程序监控及其运行状态等信息。同样，它也不能重启因应用失败或者硬件错误而运行失败的任务

    Scheduler是一个可插拔的插件，它可以调度集群中的各种队列、应用等。在Hadoop的MapReduce框架中主要有两种Scheduler：Capacity Scheduler和Fair Scheduler，关于这两个调度器后面会详细介绍。

2. 应用程序管理器
    应用程序管理器负责管理整个系统中所有应用程序，包括应用程序提交、与调度器协商资源以启动ApplicationMaster、监控ApplicationMaster运行状态并在失败时重新启动它等。

    ApplicationManager主要负责接收job的提交请求，为应用分配第一个Container来运行ApplicationMaster，还有就是负责监控ApplicationMaster，在遇到失败时重启ApplicationMaster运行的Container。

### ApplicationMaster(AM)
用户提交的每个应用程序均包含1个AM，主要功能包括：

1. 与RM调度器协商以获取资源（用Container表示）；
2. 将得到的任务进一步分配给内部的任务；
3. 与NM通信以启动/停止任务；
4. 监控所有任务运行状态，并在任务运行失败时重新为任务申请资源以重启任务。

当前YARN 自带了两个AM实现，一个是用于演示AM编写方法的实例程序distributedshell，它可以申请一定数目的Container以并行运行一个 Shell命令或者Shell脚本；另一个是运行MapReduce应用程序的AM—MRAppMaster，我们将在第8章对其进行介绍。此外，一些其 他的计算框架对应的AM正在开发中，比如Open MPI、Spark等。

### NodeManager(NM)
NM是每个节点上的资源和任务管理器，一方面，它会定时地向RM汇报本节点上的资源使用情况和各个Container的运行状态；另一方面，它接收并处理来自AM的Container启动/停止等各种请求。

NodeManager进程运行在集群中的节点上，每个节点都会有自己的NodeManager。NodeManager是一个slave服务：它负责接收ResourceManager的资源分配请求，分配具体的Container给应用。同时，它还负责监控并报告Container使用信息给ResourceManager。通过和ResourceManager配合，NodeManager负责整个Hadoop集群中的资源分配工作。ResourceManager是一个全局的进程，而NodeManager只是每个节点上的进程，管理这个节点上的资源分配和监控运行节点的健康状态。下面是NodeManager的具体任务列表：
- 接收ResourceManager的请求，分配Container给应用的某个任务
- 和ResourceManager交换信息以确保整个集群平稳运行。ResourceManager就是通过收集每个NodeManager的报告信息来追踪整个集群健康状态的，而NodeManager负责监控自身的健康状态。
- 管理每个Container的生命周期
- 管理每个节点上的日志
- 执行Yarn上面应用的一些额外的服务，比如MapReduce的shuffle过程

当一个节点启动时，它会向ResourceManager进行注册并告知ResourceManager自己有多少资源可用。在运行期，通过NodeManager和ResourceManager协同工作，这些信息会不断被更新并保障整个集群发挥出最佳状态。

NodeManager只负责管理自身的Container，它并不知道运行在它上面应用的信息。负责管理应用信息的组件是ApplicationMaster，在后面会讲到。

### Container
Container 是YARN中的资源抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等，当AM向RM申请资源时，RM为AM返回的资源便是用 Container表示的。YARN会为每个任务分配一个Container，且该任务只能使用该Container中描述的资源。

需要注意的是，Container不同于MRv1中的slot，它是一个动态资源划分单位，是根据应用程序的需求动态生成的。截至本书完成时，YARN仅支持CPU和内存两种资源，且使用了轻量级资源隔离机制Cgroups进行资源隔离。

Container和集群节点的关系是：一个节点会运行多个Container，但一个Container不会跨节点。

## YARN工作流程
当用户向YARN中提交一个应用程序后，YARN将分两个阶段运行该应用程序：

第一个阶段是启动ApplicationMaster；

第二个阶段是由ApplicationMaster创建应用程序，为它申请资源，并监控它的整个运行过程，直到运行完成。

如图所示，YARN的工作流程分为以下几个步骤：
![](/images/2018/06/yarn-sequence.png)
步骤1：　用户向YARN中提交应用程序，其中包括ApplicationMaster程序、启动ApplicationMaster的命令、用户程序等。

步骤2：　ResourceManager为该应用程序分配第一个Container，并与对应的Node-Manager通信，要求它在这个Container中启动应用程序的ApplicationMaster。

步骤3：　ApplicationMaster首先向ResourceManager注册，这样用户可以直接通过ResourceManager查看应用程序的运行状态，然后它将为各个任务申请资源，并监控它的运行状态，直到运行结束，即重复步骤4~7。

步骤4：　ApplicationMaster采用轮询的方式通过RPC协议向ResourceManager申请和领取资源。

步骤5：　一旦ApplicationMaster申请到资源后，便与对应的NodeManager通信，要求它启动任务。

步骤6：　NodeManager为任务设置好运行环境（包括环境变量、JAR包、二进制程序等）后，将任务启动命令写到一个脚本中，并通过运行该脚本启动任务。

步骤7：　各个任务通过某个RPC协议向ApplicationMaster汇报自己的状态和进度，以让ApplicationMaster随时掌握各个任务的运行状态，从而可以在任务失败时重新启动任务。在应用程序运行过程中，用户可随时通过RPC向ApplicationMaster查询应用程序的当前运行状态。

步骤8：　应用程序运行完成后，ApplicationMaster向ResourceManager注销并关闭自己。

### Yarn request分析
Application在Yarn中的执行过程，整个执行过程可以总结为三步：
1. 应用程序提交
2. 启动应用的ApplicationMaster实例
3. ApplicationMaster实例管理应用程序的执行

这幅图展示了应用程序的整个执行过程：
![](/images/2018/06/yarn-sequence2.png)
1. 客户端程序向ResourceManager提交应用并请求一个ApplicationMaster实例

2. ResourceManager找到可以运行一个Container的NodeManager，并在这个Container中启动ApplicationMaster实例

3. ApplicationMaster向ResourceManager进行注册，注册之后客户端就可以查询ResourceManager获得自己ApplicationMaster的详细信息，以后就可以和自己的ApplicationMaster直接交互了

4. 在平常的操作过程中，ApplicationMaster根据resource-request协议向ResourceManager发送resource-request请求

5. 当Container被成功分配之后，ApplicationMaster通过向NodeManager发送container-launch-specification信息来启动Container， container-launch-specification信息包含了能够让Container和ApplicationMaster交流所需要的资料

6. 应用程序的代码在启动的Container中运行，并把运行的进度、状态等信息通过application-specific协议发送给ApplicationMaster

7. 在应用程序运行期间，提交应用的客户端主动和ApplicationMaster交流获得应用的运行状态、进度更新等信息，交流的协议也是application-specific协议

8. 一但应用程序执行完成并且所有相关工作也已经完成，ApplicationMaster向ResourceManager取消注册然后关闭，用到所有的Container也归还给系统

## 多角度理解YARN
可将YARN看做一个云操 作系统，它负责为应用程序启动ApplicationMaster（相当于主线程），然后再由ApplicationMaster负责数据切分、任务分 配、启动和监控等工作，而由ApplicationMaster启动的各个Task（相当于子线程）仅负责自己的计算任务。当所有任务计算完成 后，ApplicationMaster认为应用程序运行完成，然后退出。
![](/images/2018/06/yarn-yunos.png)
