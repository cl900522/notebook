高级Docker
=========

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

# 容器数据管理
## 数据卷
数据卷是一个或多个容器专门指定绕过Union File System的目录，为持续性或共享数据提供一些有用的功能：

1. 数据卷可以在容器间共享和重用
2. 数据卷数据改变是直接修改的
3. 数据卷数据改变不会被包括在容器中
4. 数据卷是持续性的，直到没有容器使用它们

添加一个数据卷：你可以使用-v选项添加一个数据卷，或者可以使用多次-v选项为一个 docker 容器运行挂载多个数据卷。
```
$ sudo docker run --name data -v /data -t -i ubuntu:14.04 /bin/bash
# 创建数据卷绑定到到新建容器，新建容器中会创建 /data 数据卷 bash-4.1# ls -ld /data/
drwxr-xr-x 2 root root 4096 Jul 23 06:59 /data/
bash-4.1# df -Th
Filesystem    Type    Size  Used Avail Use% Mounted on
... ...
              ext4     91G  4.6G   82G   6% /data
```
创建的数据卷可以通过docker inspect获取宿主机对应路径
```
$ sudo docker inspect data
... ... "Volumes": { "/data": "/var/lib/docker/vfs/dir/151de401d268226f96d824fdf444e77a4500aed74c495de5980c807a2ffb7ea9" }, # 可以看到创建的数据卷宿主机路径 ... ...
```
或者直接指定获取
```
$ sudo docker inspect --format="{{ .Volumes }}" data
map[/data: /var/lib/docker/vfs/dir/151de401d268226f96d824fdf444e77a4500aed74c495de5980c807a2ffb7ea9]
```

挂载宿主机目录为一个数据卷
-v选项除了可以创建卷，也可以挂载当前主机的一个目录到容器中。
```
$ sudo docker run --name web -v /source/:/web -t -i ubuntu:14.04 /bin/bash
bash-4.1# ls -ld /web/
drwxr-xr-x 2 root root 4096 Jul 23 06:59 /web/
bash-4.1# df -Th
... ...
              ext4     91G  4.6G   82G   6% /web
bash-4.1# exit
```
默认挂载卷是可读写的，可以在挂载时指定只读
```
$ sudo docker run --rm --name test -v /source/:/test:ro -t -i ubuntu:14.04 /bin/bash
```

## 共享数据卷
如果你有一些持久性的数据并且想在容器间共享，或者想用在非持久性的容器上，最好的方法是创建一个数据卷容器，然后从此容器上挂载数据。

创建数据卷容器
```
$ sudo docker run -t -i -d -v /test --name test ubuntu:14.04 echo hello
```
使用--volumes-from选项在另一个容器中挂载 /test 卷。不管 test 容器是否运行，其它容器都可以挂载该容器数据卷，当然如果只是单独的数据卷是没必要运行容器的。
```
$ sudo docker run -t -i -d --volumes-from test --name test1 ubuntu:14.04 /bin/bash
```
添加另一个容器
```
$ sudo docker run -t -i -d --volumes-from test --name test2 ubuntu:14.04 /bin/bash
```
也可以继承其它挂载有 /test 卷的容器
```
$ sudo docker run -t -i -d --volumes-from test1 --name test3 ubuntu:14.04 /bin/bash
```
![共享Volumn](/images/2017/02/volumn1.png)
### 备份、恢复或迁移数据卷
备份
```
$ sudo docker run --rm --volumes-from test -v $(pwd):/backup ubuntu:14.04 tar cvf /backup/test.tar /test
tar: Removing leading `/' from member names
/test/
/test/b
/test/d
/test/c
/test/a
```
启动一个新的容器并且从test容器中挂载卷，然后挂载当前目录到容器中为 backup，并备份 test 卷中所有的数据为 test.tar，执行完成之后删除容器--rm，此时备份就在当前的目录下，名为test.tar。
```
$ ls # 宿主机当前目录下产生了 test 卷的备份文件 test.tar test.tar
```
恢复
你可以恢复给同一个容器或者另外的容器，新建容器并解压备份文件到新的容器数据卷
```
$ sudo docker run -t -i -d -v /test --name test4 ubuntu:14.04  /bin/bash $ sudo docker run --rm --vol
```

### 删除数据卷
Volume 只有在下列情况下才能被删除：
1. docker rm -v删除容器时添加了-v选项
2. docker run --rm运行容器时添加了--rm选项
否则，会在/var/lib/docker/volumns/目录中遗留很多不明目录。

## 链接容器
docker 允许把多个容器连接在一起，相互交互信息。docker 链接会创建一种容器父子级别的关系，其中父容器可以看到其子容器提供的信息。
### 容器命名
在创建容器时，如果不指定容器的名字，则默认会自动创建一个名字，这里推荐给容器命名：

1. 给容器命名方便记忆，如命名运行 web 应用的容器为 web
2. 为 docker 容器提供一个参考，允许方便其他容器调用，如把容器 web 链接到容器 db

可以通过--name选项给容器自定义命名：
```
$ sudo docker run -d -t -i --name test ubuntu:14.04 bash
$ sudo docker  inspect --format="{{ .Name }}" test
/test
```
>注：容器名称必须唯一，即你只能命名一个叫test的容器。如果你想复用容器名，则必须在创建新的容器前通过docker rm删除旧的容器或者创建容器时添加--rm选项。

### 链接容器
链接允许容器间安全通信，使用--link选项创建链接。
```
$ sudo docker run -d --name db training/postgres
```
基于 training/postgres 镜像创建一个名为 db 的容器，然后下面创建一个叫做 web 的容器，并且将它与 db 相互连接在一起
```
$ sudo docker run -d -P --name web --link db:db training/webapp python app.py
--link <name or id>:alias选项指定链接到的容器。
```
查看 web 容器的链接关系:
```
$ sudo docker inspect -f "{{ .HostConfig.Links }}" web
[/db:/web/db]
```

可以看到 web 容器被链接到 db 容器为/web/db，这允许 web 容器访问 db 容器的信息。

**容器之间的链接实际做了什么？**一个链接允许一个源容器提供信息访问给一个接收容器。在本例中，web 容器作为一个接收者，允许访问源容器 db 的相关服务信息。Docker 创建了一个安全隧道而不需要对外公开任何端口给外部容器，因此不需要在创建容器的时候添加-p或-P指定对外公开的端口，这也是链接容器的最大好处，本例为 PostgreSQL 数据库。

Docker 主要通过以下两个方式提供连接信息给接收容器：
1. 环境变量
2. 更新/etc/hosts文件

**环境变量**
当两个容器链接，Docker 会在目标容器上设置一些环境变量，以获取源容器的相关信息。

首先，Docker 会在每个通过--link选项指定别名的目标容器上设置一个<alias>_NAME环境变量。如果一个名为 web 的容器通过--link db:webdb被链接到一个名为 db 的数据库容器，那么 web 容器上会设置一个环境变量为WEBDB_NAME=/web/webdb.

以之前的为例，Docker 还会设置端口变量:
```
$ sudo docker run --rm --name web2 --link db:db training/webapp env
. . .
DB_NAME=/web2/db
DB_PORT=tcp://172.17.0.5:5432
DB_PORT_5432_TCP=tcp://172.17.0.5:5432  # <name>_PORT_<port>_<protocol> 协议可以是 TCP 或 UDP
DB_PORT_5432_TCP_PROTO=tcp
DB_PORT_5432_TCP_PORT=5432
DB_PORT_5432_TCP_ADDR=172.17.0.5
. . .
```

>注：这些环境变量只设置给容器中的第一个进程，类似一些守护进程 (如 sshd ) 当他们派生 shells 时会清除这些变量

**更新/etc/hosts文件**
除了环境变量，Docker 会在目标容器上添加相关主机条目到/etc/hosts中，上例中就是 web 容器。
```
$ sudo docker run -t -i --rm --link db:db training/webapp /bin/bash
root@aed84ee21bde:/opt/webapp# cat /etc/hosts
172.17.0.7  aed84ee21bde
. . .
172.17.0.5  db
```
/etc/host文件在源容器被重启之后会自动更新 IP 地址，而环境变量中的 IP 地址则不会自动更新的。
