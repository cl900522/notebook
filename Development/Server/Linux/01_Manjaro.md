# Manjaro/ArchLinux 日常维护

## 日志

```sh

journalctl # 查询systemd日志
journalctl -u unit

```

```sh

dmesg #打印内核 ring buffer
dmesg -H

```

## 软件安装更新

### Pacman

```sh

pacman -Ssu # 更新本地库
pacman -R ${pname} # 删除包
pacman -Ssy ${pname} # 安装包
pacman -U ${/path/to/package/file} # 安装本地包

```

Pacman 不会更新在IgnorePkg和IgnoreGroup中的软件包

```conf

# /etc/pacman.conf

IgnorePkg = libvirt
IgnoreGroup = libvirt

```

```sh

#更新 archlinux-cn 的 PGP keys
pacman -S archlinuxcn-keyring

```

```conf

# /etc/pacman.conf
# SigLevel option determines the level of trust required to install a package.

# The SigLevel TrustAll option exists for debugging purposes and makes it very easy to trust keys that have not been verified. You should use TrustedOnly for all official repositories.

SigLevel = Required DatabaseOptional
SigLevel = Required DatabaseOptional TrustedOnly
SigLevel = PackageRequired
SigLevel = TrustAll

```

### 网络工具

```sh

pacman -S net-tools dnsutils inetutils iproute2


```

### 升级系统

```sh

# 替换pacman -Ssy
pamac update

```

### 时区

```sh

# timedatectl set-local-rtc 1
timedatectl set-local-rtc true

```
