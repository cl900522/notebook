NAS
=========

# 安装应用
```shell
sudo pacman -S aria2

sudo pacman -S samba
sudo pacman -S manjaro-settings-samba

sudo pacman -S nfs-utils
sudo pacman -S rpcbind
```

# 配置
编辑 /etc/samba/smb.conf

```shell
[homes]
   comment = Home Directories
   path = /share/volume2/homes/%u
   browseable = yes
   read only = no
   writable = yes
   create mask = 0700
   directory mask = 0700
   force user = %u

[HomeShare]
   comment = HomeShare
   path = /share/volume1/HomeShare
   browseable = yes
   writable = yes
   read only = no
   create mask = 0777
   directory mask = 0777
   guest ok = yes

[Temp]
   comment = Temp
   path = /share/volume3/Temp
   browseable = yes
   writable = yes
   read only = no
   create mask = 0777
   directory mask = 0777
   guest ok = yes
```

配置/etc/exports
```
/share/volume1/HomeShare    *(rw,no_root_squash,no_subtree_check,sync)
/share/volume2/home    *(rw,no_root_squash,no_subtree_check,sync)
/share/volume3/Temp    *(rw,no_root_squash,no_subtree_check,sync)
```



# 启动

```shell
# 如果磁盘不是开机自动挂载的，最好不要开机启动这些服务
sudo systemctl enable rpcbind.service
sudo systemctl enable nfs-server.service
sudo systemctl enable smb


sudo systemctl start smb
sudo systemctl start rpcbind.service
sudo systemctl start nfs-server.service
sudo systemctl start docker

aria2c --conf-path=/home/chenmx/software/aria2/aria2c.conf -D
nohup node /home/chenmx/software/webui-aria2/node-server.js &

```

# 挂载
```shell
sudo mkdir -p /share/HomeShare
sudo mkdir -p /share/homes
sudo mkdir -p /share/Temp

sudo mount -t nfs -o nolock -o tcp 192.168.100.100:/share/volume1/HomeShare /share/HomeShare
sudo mount -t nfs -o nolock -o tcp 192.168.100.100:/share/volume2/homes /share/homes
sudo mount -t nfs -o nolock -o tcp 192.168.100.100:/share/volume3/Temp /share/Temp
```

编辑/etc/fstab
```
192.168.100.100:/share/volume1/HomeShare /share/HomeShare nfs defaults 0 0
192.168.100.100:/share/volume3/Temp /share/Temp nfs defaults 0 0
192.168.100.100:/share/volume2/homes /share/homes nfs defaults 0 0
```