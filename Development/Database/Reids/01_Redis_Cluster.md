Redis集群
=================

# 概述
>Redis3.0版本之后支持Cluster.

## redis cluster的现状
目前redis支持的cluster特性：
* 节点自动发现
* slave->master 选举,集群容错
* Hot resharding:在线分片
* 进群管理:cluster xxx
* 基于配置(nodes-port.conf)的集群管理
* ASK 转向/MOVED 转向机制.

## redis cluster 架构
![redis-cluster架构图](/images/2018/11/redis-cluster01.jpg)

架构细节:
* 所有的redis节点彼此互联(PING-PONG机制),内部使用二进制协议优化传输速度和带宽.
* 节点的fail是通过集群中超过半数的节点检测失效时才生效.
* 客户端与redis节点直连,不需要中间proxy层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可
* redis-cluster把所有的物理节点映射到[0-16383]slot上,cluster 负责维护node<->slot<->value

## redis-cluster选举:容错
![选举:容错](/images/2018/11/redis-cluster02.jpg)
* 领着选举过程是集群中所有master参与,如果半数以上master节点与master节点通信超过(cluster-node-timeout),认为当前master节点挂掉.
* 什么时候整个集群不可用(cluster_state:fail),当集群不可用时,所有对集群的操作做都不可用，收到((error) CLUSTERDOWN The cluster is down)错误
    * 如果集群任意master挂掉,且当前master没有slave.集群进入fail状态,也可以理解成进群的slot映射[0-16383]不完成时进入fail状态.
    * 如果进群超过半数以上master挂掉，无论是否有slave集群进入fail状态.
