[TOC]


# Intall Arch Linux
[虚拟机教程](http://blog.csdn.net/jaina_proudmoore/article/details/52589135)

##一、分区
1. 在VMware里开启虚拟机  
  第一项是64位的，第二项是32位的，第三项是已有的系统。  
  ```
  *Boot Arch Linux (x86_64)
  *Boot Arch linux (i686)
  *Boot existing OS
  ```

2. 开始分区，输入命令。  
`fdisk /dev/sda`

* 创建分区表：  

```
· Command (m for help): 输入 o 并按下 Enter 
然后建立第一个分区：
1. Command (m for help): 输入 n 并按下 Enter 
2. Partition type: Select (default p): 按下 Enter 
3. Partition number (1-4, default 1): 按下 Enter 
4. First sector (2048-209715199, default 2048): 按下 Enter 
5. Last sector, +sectors or +size{K,M,G} (2048-209715199....., default 209715199): 输入 +15G 并按下 Enter 
然后建立第二个分区：
1. Command (m for help): 输入 n 并按下 Enter 
2. Partition type: Select (default p): 按下 Enter 
3. Partition number (1-4, default 2): 按下 Enter 
4. First sector (31459328-209715199, default 31459328): 按下 Enter 
5. Last sector, +sectors or +size{K,M,G} (31459328-209715199....., default 209715199): 按下 Enter 
现在预览下新的分区表:
· Command (m for help): 输入 p 并按下 Enter 
Disk /dev/sda: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x5698d902
Device Boot     Start         End     Blocks   Id  System
/dev/sda1           2048    31459327   15728640   83   Linux
/dev/sda2       31459328   209715199   89127936   83   Linux
然后向磁盘写入这些改动：
· Command (m for help): 输入 w 并按下 Enter 
```

如果一切顺利无错误的话，fdisk 程序将显示如下信息:  
The partition table has been altered!  
Calling ioctl() to re-read partition table.  
Syncing disks.  
若因 fdisk 遇到错误导致以上操作无法完成，可以用 q 命令来退出。  
（当然你也可以分多个分区，分别挂载/boot,/home/,/,/var等）  

1. 接下来格式化成ext4文件系统
```
mkfs.ext4 /dev/sda1
mkfs.ext4 /dev/sda2
```

>  （ 若您分了一个 swap 区，也不要忘了格式化并启用它（x代表你的那个分区数字）：
```
# mkswap /dev/sdaX
# swapon /dev/sdaX
```

注意要按照顺序挂载，先挂载根分区到 /mnt （你实际所要挂载的分区名当然可能会不同）：  
```mount /dev/sda1 /mnt```  
然后挂载 /home 分区（以及其它其余单独分区，比如 /boot，/var，如果您有的话）：  
```mkdir /mnt/home```  
```mount /dev/sda2 /mnt/home```  
（如果有其他分区，先创建目录，再挂载）  

## 二、安装基本系统

重申一遍，这里及以后一些步骤必须联网，尤其是运行pacman命令时。关于联网问题请参照archwiki,里面有十分详细的解说。  
2. 安装前需要编辑文件`/etc/pacman.d/mirrorlist`  
  你的系统和软件将从这里的地址下载。将偏好的镜像放到最前面，下面加入了一个比较快的源，当然你可以去网上搜其他比较好的源：
  `nano /etc/pacman.d/mirrorlist`

```
##
## Arch Linux repository mirrorlist
## Sorted by mirror score from mirror status page
## Generated on 2012-MM-DD
##
```  
加入  
`Server=http://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch`  
`Server=http://mirrors.bjtu.edu.cn/archlinux/$repo/os/$arch`


如果您愿意，您可以只使用一个镜像并全删光其他行，但为保险，还是留其他几个离您较近的镜像作备用好
然后敲入：
```
# pacman -Syy          刷新列表
# pacstrap -i /mnt base    安装基本系统
```
若运行 pacstrap 时卡住并出现 `failed retrieving file 'core.db' from mirror... : Connection time-out` 字样，请检查刚才的源是否正确或去网上搜索其他能用的源。  
2. 生成fstab分区表  
```
genfstab -U -p /mnt >> /mnt/etc/fstab
```
生成完成之后必须查看一下fstab是否生成成功。
`nano /mnt/etc/fstab`

2. 下面要 chroot 到新系统开始配置：
```
# arch-chroot /mnt /bin/bash
```
2. 系统本地化，设置本地语言，地点等信息
```
# nano /etc/locale.gen
```
```
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
```
接着执行locale-gen以生成locale讯息：
```
# locale-gen
```
（创建 locale.conf 并提交您的本地化选项：
```
# echo LANG=en_US.UTF-8 > /etc/locale.conf
```
这里先不要设置中文编码，等安装了图形界面再修改，否则会乱码）
5、设置时区，一般以上海就行：
`ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`
6、设置时间
`# hwclock --systohc --utc`
7、设置个您喜欢的主机名，例如：
`# echo 主机名 > /etc/hostname`
并在 /etc/hosts 添加同样的主机名：
`# nano /etc/hosts`
```
/etc/hosts: static lookup table for host names
 
#<ip-address> <hostname.domain.org> <hostname>
127.0.0.1    localhost.localdomain  localhost 主机名   
::1          localhost.localdomain  localhost

# End of file
```
8、设置root密码

`passwd`

9、安装启动引导器grub:
安装 grub 包，并执行 grub-install 已安装到 MBR:  
须根据实际分区自行调整 /dev/sda, 切勿在块设备后附加数字，比如/dev/sda1 就不对
```
pacman -S grub
grub-install --target=i386-pc --recheck /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

10、卸载分区并重启系统  
离开 chroot 环境：  
`exit`  
#重启计算机：  
`reboot`  
好了，一个最基本的字符系统建好了，接下来可以选择安装桌面等图形环境了。

## 三、安装图形界面
先进行网络设置，在上面的livecd中一般会自动联网
`# ip link`
找到网络设备，我的是enp0s3
```
# ip link set enp0s3 up
# dhcpcd enp0s3
# systemctl enable dhcpcd@enp0s3.service
```
以后系统就会自动联网了
对于无线还未尝试，可以看archwiki。
进入系统后首先更新软件包
```# pacman -Syu```
然后安装x window:
``` 
pacman -S xorg
pacman -S xterm
pacman -S xorg-xinit
```
默认安装就行

* 安装显卡驱动
```
pacman -S xf86-video-vesa     # 通用显卡驱动，不提供任何2D和3D加速功能
pacman -S xf86-video-intel    # Intel
pacman -S xf86-video-nouveau  # Nvidia
pacman -S nouveau-dri
pacman -S xf86-video-ati      # Ati
pacman -S xf86-video-vesa     # 虚拟机
```
安装声卡驱动键入

`# pacman -S alsa-utils`

安装XFCE4 桌面套件
xfce4是一个小巧但功能完备的桌面环境，比较适合配置较差的电脑。要安装xfce4
`# pacman -S xfce4`
如果还需要一些基础的工具的话，还可以安装xfce4-goodies，也就是xfce4自带的一些工具，
`pacman -S xfce4-goodies`
安装显示管理器sddm

显示管理器也就是Linux启动之后的启动界面。世界上有很多个显示管理器，这里使用sddm，一个不错的显示管理器。要安装它，输入以下命令进行安装。

`pacman -S sddm`
安装完成后开机启动显示管理器

`systemctl enable sddm.service`

安装lightDM和图形化管理工具
`pacman -S lightdm lightdm-gtk-greeter lightdm-gtk-greeter-se`
配置显示管理器sddm

安装完桌面之后，就可以配置显示管理器了。首先需要生成一个配置文件，

sddm --example-config >/etc/sddm.conf

配置/etc/sddm.conf文件以实现自动登录到plasma桌面（如果使用其他桌面的话对应修改）。
```
[Autologin]
User=john
Session=plasma.desktop
```
这会以john用户自动登录到系统。
注意：自动登录有风险，可能会让不合法的人登录到你的系统
```
编辑/etc/sddm.conf文件
#使用tty7开启图形会话
[XDisplay]
MinimumVT=7
#开机时自动打开数字锁定键
[General]
Numlock=on
```
安装sudo,让普通用户无需切换执行一些root用户指令:
`# pacman -S sudo`


安装中文字体
文泉驿正黑和微米黑
`pacman -S wqy-zenhei wqy-microhei`

添加一个普通用户,比如kimolate
`# useradd -m -s /bin/bash kimolate`
添加完毕为普通用户设定一个密码
`# passwd kimolate`
为刚才添加的普通用户添加sudo的相关权限
`# visudo`
找到'root ALL=(ALL) ALL'一行,添加
'kimolate ALL=(ALL) ALL'

保存重启）

为了避免出现没有~/.xinitrc的情况，所以可以从系统中复制一个：
`# cp /etc/skel/.xinitrc ~`
（或者直接新建一个 #touch ~/.xinitrc）
然后打开.xinitrc
```
# cd ~
# sudo nano .xinitrc
```
找到
```
#exec gnome-session
#exec startkde
#exec startlxde
#exec startxfce4
…......
```
添加 exec startxfce4或直接去掉你对应桌面的语句前面的#
保存退出

添加执行权限
`#sudo chmod +x ~/.xinitrc`
最后设置自动启动slim登陆器
`# sudo systemctl enable slim.service`
现在一个基本的图形界面建好了。
登陆系统后，打开终端：
`# nano /etc/locale.conf`
修改LANG变量en_US.UTF-8为zh_CN.UTF-8,重启后就能显示中文了。
```
# export  LANG=zh_CN.UTF-8
# export  LC_ALL="zh_CN.UTF-8"
```
改.xprofile的，归根结底就是在启动xfce之前把环境变量LANG改成zh_CN.utf-8

接下来安装fcitx输入法
`# sudo pacman -S fcitx-im fcitx-configtool`
（如果你采用 KDM、GDM、LightDM 等显示管理器，请在~/.xprofile (没有则新建一个)中加入如下3行）如果你采用 startx 或者 Slim 启动 （即使用.xinitrc的场合），则在 ~/.xinitrc 中加入：
 ```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
 ```
重新登录后让环境变量生效。
如果你使用 XDG 兼容的桌面环境如 KDE, GNOME, XFCE, LXDE, 当你重新登录后，Fcitx 应该会自动启动，如果没有的话，可以打开控制台并运行：
`# fcitx`
现在终于大功告成了，重启后你应该会看到这样的画面：

网络连接

使用 network-manager 并设置开机启动
```
#pacman -S networkmanager
#systemctl enable NetworkManager.service
```
install vim vundle 
新建bundle没有写权限，插件不能安装

```
cd ~/.vim/
chmod 777 bundle
```

Linux命令行模式切换控制台，由图形转换到控制台模式：ctrl+alt+f1(同时按下3秒钟不要马上松开)。由控制台转向图形模式是：alt+f7


下面这个命令会从官方镜像列表中获取20个中国最近同步过的源，并对这20个源进行大文件下载来，根据在你电脑里的下载速度进行排序，写入mirrorlist（强烈推荐）
先安装Reflector
`reflector --verbose --country 'china'-l 20 -p http --sort rate --save /etc/pacman.d/mirrorlist`
pacman -Syy

## 无线连接

### 大致过程
arch安装重启之后，是没有网络连接（大坑）！插上网线，dhcpcd总是连不上
跟着<https://www.cnblogs.com/vachester/p/5637027.html>这份教程，
因为共用home目录的原因，使用其他版本的linux下载相应如工具到本地
`#iw dev wlan0 connect HIT-WLAN `  
使用这个命令可以访问无密码的手机热点。
下载networkmanager就可以连接wifi
`nmcli dev wifi connect <name> password <password>`
现在的解决办法是，netctl连接静态ip
直接从`/etc/netctl/`的examples复制静态ip的配置文件
```
Description='A simple WPA encrypted wireless connection using a static IP'
Interface=wlp2s0
Connection=wireless
Security=wpa
ESSID='901'
Key='qwert12345'
IP=static
Address='192.168.1.133/24'
Gateway='192.168.1.1'
DNS=('192.168.1.1')
# Uncomment this if your ssid is hidden
#Hidden=yes
```  
`netctl start wlp2s0-901  # 建立连接`  
`netctl enable profwlp2s0-901ile  # 开机时自动启动配置文件`  
`netctl reenable profile  # 创建开机服务`  

感觉是dchpcd的问题，重新安装驱动，估计不用也可以

ssh server
`pacman -S openssh`

## 安装ibus-rime输入法
* 安装  
> `pacman -S ibus-rime ibus-qt`  
* 配置默认输入法  
> `qtconfig-qt4`  
  `Interface -> Default Input Method -> ibus`
* 手工启动输入法  
> `ibus-setup`  
* 开机启动（应该是开X启动）
> 
``` 
vim ~/.xprofile`
# 输入一下代码
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus
ibus-daemon -x -d
# 如果存在~/.xinitrc，则需修改
vim ~/.xinitrc
# 添加如下代码
# 执行.xinitrc之前，先执行.xprofile
[ -f /etc/xprofile ] && source /etc/xprofile
[ -f ~/.xprofile ] && source ~/.xprofile
```  
* 设置
> 
```
cd /usr/share/rime-data
# 选择需要的输入法
# github 克隆配置文件（个人）
git clone git@github.com:ksino/rime.git
# 将需要的文件复制到一下目录。
# 一般是下面三个文件
# default.custom.yaml
# double_pinyin.custom.yaml
# weasel.custom.yaml
cd ~/.config/ibus/rime
```
