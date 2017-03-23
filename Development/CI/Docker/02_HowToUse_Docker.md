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
    service   Manage Docker services
    start     Start a stopped containers                    # 启动容器
    stats     Display a live stream of container(s) resource usage statistics
    stop      Stop a running containers                     # 停止容器
    swarm     Manage Docker Swarm
    tag       Tag an image into a repository                # 给源中镜像打标签
    top       Lookup the running processes of a container   # 查看容器中运行的进程信息
    unpause   Unpause a paused container                    # 取消暂停容器
    update    Update configuration of one or more containers
    version   Show the docker version information           # 查看 docker 版本号
    volume    Manage Docker volumes
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
