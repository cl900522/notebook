Kubernetes核心概念
==========

# 什么是Kubernetes

Kubernetes（k8s）是自动化容器操作的开源平台，这些操作包括部署，调度和节点集群间扩展。如果你曾经用过Docker容器技术部署容器，那么可以将Docker看成Kubernetes内部使用的低级别组件。Kubernetes不仅仅支持Docker，还支持Rocket，这是另一种容器技术。
使用Kubernetes可以：

* 自动化容器的部署和复制
* 随时扩展或收缩容器规模
* 将容器组织成组，并且提供容器间的负载均衡
* 很容易地升级应用程序容器的新版本
* 提供容器弹性，如果容器失效就替换它，等等...

## 使用Kubernetes能做什么？

可以在物理或虚拟机的Kubernetes集群上运行容器化应用，Kubernetes能提供一个以“容器为中心的基础架构”，满足在生产环境中运行应用的一些常见需求，如：

* 多个进程（作为容器运行）协同工作。（Pod）
* 存储系统挂载
* Distributing secrets
* 应用健康检测
* 应用实例的复制
* Pod自动伸缩/扩展
* Naming and discovering
* 负载均衡
* 滚动更新
* 资源监控
* 日志访问
* 调试应用程序
* 提供认证和授权

## Kubernetes不是什么？

Kubernetes并不是传统的PaaS（平台即服务）系统。

* Kubernetes不限制支持应用的类型，不限制应用框架。不限制受支持的语言runtimes (例如, Java, Python, Ruby)，满足12-factor applications 。不区分 “apps” 或者“services”。 Kubernetes支持不同负载应用，包括有状态、无状态、数据处理类型的应用。只要这个应用可以在容器里运行，那么就能很好的运行在Kubernetes上。
* Kubernetes不提供中间件（如message buses）、数据处理框架（如Spark）、数据库(如Mysql)或者集群存储系统(如Ceph)作为内置服务。但这些应用都可以运行在Kubernetes上面。
* Kubernetes不部署源码不编译应用。持续集成的 (CI)工作流方面，不同的用户有不同的需求和偏好的区域，因此，我们提供分层的 CI工作流，但并不定义它应该如何工作。
* Kubernetes允许用户选择自己的日志、监控和报警系统。
* Kubernetes不提供或授权一个全面的应用程序配置 语言/系统（例如，jsonnet）。
* Kubernetes不提供任何机器配置、维护、管理或者自修复系统。

## 集群
集群是一组节点，这些节点可以是物理服务器或者虚拟机，之上安装了Kubernetes平台。下图展示这样的集群。注意该图为了强调核心概念有所简化。这里可以看到一个典型的Kubernetes架构图。

![k8s-cluster](/images/2019/01/k8s-cluster.png)

上图可以看到如下组件，使用特别的图标表示Service和Label：
* Pod
* Container（容器）
* Label(label)（标签）
* Replication Controller（复制控制器）
* Service（enter image description here）（服务）
* Node（节点）
* Kubernetes Master（Kubernetes主节点）

# 术语

## Pod

Pod（上图绿色方框）安排在节点上，包含一组容器和卷。同一个Pod里的容器共享同一个网络命名空间，可以使用localhost互相通信。Pod是短暂的，不是持续性实体。你可能会有这些问题：

* 如果Pod是短暂的，那么我怎么才能持久化容器数据使其能够跨重启而存在呢？ 是的，Kubernetes支持卷的概念，因此可以使用持久化的卷类型。

* 是否手动创建Pod，如果想要创建同一个容器的多份拷贝，需要一个个分别创建出来么？可以手动创建单个Pod，但是也可以使用Replication Controller使用Pod模板创建出多份拷贝，下文会详细介绍。

* 如果Pod是短暂的，那么重启时IP地址可能会改变，那么怎么才能从前端容器正确可靠地指向后台容器呢？这时可以使用Service，下文会详细介绍。

## Lable

正如图所示，一些Pod有Label。一个Label是attach到Pod的一对键/值对，用来传递用户定义的属性。比如，你可能创建了一个"tier"和“app”标签，通过Label（tier=frontend, app=myapp）来标记前端Pod容器，使用Label（tier=backend, app=myapp）标记后台Pod。然后可以使用Selectors选择带有特定Label的Pod，并且将Service或者Replication Controller应用到上面。

## Replication Controller
>是否手动创建Pod，如果想要创建同一个容器的多份拷贝，需要一个个分别创建出来么，能否将Pods划到逻辑组里？

Replication Controller确保任意时间都有指定数量的Pod“副本”在运行。如果为某个Pod创建了Replication Controller并且指定3个副本，它会创建3个Pod，并且持续监控它们。如果某个Pod不响应，那么Replication Controller会替换它，保持总数为3.如下面的动画所示：

![k8s-replication](/images/2019/01/k8s-replication.gif)

如果之前不响应的Pod恢复了，现在就有4个Pod了，那么Replication Controller会将其中一个终止保持总数为3。如果在运行中将副本总数改为5，Replication Controller会立刻启动2个新Pod，保证总数为5。还可以按照这样的方式缩小Pod，这个特性在执行滚动升级时很有用。

当创建Replication Controller时，需要指定两个东西：

1. Pod模板：用来创建Pod副本的模板

2. Label：Replication Controller需要监控的Pod的标签。

现在已经创建了Pod的一些副本，那么在这些副本上如何均衡负载呢？我们需要的是Service。

## Service

如果Pods是短暂的，那么重启时IP地址可能会改变，怎么才能从前端容器正确可靠地指向后台容器呢？

Service是定义一系列Pod以及访问这些Pod的策略的一层抽象。Service通过Label找到Pod组。因为Service是抽象的，所以在图表里通常看不到它们的存在，这也就让这一概念更难以理解。

现在，假定有2个后台Pod，并且定义后台Service的名称为‘backend-service’，lable选择器为（tier=backend, app=myapp）。backend-service 的Service会完成如下两件重要的事情：

1. 会为Service创建一个本地集群的DNS入口，因此前端Pod只需要DNS查找主机名为 ‘backend-service’，就能够解析出前端应用程序可用的IP地址。

2. 现在前端已经得到了后台服务的IP地址，但是它应该访问2个后台Pod的哪一个呢？Service在这2个后台Pod之间提供透明的负载均衡，会将请求分发给其中的任意一个（如下面的动画所示）。通过每个Node上运行的代理（kube-proxy）完成。这里有更多技术细节。

下述动画展示了Service的功能。注意该图作了很多简化。如果不进入网络配置，那么达到透明的负载均衡目标所涉及的底层网络和路由相对先进。如果有兴趣，这里有更深入的介绍。

![k8s-service](/images/2019/01/k8s-service.gif)

有一个特别类型的Kubernetes Service，称为'LoadBalancer'，作为外部负载均衡器使用，在一定数量的Pod之间均衡流量。比如，对于负载均衡Web流量很有用。

## Node

节点（上图橘色方框）是物理或者虚拟机器，作为Kubernetes worker，通常称为Minion。每个节点都运行如下Kubernetes关键组件：

* Kubelet：是主节点代理。

* Kube-proxy：Service使用其将链接路由到Pod，如上文所述。

* Docker或Rocket：Kubernetes使用的容器技术来创建容器。

## Kubernetes Master

集群拥有一个Kubernetes Master（紫色方框）。Kubernetes Master提供集群的独特视角，并且拥有一系列组件，比如Kubernetes API Server。API Server提供可以用来和集群交互的REST端点。master节点包括用来创建和复制Pod的Replication Controller。


# Kubernetes 组件

## Master 组件
Master 组件提供的集群控制。Master 组件对集群做出全局性决策(例如：调度)，以及检测和响应集群事件(副本控制器的replicas字段不满足时,启动新的副本)。Master 组件可以在集群中的任何节点上运行。然而，为了简单起见，设置脚本通常会启动同一个虚拟机上所有 Master 组件，并且不会在此虚拟机上运行用户容器。

### API服务器
kube-apiserver对外暴露了Kubernetes API。它是的 Kubernetes 前端控制层。它被设计为水平扩展，即通过部署更多实例来缩放

### etcd
etcd 用于 Kubernetes 的后端存储。所有集群数据都存储在此处，始终为您的 Kubernetes 集群的 etcd 数据提供备份计划。

### kube-controller-manager
kube-controller-manager运行控制器，它们是处理集群中常规任务的后台线程。逻辑上，每个控制器是一个单独的进程，但为了降低复杂性，它们都被编译成独立的可执行文件，并在单个进程中运行。

这些控制器包括:

* 节点控制器: 当节点移除时，负责注意和响应。
* 副本控制器: 负责维护系统中每个副本控制器对象正确数量的 Pod。
* 端点控制器: 填充 端点(Endpoints) 对象(即连接 Services & Pods)。
* 服务帐户和令牌控制器: 为新的命名空间创建默认帐户和 API 访问令牌

### cloud-controller-manager

cloud-controller-manager 是用于与底层云提供商交互的控制器。云控制器管理器可执行组件是 Kubernetes v1.6 版本中引入的 Alpha 功能。

cloud-controller-manager 仅运行云提供商特定的控制器循环。您必须在 kube-controller-manager 中禁用这些控制器循环，您可以通过在启动 kube-controller-manager 时将 --cloud-provider 标志设置为external来禁用控制器循环。

cloud-controller-manager 允许云供应商代码和 Kubernetes 核心彼此独立发展，在以前的版本中，Kubernetes 核心代码依赖于云提供商特定的功能代码。在未来的版本中，云供应商的特定代码应由云供应商自己维护，并与运行 Kubernetes 的云控制器管理器相关联。

以下控制器具有云提供商依赖关系:

* 节点控制器: 用于检查云提供商以确定节点是否在云中停止响应后被删除
* 路由控制器: 用于在底层云基础架构中设置路由
* 服务控制器: 用于创建，更新和删除云提供商负载平衡器
* 数据卷控制器: 用于创建，附加和装载卷，并与云提供商进行交互以协调卷

### kube-scheduler

kube-scheduler监视没有分配节点的新创建的 Pod，选择一个节点供他们运行.


### 插件(addons)
插件是实现集群功能的 Pod 和 Service。 Pods 可以通过 Deployments，ReplicationControllers 管理。插件对象本身是受命名空间限制的，被创建于 kube-system 命名空间。Addon 管理器用于创建和维护附加资源

### DNS
虽然其他插件并不是必需的，但所有 Kubernetes 集群都应该具有Cluster DNS，许多示例依赖于它。

Cluster DNS 是一个 DNS 服务器，和您部署环境中的其他 DNS 服务器一起工作，为 Kubernetes 服务提供DNS记录。

Kubernetes 启动的容器自动将 DNS 服务器包含在 DNS 搜索中。

### 用户界面
dashboard 提供了集群状态的只读概述。有关更多信息，请参阅使用HTTP代理访问 Kubernetes API

### 容器资源监控
容器资源监控将关于容器的一些常见的时间序列度量值保存到一个集中的数据库中，并提供用于浏览这些数据的界面。

### 集群层面日志
集群层面日志 机制负责将容器的日志数据保存到一个集中的日志存储中，该存储能够提供搜索和浏览接口

## 节点组件

节点组件在每个节点上运行，维护运行的 Pod 并提供 Kubernetes 运行时环境。

### kubelet

kubelet是主要的节点代理,它监测已分配给其节点的 Pod(通过 apiserver 或通过本地配置文件)，提供如下功能:

* 挂载 Pod 所需要的数据卷(Volume)。
* 下载 Pod 的 secrets。
* 通过 Docker 运行(或通过 rkt)运行 Pod 的容器。
* 周期性的对容器生命周期进行探测。
* 如果需要，通过创建 镜像 Pod（Mirror Pod） 将 Pod 的状态报告回系统的其余部分。
* 将节点的状态报告回系统的其余部分。

### kube-proxy

kube-proxy通过维护主机上的网络规则并执行连接转发，实现了Kubernetes服务抽象。

### docker

Docker 用于运行容器。

### rkt

支持 rkt 运行容器作为 Docker 的试验性替代方案。

### supervisord

supervisord 是一个轻量级的进程监控系统，可以用来保证 kubelet 和 docker 运行。

### fluentd

fluentd 是一个守护进程，它有助于提供集群层面日志 集群层面的日志



# 工具

## kubeadmin

## kubectl

## minikube