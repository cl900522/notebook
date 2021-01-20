
/etc/default/grub

```conf

# If you change this file, run 'update-grub' afterwards to update
# /boot/grub/grub.cfg.
# For full documentation of the options in this file, see:
#   info -f grub -n 'Simple configuration'

GRUB_DEFAULT=0
#属性名：默认启动项（就是我要的开机默认启动系统）
#值说明：
#   数字：从0开始（按照开机选择界面的顺序对应）
#   saved:默认上次的启动项

GRUB_SAVEDEFAULT=true
# 将下次选择的启动项设为默认

#GRUB_HIDDEN_TIMEOUT=0
#属性名：是否隐藏菜单（grub2不再使用）
#值说明：0：不隐藏，1：隐藏

GRUB_HIDDEN_TIMEOUT_QUIET=true
#属性名：是否显示等待倒计时
#值说明：true：不显示，false：显示

GRUB_TIMEOUT=10
#属性名：进入默认启动项的等候时间
#值说明：单位：秒，默认10秒，-1表示一直等待

GRUB_TIMEOUT_STYLE=hidden
# GRUB_TIMEOUT_STYLE=menu

GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`

GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
#属性名：内核启动参数的默认值
#值说明：quiet splash为不显示启动信息，安静的启动，如值为空则显示启动信息

GRUB_CMDLINE_LINUX=""
#属性名：手动添加内核启动参数
#值说明：默认为空，可以添加你需要的参数，以 “name=value” 的格式添加，多个参数用空格隔开
#例如：GRUB_CMDLINE_LINUX="name1=value1 name2=value2"

# Uncomment to enable BadRAM filtering, modify to suit your needs
# This works with Linux (no patch required) and with any kernel that obtains
# the memory map information from GRUB (GNU Mach, kernel of FreeBSD ...)
#GRUB_BADRAM="0x01234567,0xfefefefe,0x89abcdef,0xefefefef"

# Uncomment to disable graphical terminal (grub-pc only)
#GRUB_TERMINAL=console
#属性名：是否使用图形介面
#值说明：默认使用图像界面，去掉前面的“#”则使用控制台终端

# The resolution used on graphical terminal
# note that you can use only modes which your graphic card supports via VBE
# you can see them in real GRUB with the command `vbeinfo'

#GRUB_GFXMODE=640x480
#属性名：图形界面分辨率
#值说明：分辨率啦（还要怎么说明），修改时记得去掉前面的“#”

# Uncomment if you don't want GRUB to pass "root=UUID=xxx" parameter to Linux
#GRUB_DISABLE_LINUX_UUID=true
#属性名：grub命令是否使用UUID
#值说明：不知道是干什么的，不常用（如果你知道，欢迎留言，谢谢）

# Uncomment to disable generation of recovery mode menu entries
#GRUB_DISABLE_RECOVERY="true"
#属性名：是否创建修复模式菜单项
#值说明：true:禁用，false：使用，默认false

# Uncomment to get a beep at grub start
#GRUB_INIT_TUNE="480 440 1"
#属性名：启动时发出哔哔声
#值说明：默认不发声，去掉“#”则发声，值是什么意思不明白（应该是发出声音方式吧）

```