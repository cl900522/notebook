Docker Swarm
=========
>To use Docker in swarm mode, install Docker 1.12.0 or later.

# 相关术语
## 什么是群体？
Docker Engine中嵌入的集群管理和编排功能是使用SwarmKit构建的。参与群集的Docker引擎以群模式运行。您可以通过初始化群集或加入现有群组来为引擎启用群组模式。
一个群是泊坞引擎或集群节点，在这里部署 服务。Docker Engine CLI和API包括管理群组节点（例如，添加或删除节点）的命令，以及在群集中部署和编排服务。
当您运行Docker而不使用swarm模式时，您将执行容器命令。当您以群组模式运行Docker时，您可以编排服务。您可以在同一Docker实例上运行群集服务和独立容器。

## 什么是节点？
一个节点是泊坞引擎参与群的一个实例。您也可以将其视为Docker节点。您可以在单个物理计算机或云服务器上运行一个或多个节点，但生产群集部署通常包括分布在多个物理机器和云计算机上的Docker节点。
要将应用程序部署到群集，您可以向管理器节点提交服务定义 。经理节点将称为任务的工作单元分派 到工作节点。
Manager节点还执行所需的业务流程和集群管理功能，以保持群集的所需状态。经理节点选举一名领导进行协调任务。
工作节点接收并执行从管理器节点分派的任务。默认情况下，管理器节点也会以服务器节点运行服务，但您可以将其配置为独占运行管理器任务，并且仅管理节点。代理在每个工作节点上运行，并报告分派给它的任务。工作节点向管理者节点通知其分配任务的当前状态，以便管理员可以维护每个工作者的所需状态。

![swarm-diagram](/images/2017/07/swarm-diagram.png)

## 服务和任务
一个服务是任务的定义为工作节点上执行。它是群体系统的中心结构，也是用户与群体交互的主要根源。
创建服务时，您可以指定要使用的容器映像以及运行容器中要执行的命令。
在复制服务模型中，群组管理器根据您在所需状态下设置的比例，在节点之间分配特定数量的副本任务。
对于全局服务，群集在集群中的每个可用节点上为服务运行一个任务。
一个任务带有一个Docker容器和在容器内运行的命令。它是群体的原子调度单元。管理器节点根据服务规模中设置的副本数量将任务分配给工作节点。一旦将任务分配给某个节点，它就不能移动到另一个节点。它只能在分配的节点上运行或失败。
![services-diagram](/images/2017/07/services-diagram.png)

## 负载均衡
群组管理员使用入口负载平衡来暴露您想从外部向群集提供的服务。群组管理员可以自动将服务分配给一个PublishedPort，也可以为该服务配置一个PublishedPort。您可以指定任何未使用的端口。如果不指定端口，则swarm管理器将该端口分配给30000-32767的端口。
外部组件（如云端负载平衡器）可以访问集群中任何节点的PublishedPort上的服务，无论节点当前是否正在运行该服务的任务。群集路由中的所有节点都将连接连接到正在运行的任务实例。
Swarm模式具有一个内部DNS组件，可自动为群集中的每个服务分配DNS条目。群组管理员使用内部负载平衡，根据服务的DNS名称在集群内部的服务之间分配请求。

# 创建群体
1. 运行以下命令创建一个新的群组：
    ```shell
    docker swarm init --advertise-addr <MANAGER-IP>
    ```

2. 输出包括将新节点加入群组的命令。节点将作为管理员或工作人员，视--token 标志的价值而定。

    ```shell
    $ docker swarm init --advertise-addr 192.168.99.100
    Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.

    To add a worker to this swarm, run the following command:

        docker swarm join \
        --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
        192.168.99.100:2377

    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
    ```

3. 执行docker node ls命令查看节点信息：

# 将节点添加到群集
1. 运行docker swarm init由“创建群组”教程步骤的输出生成的命令，以创建加入现有群组的工作节点：
    ```shell
    $ docker swarm join \
      --token  SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
      192.168.99.100:2377
    This node joined a swarm as a worker.
    ```

2. 如果您没有可用命令，可以在管理器节点上运行以下命令来检索worker的join命令：
    ```shell
    $ docker swarm join-token worker

    To add a worker to this swarm, run the following command:

        docker swarm join \
        --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
        192.168.99.100:2377
    ```

3. 将终端和ssh打开到管理器节点运行的机器中，并运行docker node ls命令查看工作节点：
    ```shell
    $ docker node ls

    ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
    03g1y59jwfg7cf99w4lt0f662    worker2   Ready   Active
    9j68exjopxe7wfl6yuxml7a7j    worker1   Ready   Active
    dxn1zf6l61qsb1josjja83ngz *  manager1  Ready   Active        Leader
    ```

# 将服务部署到群集
1. 运行以下命令：
    ```shell
    $ docker service create --replicas 1 --name helloworld alpine ping docker.com

    9uk4639qpg7npwf3fn2aasksr
    ```
    1. 该docker service create命令创建服务。
    2. 该--name标志命名该服​​务helloworld。
    3. 该--replicas标志指定1个运行实例的所需状态。
    4. 参数alpine ping docker.com将服务定义为执行该命令的Alpine Linux容器ping docker.com。

2. 运行docker service ls以查看正在运行的服务列表：
    ```shell
    $ docker service ls

    ID            NAME        SCALE  IMAGE   COMMAND
    9uk4639qpg7n  helloworld  1/1    alpine  ping docker.com
    ```

3. 服务启动流程
![/service-lifecycle](/images/2017/07/service-lifecycle.png)

# 删除服务
1. 运行docker service rm helloworld删除helloworld服务。
    ```shell
    $ docker service rm helloworld

    helloworld
    ```

2. 运行docker service inspect <SERVICE-ID>以验证swarm manager是否删除了该服务。CLI返回一条消息，指出没有找到该服务：
    ```shell
    $ docker service inspect helloworld
    []
    Error: no such service: helloworld
    ```

3. 即使服务不再存在，任务容器需要几秒钟的时间来清理。您可以使用docker ps节点来验证任务何时被删除。

    ```shell
    $ docker ps

        CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
        db1651f50347        alpine:latest       "ping docker.com"        44 minutes ago      Up 46 seconds                           helloworld.5.9lkmos2beppihw95vdwxy1j3w
        43bf6e532a92        alpine:latest       "ping docker.com"        44 minutes ago      Up 46 seconds                           helloworld.3.a71i8rp6fua79ad43ycocl4t2
        5a0fb65d8fa7        alpine:latest       "ping docker.com"        44 minutes ago      Up 45 seconds                           helloworld.2.2jpgensh7d935qdc857pxulfr
        afb0ba67076f        alpine:latest       "ping docker.com"        44 minutes ago      Up 46 seconds                           helloworld.4.1c47o7tluz7drve4vkm2m5olx
        688172d3bfaa        alpine:latest       "ping docker.com"        45 minutes ago      Up About a minute                       helloworld.1.74nbhb3fhud8jfrhigd7s29we
    ```
