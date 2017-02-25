Docker Swarm
============

# Docker Swarm 是什么？

Docker Swarm是一个用于创建Docker主机（运行Docker守护进程的服务器）集群的工具，使用Swarm操作集群，会使用户感觉就像是在一台主机上进行操作。在DockerCon欧洲大会宣布Swarm相关技术之前，我就曾经在Docker Global Hack Day上对于Swarm的相关技术进行了介绍。在这次hackday上，大家介绍了一些非常酷的新技术，包括Docker Swarm（译者注：Docker集群工具）、Docker Machine（译者注：Docker管理工具）以及Docker Compose（译者注：Docker编排工具）。由于Ansible (一个自动化的运维工具)与Machine和Compose有类似的功能，所以Swarm服务更能引起我的兴趣。

在大会上，Victor Vieux与Andrea Luzzardi介绍了有关Swarm的基本概念，并且演示了Swarm的基本工作流程，他们还提出了一个非常有趣的结论：虽然POC （概念验证proof of concept）在功能上对项目进行论证并且能够进行基本的展示，但是这是以忽略整个的项目代码并且从头开始构建项目为代价的。我很赞同这个想法，在以后对一个项目进行POC时也要牢记这一点。

Swarm守护进程是使用Go语言编写的，截止到目前仍然处于Alpha阶段。Swarm项目的发展速度很快，几乎每天都会有新的功能和特性更新，据说，@vieux本人也十分支持在项目中添加的功能，并且欢迎通过GitHub的Issue来修复bug。虽然我暂时不推荐在生产环境中使用Swarm，但是我相信它会是一个非常有前景的技术。

# Docker Swarm 如何工作？

对Swarm进行操作的过程与处理单个Docker主机非常相似，无需进行太多修改，它就可以和现有的工具链进行很好的交互。Swarm是运行在Linux机器上的守护程序，它所绑定的网络接口与独立Docker实例相同（http/2375或https/2376）。Swarm 守护进程可以与标准的Docker客户端>=1.4.0相连接并接受其发送来的信息，之后，Swarm服务会对来自Docker客户端的指令信息进行配置，最后通过代理的方式把信息发送给不同Docker的守护进程，Swarm服务同时也监听着标准的Docker端口。 比如，Swarm会基于不同的打包算法并结合Docker守护进程在启动时指定好标签（tags），把create命令分配到不同的Docker守护进程上来执行。根据这一特性，用户可以创建由不同的Docker主机所构成的分区集群（partitioned cluster）并且将整个集群在逻辑上以一个单一的Docker终端的形式公开给用户，Swarm使这个过程变得极其简单。

与Swarm的交互“或多或少”地类似于与一个非集群的Docker实例的交互，但是也有一些需要注意的地方。Swarm并非对所有的Docker命令提供一对一的支持。这不仅是由于两种服务在体系结构上的区别，还因为Swarm服务刚刚起步不久，有些命令尚未得到实现（我想有些可能永远不会被实现）。当前几乎一切需要运行容器的命令都是可用的，包括以下的常用命令（还有其他的）：
docker run
docker create
docker inspect
docker kill
docker logs
docker start

以上介绍的内容是运行该工具时所需的重要组成部分。下面介绍一下在最常见的配置情况下，相关的技术具体如何运行：
Docker主机服务（服务器上的Docker守护进程）通过--label key=value被启动并对网络进行监听。
Swarm守护进程被启动并指向一个文件，这个文件包含有构成集群的主机以及这些主机所监听的端口列表。
Swarm与每个Docker主机交互并确定他们的标记、健康状况以及资源使用量，并维护后端和它们元数据的列表。
客户端通过它的网络端口（2375）与Swarm交互。与Swarm的交互方式与Docker交互的方式类似：创建、销毁、运行、依附（attach）并获得运行容器的日志以及其他相关内容。
当一个命令发出给Swarm，Swarm会：
基于提供的constraint标签、终端的健康程度以及调度算法来决定把命令发送到哪里。
针对合适的Docker守护进程执行命令。
返回结果的格式与Docker守护进程相同。

Swarm_Diagram_Omnigraffle.jpg

Swarm守护进程本身相当于是一个调度器和一个路由器。它实际上并没有运行容器，也就是说，如果Swarm服务停止了，它在终端Docker主机上已分配好的容器仍然是开启的。另外，由于它不处理任何网络路由（网络连接需要被直接发送到后端的Docker主机上），即使Swarm守护进程意外终止，运行的容器仍然可用。当Swarm从这样的崩溃中恢复，它依然能够查询终端以重建其元数据的列表。

由于Swarm的设计理念，在所有运行时的活动与Swarm服务交互的过程中，几乎与其他Docker守护进程的交互过程是相同的，比如：Docker 客户端、docker-py、docker-api gem等。然而，目前Ansible似乎在TLS 模式下不能与Swarm共同使用，但它似乎不仅仅影响Swarm的使用，也影响Docker守护程序本身的使用。

这是有关Docker Swarm系列博文的第一篇。由于缺少相关的技术细节，我表示抱歉，但在随后的文章中会包含一些架构、代码片段以及相关的实践 :) 请留意第二篇：Docker Swarm的配置选项与需求，即将推出。

所有这些研究工作得以顺利进行，都归功于我工作的公司： Rally Software in Boulder, CO.。每季度我们至少有一个骇客周，它使我们能够研究很棒的东西，像Docker Swarm。如果您想切入正题，直接开始Vagrant 例子，这里有一个repo，它是我在2014年Q1骇客周研究的成果：https://github.com/technolo-g/docker-swarm-demo
