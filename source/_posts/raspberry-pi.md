title: Raspberry Pi 介绍与食用方法
date: 2015-10-11 20:29:16
categories:
 - Linux
tags:
 - linux
 - raspberry pi
---

##### AUTHOR: [Locez](http://locez.com)
##### VERSION: 2
##### UPDATE： 2015-10-14
![](http://7narru.com1.z0.glb.clouddn.com/raspberrypiraspberry-pi-logo.jpg)
Raspberry Pi 是 Raspberry Pi Foundation 推出的迷你电脑，它只有信用卡大小，但可以完成一台普通 PC 能完成的大部分工作，并且价格很便宜，是电脑爱好者的不二选择，如果你是一名 Linuxer 更应该拥有一台这样的迷你电脑。

<!--more-->
* [发展](#develop)
* [Raspberry Pi 的用途](#usage)
* [操作系统的选择](#op)
* [个人用途](#mine)
    * [安装系统](#install)
    * [配置SSH](#sshconfig)
    * [配置源与更新系统](#configmirrors)
    * [桌面化 Raspberry Pi](#desktop)
    * [使用 vncviewer 访问 Raspberry Pi](#vncviewer)
    * [成果展示](#result)
* [参考资料](#reference)

<h2 id="develop">发展</h2>
***
Raspberry Pi 自 2012 年发布以来，依次发布了 Raspberry Pi 1 A , Raspberry Pi 1 B ,Raspberry Pi 1 B+ ,Raspberry Pi 1 A+ ,Raspberry 2 B 五个版本，这些版本硬件上有不少变化，具体可以查阅 Wikipedia [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi) ，另外 Raspberry Pi 2 B 将支持 Windows 10 iot ，这对非 Linux 用户来说也是一个福音，因为你可以完全把 Raspberry Pi 2 B 当成你的另一台 Windows PC ，详情可以查看 [Raspberry Pi Windows 10 iot](http://ms-iot.github.io/content/en-US/Downloads.htm)。

<h2 id="usage">Raspberry Pi 的用途</h2>
***
Raspberry Pi 到底能拿来做什么呢？它的玩法多的数不清了，因为这取决于我们的创意，作为一块开发板，它给我们提供了很大的自由。

 - [树莓派构建的超级计算机](https://linux.cn/article-1402-1-rel.html)
 - [作为家用服务器](https://linux.cn/article-2228-1.html)
 - [我拿树莓派来做什么](https://linux.cn/article-1618-1-rel.html)
 - [Raspberry Pi 五种有趣玩法](https://linux.cn/article-681-1-rel.html)
 - 微型个人电脑 （笔者也就只是当作个人微型电脑而已）

下面是几张引用文章内的图：
![](https://dn-linuxcn.qbox.me/data/attachment/album/201305/31/100902rmu4rm3uemxruw55.jpg)
![](https://dn-linuxcn.qbox.me/data/attachment/album/201305/31/100904mgrbvrgafragqcfj.jpg)
![](https://dn-linuxcn.qbox.me/data/attachment/album/201311/07/230232hzveaaihvovvbbl8.jpg)



 
<h2 id="op">操作系统的选择</h2>
***
由于 Raspberry Pi 几乎是为 Linux 而生的，所以 Raspberry Pi 的操作系统也是多样的，为此以下介绍几个操作系统。

 - [Raspbian](https://www.raspberrypi.org/downloads/raspbian/) 基于 debian 的 Raspberry Pi 官方操作系统，如果是当开发板使用，最建议使用此系统
 - [Ubuntu Mate](https://ubuntu-mate.org/raspberry-pi/) 
 - [Snappy Ubuntu Core](https://developer.ubuntu.com/en/snappy/start/#snappy-raspi2)
 - [Windows 10 Iot Core](http://ms-iot.github.io/content/en-US/Downloads.htm)
 - [OPENELEC](http://openelec.tv/get-openelec)
 - [PINET](http://pinet.org.uk/)
 - [RISC OS](https://www.riscosopen.org/content/downloads/raspberry-pi)
以上操作系统都可以在 [Raspberry Pi 主页](https://www.raspberrypi.org/downloads/) 找到相关信息。
 - [Arch Arm](http://archlinuxarm.org/)
 - [Gentoo Arm](https://www.gentoo.org/downloads/)
 

 <h2 id="mine">个人用途</h2>
 ***
 笔者只是将 Raspberry Pi 当成个人电脑使用而已，因此，未选择 Raspbian 而是选择了 Arch Arm。
 下面是我选择的配件
 - Raspberry Pi 2 B
 - 8 G 闪迪内存卡
 - USB Wi-Fi 模块（可选）
 - 亚克力外壳（可选）
 - 散热铝片或铜片
 - 电源线
 - 键盘鼠标（可选）

<h3 id="install">安装系统</h3>
***
首先参照 [Arch Arm Installation](http://archlinuxarm.org/platforms/armv7/broadcom/raspberry-pi-2) 安装系统。
``` bash
    用你 Linux 上的 sd 卡设备代替 sdX 
    为 SD 卡分区：
    
    fdisk /dev/sdX
    
    第一步请先删除原来的分区并重新创建：
    输入 o 清除所有分区。
    输入 p 列出所有分区，此时应该没有分区。
    输入 n ， 然后输入 p 选择主分区，1 是第一个分区 ，输入 ENTER 确定第一个扇区，然后输入 +100M 。
    输入 t ， 然后输入 c 设置第一个分区类型为 W95 FAT32 (LBA)。
    输入 n ， 然后输入 p 选择主分区, 2 是第二个分区, 直接输入 ENTER 确定默认的扇区和最后的扇区（剩下的所有容量作为第二个分区）
    输入 w 写入分区表并退出。
    创建和挂载 vfat 文件系统:
    
    mkfs.vfat /dev/sdX1
    mkdir boot
    mount /dev/sdX1 boot
    
    创建个挂载 ext4 文件系统:
    
    mkfs.ext4 /dev/sdX2
    mkdir root
    mount /dev/sdX2 root
    
    使用 root 用户下载和解压 根文件系统:
    
    wget http://archlinuxarm.org/os/ArchLinuxARM-rpi-2-latest.tar.gz
    bsdtar -xpf ArchLinuxARM-rpi-2-latest.tar.gz -C root
    sync
    
    移动启动文件到第一分区:
    
    mv root/boot/* boot
    
    卸载挂载点:
    
    umount boot root
    
    将 SD 卡插入 Raspberry Pi ，连接以太网和 5v 电源。
    使用 SSH 登录
        默认用户是 alarm 密码 alarm。（ssh 请先用此用户登录，再修改 ssh 配置允许 root 登录）
        root 的默认密码是 root。
```


<h3 id="sshconfig">配置SSH</h3>
***
**注意** ：以下用到 `sudo` 命令的，若你未配置 `sudo` 请直接用 `root` 用户执行。
首次使用应该按如下格式 `ssh` 登录
``` bash
 $ ssh -p 22 user@address
```
进去后使用 `su` 切换到 `root` ，并修改密码
``` bash
 # password
```
为了能使 `root` 通过 `ssh` 登录，编辑 `/etc/ssh/sshd_config`
``` bash
 $ nano /etc/ssh/sshd_config
```
将 `#PermitRootLogin` 这行去掉注释，并将值设置为 `yes` ，`PermitRootLogin yes`。

<h3 id="configmirrors">配置源与更新系统</h3>
***
编辑 `/etc/pacman.d/mirrorlist`

``` bash
 $ sudo nano /etc/pacman.d/mirrorlist
```
在顶部增加以下代码，这是中科大的源

    ## USTC
    Server = http://mirrors.ustc.edu.cn/archlinuxarm/armv7h/$repo

编辑好后按 `ctrl +x` ，然后按 `y` 保存，然后升级整个系统
``` bash
 $ sudo pacman -Syu
```

<h3 id="desktop">桌面化 Raspberry Pi</h3>
***
首先安装 `xorg` 

``` bash
 $ sudo pacman -S xorg
 $ sudo pacman -S xorg-xinit
```
然后安装 `lxqt` 桌面
``` bash
 $ sudo pacman -S lxqt
```
<h3 id="vncviewer">使用 vncviewer 访问 Raspberry Pi</h3>
***
配置 `vncviewer` 
本机与 Raspberry Pi 都需要安装 `tigervnc`
``` bash
 $ sudo pacman -S tigervnc
```
在 Raspberry Pi 中执行 `vncserver`

``` bash
 $ vncserver 

 You will require a password to access your desktops.

 Password:
 Verify:
 Would you like to enter a view-only password (y/n)? n

 New 'ArchRaspi:1 (locez)' desktop is ArchRaspi:1

 Creating default startup script /home/locez/.vnc/xstartup
 Starting applications specified in /home/locez/.vnc/xstartup
 Log file is /home/locez/.vnc/ArchRaspi:1.log
```

然后编辑 `~/.vnc/xstartup` ,将原来的内容替换为以下内容，你也可以直接删除原文件，再新建一个同名文件
``` bash

#!/bin/sh

unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
exec startlxqt
```

然后杀掉 vnc 服务，并重启它

``` bash
 $ vncserver -kill :1
 Killing Xvnc process ID 400
 
 $ vncserver 

 New 'ArchRaspi:1 (locez)' desktop is ArchRaspi:1

 Starting applications specified in /home/locez/.vnc/xstartup
 Log file is /home/locez/.vnc/ArchRaspi:1.log

```

记住上面的 `:1` 可能根据实际情况不同，这个端口是你用 `vncviewer` 连接时用的端口
然后本机中执行

``` bash
 $ vncviewer address:port
```

然后输入密码就可以了

![small](http://7narru.com1.z0.glb.clouddn.com/raspberrypismall.png)

现在我们看见这个不是全屏的，但是可以在启动 vncserver 的时候增加参数，来指定分辨率。

``` bash
 $ vncserver -kill :1
 $ vcnserver -geometry 1920x1000
```

这下就全屏了
![big](http://7narru.com1.z0.glb.clouddn.com/raspberrypiSelection_013.png)

然后你可以在不另外配显示屏的情况下正常使用 Raspberry Pi ，将它配置成一个 `samba` 服务器，或者做成一个下载器，这仅仅取决于，你想将 Raspberry Pi 拿来干什么。

笔者还尝试将 Raspberry Pi 直接连入手机 Wi-Fi ，把手机当成路由器使用，获取手机内网 IP 后，直接在手机上 `ssh` 内网登录，从而实现 Raspberry Pi 使用手机流量上网，并且不用借助路由器就可以连接 Raspberry Pi。
这里笔者推荐的是 **juiceSSH** ，手机上简单实用的一款 ssh 工具。由于篇幅原因，此处不再详述如何使用手机直连 Raspberry Pi，动手能力强的同学可以参看上面我给的思路，自行折腾。

后来笔者，买了键盘以后，又利用 `tmux` 将手机纯粹当屏幕使用。
首先 手机先 `ssh` 登陆 Raspberry Pi，执行

    tmux

然后在看不见屏幕的情况下，使用键盘输入 **用户名** 、 **密码** 进行登录，然后执行

    tmux attach
    
此时键盘与手机屏幕的输入已经是同步的，一台个人作品就此完成！
<h3 id="result">成果展示</h3>
***
下面几张图，是笔者在学校折腾的时候拍的
![](http://7narru.com1.z0.glb.clouddn.com/raspberrypiraspberry1.jpg)
![](http://7narru.com1.z0.glb.clouddn.com/raspberrypiraspberry2.jpg)
![](http://7narru.com1.z0.glb.clouddn.com/raspberrypiraspberry3.jpg)

<h2 id="reference">参考资料</h2>
***
 - [Linux.中国](https://linux.cn)
 - [Wikipedia Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi)
 - [RaspberryPi](https://www.raspberrypi.org)
 
