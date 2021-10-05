# Manjaro/ArchLinux 日常维护

## 镜像

配置镜像源

```sh
pacman-mirrors -i -c China -m rank
```

设置 archlinuxcn 源

```conf
#/etc/pacman.conf
[archlinuxcn]
SigLevel = Optional TrustedOnly
# 阿里源
Server = https://mirrors.aliyun.com/archlinuxcn/$arch
# 清华源
Server = http://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
# 中科大源
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch

[arch4edu]
SigLevel = TrustAll
Server = https://mirrors.aliyun.com/arch4edu/$arch
Server = http://mirrors.tuna.tsinghua.edu.cn/arch4edu/$arch

```

全面更新系统

```sh
pacman -Syyu
```

PGP签名

```sh
pacman -S archlinuxcn-keyring
```

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

### 输入法

```sh
pacman -S fcitx5-im fcitx5-chinese-addons fcitx5-pinyin-zhwiki
```

```conf
# ~/.pam_environment

GTK_IM_MODULE DEFAULT=fcitx
QT_IM_MODULE  DEFAULT=fcitx
XMODIFIERS    DEFAULT=\@im=fcitx
SDL_IM_MODULE DEFAULT=fcitx

```

```conf
# ~/.xprofile
export QT_IM_MODULE=fcitx5

```

## vmware 全屏

```sh
su

pacman -S open-vm-tools

pacman -Su xf86-input-vmmouse xf86-video-vmware mesa gtk2 gtkmm

echo needs_root_rights=yes >>/etc/X11/Xwrapper.config


systemctl enable vmtoolsd
systemctl restart vmtoolsd
```

开机启动的应用增加启动，配置命令为以下内容：
![](/images/2021/10/add%20start%20up%20app.png)

```sh
sudo -S <<< "pwdchars" systemctl restart vmtoolsd
```
