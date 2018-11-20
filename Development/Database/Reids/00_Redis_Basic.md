Redis
=======

# 特点
## 优点：
1 读写性能优异，从内存当中进行IO读写速度快。
2 支持数据持久化，支持AOF和RDB两种持久化方式(由于Redis的数据都存放在内存中，如果没有配置持久化，redis重启后数据就全丢失了，于是需要开启redis的持久化功能，将数据保存到磁 盘上，当redis重启后，可以从磁盘中恢复数据。redis提供两种方式进行持久化，一种是RDB持久化:指在指定的时间间隔内将内存中的数据集快照写入磁盘，实际操作过程是fork一个子进程，先将数据集写入临时文件，写入成功后，再替换之前的文件，用二进制压缩存储。还有一种是AOF持久化：以日志的形式记录服务器所处理的每一个写、删除操作，查询操作不会记录，以文本的方式记录，可以打开文件看到详细的操作记录。）
3 支持主从复制，主机会自动将数据同步到从机，可以进行读写分离。
4 数据结构丰富：除了支持string类型的value外还支持string、hash、set、sortedset、list等数据结构。
5 Redis是单线程多CPU，这样速度更快。因为单线程，没有线程切换的开销，不需要考虑加锁释放锁，也就没有死锁的问题。单线程-多路复用IO模型。效率高。

## 缺点：
1 主从同步，如果主机宕机，宕机前有一部分数据没有同步到从机，会导致数据不一致。
2 主从同步，数据同步会有延迟。
3 读写分离，主机写的负载量太大，也会导致主机的宕机

## 扩容：
1 集群，使用代理，达到集群的目的。
2 主从同步，读写分离。

# 持久化机制
Redis是一个支持持久化的内存数据库，也就是说redis需要经常将内存中的数据同步到硬盘来保证持久化。

redis支持两种持久化的方式：1、snapshotting（快照）也是默认方式  2、Append-only file（缩写是aof）的方式

## snapshotting（快照）也是默认方式
刚装完redis的时候没有dump.rdb文件，满足快照要求后，系统会自动生成dump.rdb文件，且文件类型为二进制文件。
![](/images/2018/11/redis-basic02.png)

## Append-only file（缩写是aof）的方式
由于快照方式是在一定的时间间隔做一次，所以redis宕机的话会丢失最后一次快照以后修改的内容，aof有更好的持久化性，使用aof时，redis会将每一个收到的写命令都通过write函数追加到文件中，当redis重启时，会通过重新执行文件中保存的写命令在内存中重建整个数据库的内容。但是由于操作系统（OS）会在内核中缓存write做的修改，所以可能不是立即写到磁盘上的。这样aof方式的持久化还是有可能会丢失部分修改。可以通过配置文件告诉redis我们想通过fsync函数强制OS写入到磁盘的时机。

配置文件为redis.conf
将no改为yes，满足aof方式后，系统会自动生成一个appendonly.aof文件，且非二进制，用cat查看，里面保存的是刚新加的命令。
![](/images/2018/11/redis-basic01.png)

# 主从同步
Redis的主从同步机制可以确保redis的master和slave之间的数据同步。按照同步内容的多少可以分为全同步和部分同步；按照同步的时机可以分为slave刚启动时的初始化同步和正常运行过程中的数据修改同步；本文将对这两种机制的流程进行分析。

# 全备份过程中
，在slave启动时，会向其master发送一条SYNC消息，master收到slave的这条消息之后，将可能启动后台进程进行备份，备份完成之后就将备份的数据发送给slave，初始时的全同步机制是这样的：

（1）slave启动后向master发送同步指令SYNC，master接收到SYNC指令之后将调用该命令的处理函数syncCommand（）进行同步处理；
（2）在函数syncCommand中，将调用函数rdbSaveBackground启动一个备份进程用于数据同步，如果已经有一个备份进程在运行了，就不会再重新启动了。
（3）备份进程将执行函数rdbSave（）完成将redis的全部数据保存为rdb文件。
（4）在redis的时间事件函数serverCron（redis的时间处理函数是指它会定时被redis进行操作的函数）中，将对备份后的数据进行处理，在serverCron函数中将会检查备份进程是否已经执行完毕，如果备份进程已经完成备份，则调用函数backgroundSaveDoneHandler完成后续处理。
（5）在函数backgroundSaveDoneHandler中，首先更新master的各种状态，例如，备份成功还是失败，备份的时间等等。然后调用函数updateSlavesWaitingBgsave，将备份的rdb数据发送给等待的slave。
（6）在函数updateSlavesWaitingBgsave中，将遍历所有的等待此次备份的slave，将备份的rdb文件发送给每一个slave。另外，这里并不是立即就把数据发送过去，而是将为每个等待的slave注册写事件，并注册写事件的响应函数sendBulkToSlave，即当slave对应的socket能够发送数据时就调用函数sendBulkToSlave（），实际发送rdb文件的操作都在函数sendBulkToSlave中完成。
（7）sendBulkToSlave函数将把备份的rdb文件发送给slave。

![redis全备份时master部分的的函数调用过程](/images/2018/11/redis-sync01.jpg)

## 数据修改同步
Redis的正常部署中一般都是一个master用于写操作，若干个slave用于读操作，另外定期的数据备份操作也是单独选址一个slave完成，这样可以最大程度发挥出redis的性能。在部署完成，各master\slave程序启动之后，首先进行第一阶段初始化时的全同步操作，全同步操作完成之后，后续所有写操作都是在master上进行，所有读操作都是在slave上进行，因此用户的写操作需要及时扩散到所有的slave以便保持数据最大程度上的同步。Redis的master-slave进程在正常运行期间更新操作（包括写、删除、更改操作）的同步方式如下：

（1）master接收到一条用户的操作后，将调用函数call函数来执行具体的操作函数（此过程可参考另一文档《redis命令执行流程分析》），在该函数中首先通过proc执行操作函数，然后将判断操作是否需要扩散到各slave，如果需要则调用函数propagate（）来完成此操作。
（2）propagate（）函数完成将一个操作记录到aof文件中或者扩散到其他slave中；在该函数中通过调用feedAppendOnlyFile（）将操作记录到aof中，通过调用replicationFeedSlaves（）将操作扩散到各slave中。
（3）函数feedAppendOnlyFile（）中主要保存操作到aof文件，在该函数中首先将操作转换成redis内部的协议格式，并以字符串的形式存储，然后将字符串存储的操作追加到aof文件后。
（4）函数replicationFeedSlaves（）主要将操作扩散到每一个slave中；在该函数中将遍历自己下面挂的每一个slave，以此对每个slave进行如下两步的处理：将slave的数据库切换到本操作所对应的数据库（如果slave的数据库id与当前操作的数据id不一致时才进行此操作）；将命令和参数按照redis的协议格式写入到slave的回复缓存中。写入切换数据库的命令时将调用addReply，写入命令和参数时将调用addReplyMultiBulkLen和addReplyBulk，函数addReplyMultiBulkLen和addReplyBulk最终也将调用函数addReply。
（5）在函数addReply中将调用prepareClientToWrite（）设置slave的socket写入事件处理函数sendReplyToClient（通过函数aeCreateFileEvent进行设置），这样一旦slave对应的socket发送缓存中有空间写入数据，即调用sendReplyToClient进行处理。
（6）函数sendReplyToClient（）的主要功能是将slave中要发送的数据通过socket发出去。

![redis操作过程中数据同步的函数调用关系](/images/2018/11/redis-sync02.jpg)

# 参考

redis 双写一致性
https://blog.csdn.net/hjm4702192/article/details/80518922

分布式数据库与缓存双写一致性方案解疑
https://blog.csdn.net/weixin_40581617/article/details/80567279

Redis实现分布式锁原理与实现分析
https://www.cnblogs.com/SUNSHINEC/p/8302540.html
