# Docker基础用法
## 查找、下载、查看、删除镜像
```shell
    $ sudo docker search ubuntu
    $ sudo docker search -s 100 ubuntu # 查找超过100星的镜像

    $ sudo docker pull ubuntu # 获取 ubuntu 官方镜像
    $ sudo docker pull ubuntu:latest # 获取 最新ubuntu 官方镜像

    $ sudo docker images # 查看当前镜像列表

    $ sudo rmi ubuntu:12.04 # 删除ubuntu镜像
```

## 运行一个交互的shell
```shell
    $ sudo docker run -i -t ubuntu:14.04 /bin/bash

    #docker run - 运行一个容器
    #-t - 分配一个（伪）tty (link is external)
    #-i - 交互模式 (so we can interact with it)
    #ubuntu:14.04 - 使用 ubuntu 基础镜像 14.04
    #/bin/bash - 运行命令 bash shell
    #注: ubuntu 会有多个版本，通过指定 tag 来启动特定的版本 [image]:[tag]
```

## 查看、停止、删除容器
```shell
    $ sudo docker ps # 查看当前运行的容器
    IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    6c9129e9df10        ubuntu:14.04        /bin/bash 6 minutes ago       Up 6 minutes                            cranky_babbage

    $ sudo docker ps -a #列出当前系统所有的容器（包括停止的）

    $ sudo docker stop [CONTAINER-ID][CONTAINER-NAME] #停止容器运行
    $ sudo docker rm [CONTAINER-ID][CONTAINER-NAME] #删除容器
```

# 运行docker容器时发生了什么？
不论你使用docker命令或者是RESTful API，Docker客户端都会告诉Docker守护进程运行一个容器。
```shell
    $ sudo docker run -i -t ubuntu /bin/bash
```

让我们来分析这个命令。Docker客户端使用docker命令来运行，run参数表名客户端要运行一个新的容器。Docker客户端要运行一个容器需要告诉Docker守护进程的最小参数信息是：
1. 这个容器从哪个镜像创建，这里是ubuntu，基础的Ubuntu镜像。
2. 在容器中要运行的命令，这里是/bin/bash，在容器中运行Bash shell。

那么运行这个命令之后在底层发生了什么？按照顺序，Docker做了这些事情：
1. 拉取ubuntu镜像: Docker检查ubuntu镜像是否存在，如果在本地没有该镜像，Docker会从Docker Hub下载。如果镜像已经存在，Docker会使用它来创建新的容器。
2. 创建新的容器: 当Docker有了这个镜像之后，Docker会用它来创建一个新的容器。
3. 分配文件系统并且挂载一个可读写的层: 容器会在这个文件系统中创建，并且一个可读写的层被添加到镜像中。
4. 分配网络/桥接接口: 创建一个允许容器与本地主机通信的网络接口。
5. 设置一个IP地址: 从池中寻找一个可用的IP地址并且服加到容器上。
6. 运行你指定的程序: 运行指定的程序。
7. 捕获并且提供应用输出: 连接并且记录标准输出、输入和错误让你可以看到你的程序是如何运行的。

你现在拥有了一个运行着的容器！从这里开始你可以管理你的容器，与应用交互，应用完成之后，可以停止或者删除你的容器。

# Docker命令
## Docker help
``` shell
$ sudo docker   # docker 命令帮助

Commands:
    attach    Attach to a running container                 # 当前 shell 下 attach 连接指定运行镜像
    build     Build an image from a Dockerfile              # 通过 Dockerfile 定制镜像
    commit    Create a new image from a container\'s changes # 提交当前容器为新的镜像
    cp        Copy files/folders from the container\'s filesystem to the host path # 从容器中拷贝指定文件或者目录到宿主机中
    create    Create a new container                        # 创建一个新的容器，同 run，但不启动容器
    diff      Inspect changes on a container\'s filesystem   # 查看 docker 容器变化
    events    Get real time events from the server          # 从 docker 服务获取容器实时事件
    exec      Run a command in an existing container        # 在已存在的容器上运行命令
    export    Stream the contents of a container as a tar archive   # 导出容器的内容流作为一个 tar 归档文件[对应 import ]
    history   Show the history of an image                  # 展示一个镜像形成历史
    images    List images                                   # 列出系统当前镜像
    import    Create a new filesystem image from the contents of a tarball  # 从tar包中的内容创建一个新的文件系统映像[对应 export]
    info      Display system-wide information               # 显示系统相关信息
    inspect   Return low-level information on a container   # 查看容器详细信息
    kill      Kill a running container                      # kill 指定 docker 容器
    load      Load an image from a tar archive              # 从一个 tar 包中加载一个镜像[对应 save]
    login     Register or Login to the docker registry server   # 注册或者登陆一个 docker 源服务器
    logout    Log out from a Docker registry server         # 从当前 Docker registry 退出
    logs      Fetch the logs of a container                 # 输出当前容器日志信息
    port      Lookup the public-facing port which is NAT-ed to PRIVATE_PORT # 查看映射端口对应的容器内部源端口
    pause     Pause all processes within a container        # 暂停容器
    ps        List containers                               # 列出容器列表
    pull      Pull an image or a repository from the docker registry server # 从docker镜像源服务器拉取指定镜像或者库镜像
    push      Push an image or a repository to the docker registry server   # 推送指定镜像或者库镜像至docker源服务器
    restart   Restart a running container                   # 重启运行的容器
    rm        Remove one or more containers                 # 移除一个或者多个容器
    rmi       Remove one or more images                     # 移除一个或多个镜像[无容器使用该镜像才可删除，否则需删除相关容器才可继续或 -f 强制删除]
    run       Run a command in a new container              # 创建一个新的容器并运行一个命令
    save      Save an image to a tar archive                # 保存一个镜像为一个 tar 包[对应 load]
    search    Search for an image on the Docker Hub         # 在 docker hub 中搜索镜像
    start     Start a stopped containers                    # 启动容器
    stop      Stop a running containers                     # 停止容器
    tag       Tag an image into a repository                # 给源中镜像打标签
    top       Lookup the running processes of a container   # 查看容器中运行的进程信息
    unpause   Unpause a paused container                    # 取消暂停容器
    version   Show the docker version information           # 查看 docker 版本号
    wait      Block until a container stops, then print its exit code   # 截取容器停止时的退出状态值

Run 'docker COMMAND --help' for more information on a command.
```

## Docker选项
``` shell
Usage of docker:
  --api-enable-cors=false                Enable CORS headers in the remote API                      # 远程 API 中开启 CORS 头
  -b, --bridge=""                        Attach containers to a pre-existing network bridge         # 桥接网络
                                           use 'none' to disable container networking
  --bip=""                               Use this CIDR notation address for the network bridge's IP, not compatible with -b
                                         # 和 -b 选项不兼容，具体没有测试过
  -d, --daemon=false                     Enable daemon mode                                         # daemon 模式
  -D, --debug=false                      Enable debug mode                                          # debug 模式
  --dns=[]                               Force docker to use specific DNS servers                   # 强制 docker 使用指定 dns 服务器
  --dns-search=[]                        Force Docker to use specific DNS search domains            # 强制 docker 使用指定 dns 搜索域
  -e, --exec-driver="native"             Force the docker runtime to use a specific exec driver     # 强制 docker 运行时使用指定执行驱动器
  --fixed-cidr=""                        IPv4 subnet for fixed IPs (ex: 10.20.0.0/16)
                                           this subnet must be nested in the bridge subnet (which is defined by -b or --bip)
  -G, --group="docker"                   Group to assign the unix socket specified by -H when running in daemon mode
                                           use '' (the empty string) to disable setting of a group
  -g, --graph="/var/lib/docker"          Path to use as the root of the docker runtime              # 容器运行的根目录路径
  -H, --host=[]                          The socket(s) to bind to in daemon mode                    # daemon 模式下 docker 指定绑定方式[tcp or 本地 socket]
                                           specified using one or more tcp://host:port, unix:///path/to/socket, fd://* or fd://socketfd.
  --icc=true                             Enable inter-container communication                       # 跨容器通信
  --insecure-registry=[]                 Enable insecure communication with specified registries (no certificate verification for HTTPS and enable HTTP fallback) (e.g., localhost:5000 or 10.20.0.0/16)
  --ip="0.0.0.0"                         Default IP address to use when binding container ports     # 指定监听地址，默认所有 ip
  --ip-forward=true                      Enable net.ipv4.ip_forward                                 # 开启转发
  --ip-masq=true                         Enable IP masquerading for bridge's IP range
  --iptables=true                        Enable Docker's addition of iptables rules                 # 添加对应 iptables 规则
  --mtu=0                                Set the containers network MTU                             # 设置网络 mtu
                                           if no value is provided: default to the default route MTU or 1500 if no default route is available
  -p, --pidfile="/var/run/docker.pid"    Path to use for daemon PID file                            # 指定 pid 文件位置
  --registry-mirror=[]                   Specify a preferred Docker registry mirror
  -s, --storage-driver=""                Force the docker runtime to use a specific storage driver  # 强制 docker 运行时使用指定存储驱动
  --selinux-enabled=false                Enable selinux support                                     # 开启 selinux 支持
  --storage-opt=[]                       Set storage driver options                                 # 设置存储驱动选项
  --tls=false                            Use TLS; implied by tls-verify flags                       # 开启 tls
  --tlscacert="/root/.docker/ca.pem"     Trust only remotes providing a certificate signed by the CA given here
  --tlscert="/root/.docker/cert.pem"     Path to TLS certificate file                               # tls 证书文件位置
  --tlskey="/root/.docker/key.pem"       Path to TLS key file                                       # tls key 文件位置
  --tlsverify=false                      Use TLS and verify the remote (daemon: verify client, client: verify daemon) # 使用 tls 并确认远程控制主机
  -v, --version=false                    Print version information and quit
```

# Docker 网络配置
## 端口映射
无论如何，这些 ip 是基于本地系统的并且容器的端口非本地主机是访问不到的。此外，除了端口只能本地访问外，对于容器的另外一个问题是这些 ip 在容器每次启动的时候都会改变。
Docker 解决了容器的这两个问题，并且给容器内部服务的访问提供了一个简单而可靠的方法。Docker 通过端口绑定主机系统的接口，允许非本地客户端访问容器内部运行的服务。为了简便的使得容器间通信，Docker 提供了这种连接机制。

```shell
    # -P使用时需要指定--expose选项，指定需要对外提供服务的端口,使用docker run -P自动绑定所有对外提供服务的容器端口，映射的端口将会从没有使用的端口池中 (49000..49900) 自动选择，
    $ sudo docker run -t -P --expose 22 --name server  ubuntu:14.04

    $ sudo docker run -p [([<host_interface>:[host_port]])|(<host_port>):]<container_port>[/udp] <image> <cmd> # 默认不指定绑定 ip 则监听所有网络接口

    # Bind TCP port 8080 of the container to TCP port 80 on 127.0.0.1 of the host machine. $ sudo docker run -p 127.0.0.1:80:8080 <image> <cmd> # Bind TCP port 8080 of the container to a dynamically allocated TCP port on 127.0.0.1 of the host machine. $ sudo docker run -p 127.0.0.1::8080 <image> <cmd> # Bind TCP port 8080 of the container to TCP port 80 on all available interfaces of the host machine.
    $ sudo docker run -p 80:8080 <image> <cmd> # Bind TCP port 8080 of the container to a dynamically allocated TCP port on all available interfaces $ sudo docker run -p 8080 <image> <cmd>

    # Bind UDP port 5353 of the container to UDP port 53 on 127.0.0.1 of the host machine.
    $ sudo docker run -p 127.0.0.1:53:5353/udp <image> <cmd>
```

你可以通过docker ps、docker inspect <container_id>或者docker port <container_id> <port>确定具体的绑定信息。
```shell
    $ sudo docker inspect <container_id> | grep IPAddress | cut -d ’"’ -f 4 # Find IP address of container with ID <container_id> 通过容器 id 获取 ip
```

## 网络配置
![net1](/images/2017/01/n1.png)
Dokcer 通过使用 Linux 桥接提供容器之间的通信，docker0 桥接接口的目的就是方便 Docker 管理。当 Docker daemon 启动时需要做以下操作：
1. creates the docker0 bridge if not present # 如果 docker0 不存在则创建
2. searches for an IP address range which doesn’t overlap with an existing route # 搜索一个与当前路由不冲突的 ip 段
3. picks an IP in the selected range # 在确定的范围中选择 ip
4. assigns this IP to the docker0 bridge # 绑定 ip 到 docker0

docker run 创建 Docker 容器时，可以用 --net 选项指定容器的网络模式，Docker 有以下 4 种网络模式：
1. host 模式，使用 --net=host 指定。
2. container 模式，使用 --net=container:NAMEorID 指定。
3. none 模式，使用 --net=none 指定。
4. bridge 模式，使用 --net=bridge 指定，默认设置。

### host 模式
如果启动容器的时候使用 host 模式，那么这个容器将不会获得一个独立的 Network Namespace，而是和宿主机共用一个 Network Namespace。容器将不会虚拟出自己的网卡，配置自己的 IP 等，而是使用宿主机的 IP 和端口。
例如，我们在 10.10.101.105/24 的机器上用 host 模式启动一个含有 web 应用的 Docker 容器，监听 tcp 80 端口。当我们在容器中执行任何类似 ifconfig 命令查看网络环境时，看到的都是宿主机上的信息。而外界访问容器中的应用，则直接使用 10.10.101.105:80 即可，不用任何 NAT 转换，就如直接跑在宿主机中一样。但是，容器的其他方面，如文件系统、进程列表等还是和宿主机隔离的。

### container 模式
这个模式指定新创建的容器和已经存在的一个容器共享一个 Network Namespace，而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的 IP，而是和一个指定的容器共享 IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。两个容器的进程可以通过 lo 网卡设备通信。

### none模式
这个模式和前两个不同。在这种模式下，Docker 容器拥有自己的 Network Namespace，但是，并不为 Docker容器进行任何网络配置。也就是说，这个 Docker 容器没有网卡、IP、路由等信息。需要我们自己为 Docker 容器添加网卡、配置 IP 等。

### bridge模式
![net2](/images/2017/01/n2.png)
bridge 模式是 Docker 默认的网络设置，此模式会为每一个容器分配 Network Namespace、设置 IP 等，并将一个主机上的 Docker 容器连接到一个虚拟网桥上。当 Docker server 启动时，会在主机上创建一个名为 docker0 的虚拟网桥，此主机上启动的 Docker 容器会连接到这个虚拟网桥上。虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个二层网络中。接下来就要为容器分配 IP 了，Docker 会从 RFC1918 所定义的私有 IP 网段中，选择一个和宿主机不同的IP地址和子网分配给 docker0，连接到 docker0 的容器就从这个子网中选择一个未占用的 IP 使用。如一般 Docker 会使用 172.17.0.0/16 这个网段，并将 172.17.42.1/16 分配给 docker0 网桥（在主机上使用 ifconfig 命令是可以看到 docker0 的，可以认为它是网桥的管理接口，在宿主机上作为一块虚拟网卡使用）

```shell
    $ sudo brctl show # 查看当前主机网桥 brctl 工具依赖 bridge-utils 软件包 bridge name bridge id STP enabled interfaces
    docker0 8000.000000000000 no
    $ sudo ifconfig docker0
    docker0 Link encap:Ethernet HWaddr xx:xx:xx:xx:xx:xx
    inet addr:172.17.42.1 Bcast:0.0.0.0 Mask:255.255.0.0
```
在容器运行时，每个容器都会分配一个特定的虚拟机口并桥接到 docker0。每个容器都会配置同 docker0 ip 相同网段的专用 ip 地址，docker0 的 IP 地址被用于所有容器的默认网关。

```shell
    $ sudo docker run -t -i -d ubuntu /bin/bash
    52f811c5d3d69edddefc75aff5a4525fc8ba8bcfa1818132f9dc7d4f7c7e78b4
    $ sudo brctl show
    bridge name bridge id STP enabled interfaces
    docker0 8000.fef213db5a66 no vethQCDY1N
```
以上, docker0 扮演着 52f811c5d3d6 container 这个容器的虚拟接口 vethQCDY1N interface 桥接的角色。

使用特定范围的 IP
Docker 会尝试寻找没有被主机使用的 ip 段，尽管它适用于大多数情况下，但是它不是万能的，有时候我们还是需要对 ip 进一步规划。Docker 允许你管理 docker0 桥接或者通过-b选项自定义桥接网卡，需要安装bridge-utils软件包。

本步骤如下：
1. ensure Docker is stopped # 确保 docker 的进程是停止的
2. create your own bridge (bridge0 for example) # 创建自定义网桥
3. assign a specific IP to this bridge # 给网桥分配特定的 ip
4. start Docker with the -b=bridge0 parameter # 以 -b 的方式指定网桥

```shell
    # Stopping Docker and removing docker0
    $ sudo service docker stop
    $ sudo ip link set dev docker0 down
    $ sudo brctl delbr docker0 # Create our own bridge
    $ sudo brctl addbr bridge0
    $ sudo ip addr add 192.168.5.1/24 dev bridge0
    $ sudo ip link set dev bridge0 up # Confirming that our bridge is up and running
    $ ip addr show bridge0

    4: bridge0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state UP group default
    link/ether 66:38:d0:0d:76:18 brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.1/24 scope global bridge0
    valid_lft forever preferred_lft forever # Tell Docker about it and restart (on Ubuntu)
    $ echo 'DOCKER_OPTS="-b=bridge0"' >> /etc/default/docker
    $ sudo service docker start
```
