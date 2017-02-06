Docker
============

# Docker简介
## Docker是什么
Docker是一个程序运行、测试、交付的开放平台，Docker被设计为能够使你快速地交付应用。在Docker中，你可以将你的程序分为不同的基础部分，对于每一个基础部分都可以当做一个应用程序来管理。Docker能够帮助你快速地测试、快速地编码、快速地交付，并且缩短你从编码到运行应用的周期。

Docker使用轻量级的容器虚拟化平台，并且结合工作流和工具，来帮助你管理、部署你的应用程序。在其核心，Docker实现了让几乎任何程序都可以在一个安全、隔离的容器中运行。安全和隔离可以使你可以同时在机器上运行多个容器。容器轻量级的特性，意味着你可以得到更多的硬件性能。

围绕着容器的虚拟化工具和平台，可以在以下几个方面为你提供帮助：
1. 帮助你把应用程序(包括其余的支持组件)放入到Docker容器中。
2. 分发和转移你的容器至你的团队其它成员来进行进一步的开发和测试。
3. 部署这些应用程序至你的生产环境，不论是本地的数据中心还是云平台。

## 我可以用Docker做什么
### 快速交付你的应用程序
Docker可以为你的开发过程提供完美的帮助。Docker允许开发者在本地包含了应用程序和服务的容器进行开发，之后可以集成到连续的一体化和部署工作流中。
举个例子，开发者们在本地编写代码并且使用Docker和同事分享其开发栈。当开发者们准备好了之后，他们可以将代码和开发栈推送到测试环境中，在该环境进行一切所需要的测试。从测试环境中，你可以将Docker镜像推送到服务器上进行部署。

### 开发和拓展更加简单
Docker的以容器为基础的平台允许高度可移植的工作。Docker容器可以在开发者机器上运行，也可以在实体或者虚拟机上运行，也可以在云平台上运行。
Docker的可移植、轻量特性同样让动态地管理负载更加简单。你可以用Docker快速地增加应用规模或者关闭应用程序和服务。Docker的快速意味着变动几乎是实时的。

### 达到高密度和更多负载
Docker轻巧快速，它提供了一个可行的、 符合成本效益的替代基于虚拟机管理程序的虚拟机。这在高密度的环境下尤其有用。例如，构建你自己的云平台或者PaaS，在中小的部署环境下同样可以获取到更多的资源性能。

## Docker两个主要部件：
1. Docker: 开源的容器虚拟化平台
2. Docker Hub: 用于分享、管理 Docker 容器的 Docker SaaS 平台

## Docker使用客户端-服务器 (C/S) 架构模式
1. Docker 客户端会与 Docker 守护进程进行通信。
2. Docker 守护进程会处理复杂繁重的任务，例如建立、运行、发布你的 Docker 容器。
3. Docker 客户端和守护进程可以运行在同一个系统上，当然你也可以使用 Docker 客户端去连接一个远程的 Docker 守护进程。
4. Docker 客户端和守护进程之间通过 socket 或者 RESTful API 进行通信。
![架构模式](/images/2017/01/p1.png)

### Docker 守护进程
如上图所示，Docker 守护进程运行在一台主机上。用户并不直接和守护进程进行交互，而是通过 Docker 客户端间接和其通信。

### Docker 客户端
Docker 客户端，实际上是 docker 的二进制程序，是主要的用户与 Docker 交互方式。它接收用户指令并且与背后的 Docker 守护进程通信，如此来回往复。

## Docker 内部
要理解 Docker 内部构建，需要理解以下三种部件：
1. Docker 镜像 - Docker images
2. Docker 仓库 - Docker registeries
3. Docker 容器 - Docker containers

### Docker 镜像
Docker 镜像是 Docker 容器运行时的只读模板，每一个镜像由一系列的层 (layers) 组成。Docker 使用 UnionFS 来将这些层联合到单独的镜像中。UnionFS 允许独立文件系统中的文件和文件夹(称之为分支)被透明覆盖，形成一个单独连贯的文件系统。正因为有了这些层的存在，Docker 是如此的轻量。当你改变了一个 Docker 镜像，比如升级到某个程序到新的版本，一个新的层会被创建。因此，不用替换整个原先的镜像或者重新建立(在使用虚拟机的时候你可能会这么做)，只是一个新 的层被添加或升级了。现在你不用重新发布整个镜像，只需要升级，层使得分发 Docker 镜像变得简单和快速。

### Docker 仓库
Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。同样的，Docker 仓库也有公有和私有的概念。公有的 Docker 仓库名字是 Docker Hub。Docker Hub 提供了庞大的镜像集合供使用。这些镜像可以是自己创建，或者在别人的镜像基础上创建。Docker 仓库是 Docker 的分发部分。

### Docker 容器
Docker 容器和文件夹很类似，一个Docker容器包含了所有的某个应用运行所需要的环境。每一个 Docker 容器都是从 Docker 镜像创建的。Docker 容器可以运行、开始、停止、移动和删除。每一个 Docker 容器都是独立和安全的应用平台，Docker 容器是 Docker 的运行部分。

# Docker底层技术
Docker使用Go语言编写，并且使用了一系列Linux内核提供的性能来实现我们已经看到的这些功能。

## Docker需要什么
Docker核心解决的问题是利用LXC来实现类似VM的功能，从而利用更加节省的硬件资源提供给用户更多的计算资源。同VM的方式不同, LXC 其并不是一套硬件虚拟化方法 - 无法归属到全虚拟化、部分虚拟化和半虚拟化中的任意一个，而是一个操作系统级虚拟化方法, 理解起来可能并不像VM那样直观。所以我们从虚拟化要docker要解决的问题出发，看看他是怎么满足用户虚拟化需求的。

用户需要考虑虚拟化方法，尤其是硬件虚拟化方法，需要借助其解决的主要是以下4个问题:

1. **隔离性** - 每个用户实例之间相互隔离, 互不影响。 硬件虚拟化方法给出的方法是VM, LXC给出的方法是container，更细一点是kernel namespace
2. **可配额/可度量** - 每个用户实例可以按需提供其计算资源，所使用的资源可以被计量。硬件虚拟化方法因为虚拟了CPU, memory可以方便实现, LXC则主要是利用cgroups来控制资源
3. **移动性** - 用户的实例可以很方便地复制、移动和重建。硬件虚拟化方法提供snapshot和image来实现，docker(主要)利用AUFS实现
4. **安全性** - 这个话题比较大，这里强调是host主机的角度尽量保护container。硬件虚拟化的方法因为虚拟化的水平比较高，用户进程都是在KVM等虚拟机容器中翻译运行的, 然而对于LXC, 用户的进程是lxc-start进程的子进程, 只是在Kernel的namespace中隔离的, 因此需要一些kernel的patch来保证用户的运行环境不会受到来自host主机的恶意入侵, dotcloud(主要是)利用kernel grsec patch解决的.

## 命名空间(Namespaces)
Docker充分利用了一项称为namespaces的技术来提供隔离的工作空间，我们称之为 container(容器)。当你运行一个容器的时候，Docker为该容器创建了一个命名空间集合。这样提供了一个隔离层，每一个应用在它们自己的命名空间中运行而且不会访问到命名空间之外。

一些Docker使用到的命名空间有：
2. **net**命名空间: 使用在管理网络接口(NET: Networking)。
1. **pid**命名空间: 使用在进程隔离(PID: Process ID)。
3. **ipc**命名空间: 使用在管理进程间通信资源 (IPC: InterProcess Communication)。
4. **mnt**命名空间: 使用在管理挂载点 (MNT: Mount)。
5. **uts**命名空间: 使用在隔离内核和版本标识 (UTS: Unix Timesharing System)。

### pid namespace
不同用户的进程就是通过 pid namespace 隔离开的，且不同 namespace 中可以有相同 PID。具有以下特征:

1. 每个 namespace 中的 pid 是有自己的 pid=1 的进程(类似 /sbin/init 进程)
2. 每个 namespace 中的进程只能影响自己的同一个 namespace 或子 namespace 中的进程
3. 因为 /proc 包含正在运行的进程，因此在 container 中的 pseudo-filesystem 的 /proc 目录只能看到自己 namespace 中的进程
4. 因为 namespace 允许嵌套，父 namespace 可以影响子 namespace 的进程，所以子 namespace 的进程可以在父 namespace 中看到，但是具有不同的 pid

### mnt namespace
类似 chroot，将一个进程放到一个特定的目录执行。mnt namespace 允许不同 namespace 的进程看到的文件结构不同，这样每个 namespace 中的进程所看到的文件目录就被隔离开了。同 chroot 不同，每个 namespace 中的 container 在 /proc/mounts 的信息只包含所在 namespace 的 mount point。

### net namespace
网络隔离是通过 net namespace 实现的， 每个 net namespace 有独立的 network devices, IP addresses, IP routing tables, /proc/net 目录。这样每个 container 的网络就能隔离开来。 docker 默认采用 veth 的方式将 container 中的虚拟网卡同 host 上的一个 docker bridge 连接在一起。

### uts namespace
UTS ("UNIX Time-sharing System") namespace 允许每个 container 拥有独立的 hostname 和 domain name, 使其在网络上可以被视作一个独立的节点而非 Host 上的一个进程。

### ipc namespace
container 中进程交互还是采用 Linux 常见的进程间交互方法 (interprocess communication - IPC), 包括常见的信号量、消息队列和共享内存。然而同 VM 不同，container 的进程间交互实际上还是 host 上具有相同 pid namespace 中的进程间交互，因此需要在IPC资源申请时加入 namespace 信息 - 每个 IPC 资源有一个唯一的 32bit ID。

### user namespace
每个 container 可以有不同的 user 和 group id, 也就是说可以以 container 内部的用户在 container 内部执行程序而非 Host 上的用户。

有了以上 6 种 namespace 从进程、网络、IPC、文件系统、UTS 和用户角度的隔离，一个 container 就可以对外展现出一个独立计算机的能力，并且不同 container 从 OS 层面实现了隔离。 然而不同 namespace 之间资源还是相互竞争的，仍然需要类似 ulimit 来管理每个 container 所能使用的资源 - cgroup


## 资源配额(cgroups)
cgroups 实现了对资源的配额和度量。cgroups 的使用非常简单，提供类似文件的接口，在 /cgroup 目录下新建一个文件夹即可新建一个 group，在此文件夹中新建 task 文件，并将 pid 写入该文件，即可实现对该进程的资源控制。具体的资源配置选项可以在该文件夹中新建子 subsystem ，{子系统前缀}.{资源项} 是典型的配置方法， 如 memory.usageinbytes 就定义了该 group 在 subsystem memory 中的一个内存限制选项。 另外，cgroups 中的 subsystem 可以随意组合，一个 subsystem 可以在不同的 group 中，也可以一个 group 包含多个 subsystem - 也就是说一个 subsystem。

### memory
内存相关的限制

### cpu
在 cgroup 中，并不能像硬件虚拟化方案一样能够定义 CPU 能力，但是能够定义 CPU 轮转的优先级，因此具有较高 CPU 优先级的进程会更可能得到 CPU 运算。 通过将参数写入 cpu.shares ,即可定义改 cgroup 的 CPU 优先级 - 这里是一个相对权重，而非绝对值

### blkio
block IO 相关的统计和限制，byte/operation 统计和限制 (IOPS 等)，读写速度限制等，但是这里主要统计的都是同步 IO

### devices
设备权限限制

## Linux容器(LXC)
借助于namespace的隔离机制和cgroup限额功能，LXC提供了一套统一的API和工具来建立和管理container, LXC利用了如下 kernel 的features:

Kernel namespaces (ipc, uts, mount, pid, network and user)
Apparmor and SELinux profiles
Seccomp policies
Chroots (using pivot_root)
Kernel capabilities
Control groups (cgroups)

LXC 向用户屏蔽了以上 kernel 接口的细节, 提供了如下的组件大大简化了用户的开发和使用工作:

The liblxc library
Several language bindings (python3, lua and Go)
A set of standard tools to control the containers
Container templates

LXC 旨在提供一个共享kernel的 OS 级虚拟化方法，在执行时不用重复加载Kernel, 且container的kernel与host共享，因此可以大大加快container的 启动过程，并显著减少内存消耗。在实际测试中，基于LXC的虚拟化方法的IO和CPU性能几乎接近 baremetal 的性能, 大多数数据有相比 Xen具有优势。当然对于KVM这种也是通过Kernel进行隔离的方式, 性能优势或许不是那么明显, 主要还是内存消耗和启动时间上的差异。在参考文献中提到了利用iozone进行 Disk IO吞吐量测试KVM反而比LXC要快，而且笔者在device mapping driver下重现同样case的实验中也确实能得到如此结论。参考文献从网络虚拟化中虚拟路由的场景(个人理解是网络IO和CPU角度)比较了KVM和LXC, 得到结论是KVM在性能和隔离性的平衡上比LXC更优秀 - KVM在吞吐量上略差于LXC, 但CPU的隔离可管理项比LXC更明确。

关于CPU, DiskIO, network IO 和 memory 在KVM和LXC中的比较还是需要更多的实验才能得出可信服的结论。

## libcontainer
Docker 从 0.9 版本开始使用 libcontainer 替代 lxc，libcontainer 和 Linux 系统的交互图如下：

![libcontainer](/images/2017/01/p2.png)

## AUFS
Docker对container的使用基本是建立在LXC基础之上的，然而LXC存在的问题是难以移动 - 难以通过标准化的模板制作、重建、复制和移动 container。 在以VM为基础的虚拟化手段中，有image和snapshot可以用于VM的复制、重建以及移动的功能。想要通过container来实现快速的大规模部署和更新, 这些功能不可或缺。 Docker正是利用AUFS来实现对container的快速更新 - 在docker0.7中引入了storage driver, 支持AUFS, VFS, device mapper, 也为BTRFS以及ZFS引入提供了可能。 但除了AUFS都未经过dotcloud的线上使用，因此我们还是从AUFS的角度介绍。

AUFS (AnotherUnionFS) 是一种 Union FS, 简单来说就是支持将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)的文件系统, 更进一步地, AUFS支持为每一个成员目录(AKA branch)设定'readonly', 'readwrite' 和 'whiteout-able' 权限, 同时AUFS里有一个类似分层的概念, 对 readonly 权限的branch可以逻辑上进行修改(增量地, 不影响readonly部分的)。通常 Union FS有两个用途, 一方面可以实现不借助 LVM， RAID 将多个disk和挂在到一个目录下, 另一个更常用的就是将一个readonly的branch和一个writeable的branch联合在一起，Live CD正是基于此可以允许在 OS image 不变的基础上允许用户在其上进行一些写操作。Docker在AUFS上构建的container image也正是如此，接下来我们从启动container中的linux为例介绍docker在AUFS特性的运用。

典型的Linux启动到运行需要两个FS - bootfs + rootfs (从功能角度而非文件系统角度)
![autf1](/images/2017/01/p4.png)

bootfs (boot file system) 主要包含 bootloader 和 kernel, bootloader主要是引导加载kernel, 当boot成功后 kernel 被加载到内存中后 bootfs就被umount了. rootfs (root file system) 包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。

由此可见对于不同的linux发行版, bootfs基本是一致的, rootfs会有差别, 因此不同的发行版可以公用bootfs 如下图:
![aufs2](/images/2017/01/p5.png)

典型的Linux在启动后，首先将 rootfs 置为 readonly, 进行一系列检查, 然后将其切换为 "readwrite" 供用户使用。在docker中，起初也是将 rootfs 以readonly方式加载并检查，然而接下来利用 union mount 的将一个 readwrite 文件系统挂载在 readonly 的rootfs之上，并且允许再次将下层的 file system设定为readonly 并且向上叠加, 这样一组readonly和一个writeable的结构构成一个container的运行目录, 每一个被称作一个Layer。如下图:

![aufs3](/images/2017/01/p6.png)

得益于AUFS的特性, 每一个对readonly层文件/目录的修改都只会存在于上层的writeable层中。这样由于不存在竞争, 多个container可以共享readonly的layer。 所以docker将readonly的层称作 "image" - 对于container而言整个rootfs都是read-write的，但事实上所有的修改都写入最上层的writeable层中, image不保存用户状态，可以用于模板、重建和复制。
![aufs4](/images/2017/01/p7.png)
![autf5](/images/2017/01/p7-2.png)

上层的image依赖下层的image，因此docker中把下层的image称作父image，没有父image的image称作base image
![aufs6](/images/2017/01/p8.png)

因此想要从一个image启动一个container，docker会先加载其父image直到base image，用户的进程运行在writeable的layer中。所有parent image中的数据信息以及 ID、网络和lxc管理的资源限制等具体container的配置，构成一个docker概念上的container。如下图:
![autf7](/images/2017/01/p9.png)

由此可见，采用AUFS作为docker的container的文件系统，能够提供如下好处:
1. 节省存储空间 - 多个container可以共享base image存储
2. 快速部署 - 如果要部署多个container，base image可以避免多次拷贝
3. 内存更省 - 因为多个container共享base image, 以及OS的disk缓存机制，多个container中的进程命中缓存内容的几率大大增加
4. 升级更方便 - 相比于 copy-on-write 类型的FS，base-image也是可以挂载为可writeable的，可以通过更新base image而一次性更新其之上的container
5. 允许在不更改base-image的同时修改其目录中的文件 - 所有写操作都发生在最上层的writeable层中，这样可以大大增加base image能共享的文件内容。

以上5条 1-3 条可以通过 copy-on-write 的FS实现, 4可以利用其他的union mount方式实现, 5只有AUFS实现的很好。这也是为什么Docker一开始就建立在AUFS之上。

由于AUFS并不会进入linux主干 (According to Christoph Hellwig, linux rejects all union-type filesystems but UnionMount.), 同时要求kernel版本3.0以上(docker推荐3.8及以上)，因此在RedHat工程师的帮助下在docker0.7版本中实现了driver机制, AUFS只是其中的一个driver, 在RHEL中采用的则是Device Mapper的方式实现的container文件系统。

## GRSEC
grsec是linux kernel安全相关的patch, 用于保护host防止非法入侵。由于其并不是docker的一部分，我们只进行简单的介绍。 grsec可以主要从4个方面保护进程不被非法入侵:

1. 随机地址空间 - 进程的堆区地址是随机的
2. 用只读的memory management unit来管理进程流程, 堆区和栈区内存只包含数据结构/函数/返回地址和数据, 是non-executeable
3. 审计和Log可疑活动
4. 编译期的防护

安全永远是相对的，这些方法只是告诉我们可以从这些角度考虑container类型的安全问题可以关注的方面。

# Docker相对于LXC有什么优点
看似docker主要的OS级虚拟化操作是借助LXC, AUFS只是锦上添花。那么肯定会有人好奇docker到底比LXC多了些什么。无意中发现 stackoverflow 上正好有人问这个问题， 回答者是Dotcloud的创始人，出于备忘目的原文摘录如下。

On top of this low-level foundation of kernel features, Docker offers a high-level tool with several powerful functionalities:

Portable deployment across machines. Docker defines a format for bundling an application and all its dependencies into a single object which can be transferred to any docker-enabled machine, and executed there with the guarantee that the execution environment exposed to the application will be the same. Lxc implements process sandboxing, which is an important pre-requisite for portable deployment, but that alone is not enough for portable deployment. If you sent me a copy of your application installed in a custom lxc configuration, it would almost certainly not run on my machine the way it does on yours, because it is tied to your machine's specific configuration: networking, storage, logging, distro, etc. Docker defines an abstraction for these machine-specific settings, so that the exact same docker container can run - unchanged - on many different machines, with many different configurations.

Application-centric. Docker is optimized for the deployment of applications, as opposed to machines. This is reflected in its API, user interface, design philosophy and documentation. By contrast, the lxc helper scripts focus on containers as lightweight machines - basically servers that boot faster and need less ram. We think there's more to containers than just that.

Automatic build. Docker includes a tool for developers to automatically assemble a container from their source code, with full control over application dependencies, build tools, packaging etc. They are free to use make, maven, chef, puppet, salt, debian packages, rpms, source tarballs, or any combination of the above, regardless of the configuration of the machines.

Versioning. Docker includes git-like capabilities for tracking successive versions of a container, inspecting the diff between versions, committing new versions, rolling back etc. The history also includes how a container was assembled and by whom, so you get full traceability from the production server all the way back to the upstream developer. Docker also implements incremental uploads and downloads, similar to "git pull", so new versions of a container can be transferred by only sending diffs.

Component re-use. Any container can be used as an "base image" to create more specialized components. This can be done manually or as part of an automated build. For example you can prepare the ideal python environment, and use it as a base for 10 different applications. Your ideal postgresql setup can be re-used for all your future projects. And so on.

Sharing. Docker has access to a public registry (http://index.docker.io) where thousands of people have uploaded useful containers: anything from redis, couchdb, postgres to irc bouncers to rails app servers to hadoop to base images for various distros. The registry also includes an official "standard library" of useful containers maintained by the docker team. The registry itself is open-source, so anyone can deploy their own registry to store and transfer private containers, for internal server deployments for example.

Tool ecosystem. Docker defines an API for automating and customizing the creation and deployment of containers. There are a huge number of tools integrating with docker to extend its capabilities. PaaS-like deployment (Dokku, Deis, Flynn), multi-node orchestration (maestro, salt, mesos, openstack nova), management dashboards (docker-ui, openstack horizon, shipyard), configuration management (chef, puppet), continuous integration (jenkins, strider, travis), etc. Docker is rapidly establishing itself as the standard for container-based tooling.
