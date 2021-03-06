[TOC]

# Linux安装

## 分区方案

* 考虑多个系统，

|SWAP|home|扩展分区|
|-|-|-|

|扩展分区...|Debian/|CentOS/|Arch/ ...|
|-|-|-|-|

# Linux通用

* 传送文件  
`scp C:\Users\Administrator\Desktop\Temp\meizi\all_file kim@192.168.1.132:/home/kim/project/meizi/all_file`

## 查看内存
* `free`
* `top`
* `htop`更直观

# 效率类工具

## 安装ZSH

```
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
# 默认启动ZSH
chsh -s /bin/zsh
```
* 通配符的Bug

`find . -name *.h`
`no matches found`

* 解决

```
# vim ~/.zshrc 
# add
setopt no_nomatch
source ~/.zshrc
```

## tmux

## vim

# Linux Deploy on Andriod

> 在安卓手机上安装Linux系统,但是安装的系统需支持ARM架构.
> 安装的centos-altarch,yum源的支持不是太好.

## 安装步骤
1. 配置文件
     * 源地址
       选择速度快一点，如中科大或者清华  
       `https://mirror.tuna.tsinghua.edu.cn/archlinuxarm/`  
     * 安装路径  
       `/mnt/sdcard/linux/arch/arch.img/  # linux的文件夹就是再手机内存根目录下`


2. 安装
3. 启动
4. 文件传输
`scp D:/Catalog/Django/maxone.zip kim@192.168.1.132:/home/kim/project`




# Debian 9

## [安装显卡驱动](https://wiki.debian.org/AtiHowTo#AMD.2FATI_Open_Source_Drivers_.28amdgpu.2C_radeon.2C_r128.2C_mach64.29)
新安装的系统，并没有安装驱动，参考以上网址

```
# Debian 8 "Jessie"
deb http://httpredir.debian.org/debian/ jessie main contrib non-free

apt-get update

apt-get install firmware-linux-nonfree libgl1-mesa-dri xserver-xorg-video-ati
```

## 直接命令行启动系统

* 参考<https://www.cnblogs.com/qingkai/p/5443572.html>

* 修改grub项

`sudo nano /etc/default/grub`

* 修改其中三项

```
...
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX=""
...
#GRUB_TERMINAL=console
```

为

```
...
#GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX="text"
...
GRUB_TERMINAL=console
```

* 更新grub
`sudo update-grub`

* 设置systemd
`sudo systemctl set-default multi-user.target`

* 如果要切回图形界面, 反转以上步骤，执行一下命令

`systemctl set-default graphical.target`

* reboot  
* 如果在命令行要进入x window，直接`startx`
> 但建议`sudo systemctl start lightdm.servic`

## 音量调节

`sudo alasmixter`

* 设置快捷键
```
升高音量：  
amixer set Master 5%+  
降低音量：  
amixer set Master 5%-  
静音：  
amixer set Master toggle
```
## 软件源

* <https://www.debian.org/mirror/list>

```
ftp2.cn.debian.org ftp.cn.debian.org mirror.lzu.edu.cn mirrors.163.com  mirrors.tuna.tsinghua.edu.cn mirrors.ustc.edu.cn mirrors.xjtu.edu.cn 
```
* 使用netselect(需要安装)测试速度
> 可以使用netselect-apt的命令，直接生成速度最快软件源的sources.list

## rime输入法
`sudo apt install ibus-rime`
`sudo apt install librime-data-double-pinyin`

## 字体

* 
`wget https://github.com/downloads/adobe-fonts/source-code-pro/SourceCodePro_FontsOnly-1.013.zip`

* 解压
`unzip SourceCodePro_FontsOnly-1.013.zip`

* 复制
`cp -r ~/Downloads/SourceCodePro_FontsOnly-1.013/OTF/. ~/.fonts`

* 执行
`fc-cache -f -v`

* 在终端中使用Source Code Pro 字体

## 查看已安装的软件

`dpkg -l | grep soft-name`

## ssh server
```
sudo  apt-get install openssh-server
#ssh服务状态
ps -s|grep ssh
#启动服务
service ssh start
```
## 刻录iso到U盘
`dd bs=4M if=/home/debian/archlinux.iso of=/dev/sdb status=progress && sync`

## 更改中文为英语
`sudo vi /etc/default/locale`  
```
LANG="en_US.UTF-8"
LANGUAGE="en_US:en"
```
`reboot`
