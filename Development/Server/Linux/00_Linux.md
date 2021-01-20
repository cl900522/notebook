## 硬盘分区

```sh

fdisk -l #查看磁盘

fdisk /dev/sdd #开始分区

partprobe /dev/sdd  #不重启系统的情况下重读分区表, 用于重读分区表，当出现删除文件后，出现仍然占用空间。

mkfs -t ext4 /dev/sdd1 # 分区格式化为ext4

blkid /dev/sdd1 # 查看磁盘uuid

```

## swap创建

```sh

swapon -s # 查看swap状态


mkswap /dev/sdb1 # 将磁盘格式化为swap

# 以下为将文件创建为swap过程

dd if=/dev/zero of=/tmp/swap  bs=1M  count=1024 # 创建swap的空文件

chmod 600 /tmp/swap

mkswap /tmp/swap

swapon [设备|文件] # 开启swap

swapoff [设备|文件] # 关闭swap


```

/etc/fstab

```conf

UUID=47fe2c83-c4b9-4ee5-ad91-4a9642f66df5 　　 swap　　swap　　 defaults 　　 0 　　0 #CentOS7-1810与openSUSE15默认的格式，UUID为对应的交换分区UUID

```
