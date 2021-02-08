
# Linux

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

## modprobe

/lib/modules/\<kernel-version\> 存储对应内核版本的模块,\<kernel-version\>为uname -r的结果

系统启动后，正常工作的模块都在/proc/modules文件中列出。使用lsmod命今也可显示相同内容。

modprobe可以解决load module时的依赖关系，它是通过/lib/modules/\<kernel-version\>/modules.dep文件来查找依赖关系的

在内核中有一个“Automatic kernel module loading"功能被编译到了内核中。/etc/modules-load.d/*、/etc/modules.conf、 /etc/modprobe.conf文件是一个自动处理内核模块的控制文件

```sh

modprobe -a # 加载所有匹配模块

modprobe -r filename #卸载模块

modprobe filename

moinfo 模块名 #查看模块信息

```
