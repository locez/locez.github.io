title: 新手指南： 手把手教你安装 Ubuntu 和 Fedora
date: 2015-07-26 19:00:37
categories:
  - Linux
tags:
  - ubuntu
  - fedora
  - linux
---
##### AUTHOR: [Locez](http://locez.com)
##### VERSION: 1


Linux 由于开源，所以具备可定制性，因此衍生了许多发行版．Ubuntu 和 Fedora 算是其中对新手比较友好的两个发行版，主要是其安装较为简单，用户群多，有问题搜索出相关的信息或者找前辈解决。此文为 Linux 新手准备，通过展示整个安装过程来使 Linxu 新手完成安装 Ubuntu 或 Fedora ，也恳请各位前辈指出不足之处。

### 阅读建议
***
  - 本文将包含 Ubuntu 和 Fedora 两个发行版的安装，请先通篇浏览全文，再决定安装哪个发行版，并且配图有相应的文字说明，请不要忽视。
  - 如果你是一位新手，强烈建议使用虚拟机操作，如果你相信自己可以解决问题，也可使用 **UltraISO** 、**USBWriter** 和 `dd`命令写入 U 盘，进行实体机安装，此处不详述。
  - 有任何问题都可以加入 **Linux中国新手村** QQ 群提问，QQ 群号 `198889109` ，验证 `LINUXCN` 。


### Ubuntu 简介
***
Ubuntu 是一个基于 Debian 的 GNU/Linux 操作系统，支持 X86 、64以及 PPC 架构。Ubuntu 每隔六个月发布一个版本，即每年的四月和十月，本文使用的是 **15.04 64-bit** 版本，Ubuntu 对于新手应该是比较友好的一个 Linux 发行版，中文本地化也做的不错，有开箱即用的感觉。因为 Ubuntu 近几年用户群的增加，多了很多对于新手有用的资料，因此请不用担心遇到问题无法解决，善用搜索和提问，将使你更快速地成长。

<!--more-->
### Fedora 简介
***
Fedora 是一个由 Fedora 社区开发的 Linux 发行版，由 Red Hat 公司赞助，可以将 Fedora 看成是 Red Hat Linux 个人使用的代替，由于有 Red Hat 公司的支持，Fedora 的功能非常完善，还分为 WORKSTATION 、SERVER 和 CLOUD 版本。本文使用的是 **Fedora 22 WORKSTATION （工作站）**，Fedora 22 已经将包管理器从 `YUM` 改为 `DNF` ，因此建议学习者直接学习 `DNF` 。

### 本文环境
***
  - Windows 8.1 64-bit
  - VirtualBox-5.0 [点此下载](http://download.virtualbox.org/virtualbox/5.0.0/VirtualBox-5.0.0-101573-Win.exe) 
  - Ubuntu 15.04 64-bit [点此下载](http://releases.ubuntu.com/15.04/ubuntu-15.04-desktop-amd64.iso)
  - Ubuntu 15.04 32-bit 适合配置较低的用户使用 [点此下载](http://releases.ubuntu.com/15.04/ubuntu-15.04-desktop-i386.iso) 
  - Fedora 22 64-bit [点此下载](https://download.fedoraproject.org/pub/fedora/linux/releases/22/Workstation/x86_64/iso/Fedora-Live-Workstation-x86_64-22-3.iso)
  - Fedora 22 32-bit 适合配置较低用户使用 [点此下载](https://download.fedoraproject.org/pub/fedora/linux/releases/22/Workstation/i386/iso/Fedora-Live-Workstation-i686-22-3.iso) 
### Ubuntu 安装
***
##### 1.新建与加载盘片
当你安装完 Vbox 后，打开你应该会看到下面这样的界面
![VBOX](http://7narru.com1.z0.glb.clouddn.com/image/ubuntu/vbox.png)
点击新建后会出来如下图所示的界面，一般如图填写即可，内存可酌情填写
![NEWUBUNTU](http://7narru.com1.z0.glb.clouddn.com/image/ubuntu/newubuntu.png)
下一步将创建虚拟硬盘，如图所示，默认位置为 C 盘，如果你不想在 C 盘创建，请确保你选择的盘格式为NTFS
![NEWUBUNTU](http://7narru.com1.z0.glb.clouddn.com/image/ubuntu/newubuntu1.png)
创建完成后，请点 **设置** 如图加载 ISO 文件
![LOADUBUNTUISO](http://7narru.com1.z0.glb.clouddn.com/image/ubuntu/loadubuntuiso.png)
#### 2.安装 Ubuntu
点击 **启动** ，会开机，进入如下界面
![INSTALLING](http://7narru.com1.z0.glb.clouddn.com/image/ubuntu/install1.png)
![INSTALLING2](http://7narru.com1.z0.glb.clouddn.com/image/ubuntu/install2.png)
这里请注意，如果你与笔者一样使用虚拟机，强烈建议选择 **清除整个磁盘并安装 Ubuntu** ，但如果你要装到实体机与 Windows 形成双系统时，请选择 **其他选项** ，但这要求你对 Linxu 有一定的了解且具备一定的基础进行分区操作，注意不要覆盖 Windows 的 C 盘，此处由于篇幅原因，不再详述。
![INSTALLING3](http://7narru.com1.z0.glb.clouddn.com/image/ubuntu/install3.png)
如图，进行用户设定，**计算机名** 是主机名，**用户名** 是登录时用的账户名称，**密码** 则是你所设 **用户名** 的登录密码，请务必记牢。
![INSTALLING4](http://7narru.com1.z0.glb.clouddn.com/image/ubuntu/install4.png)
这一步之后会选择时区，直接点下一步即可，键盘选择如下图
![SELECTKEYBOARD](http://7narru.com1.z0.glb.clouddn.com/image/ubuntu/keyboard.png)
配置选择已完成，接下来请耐心等待安装过程，如图，请不要点击 **SKIP** 
![INSTALLING](http://7narru.com1.z0.glb.clouddn.com/image/ubuntu/install5.png)
耐心等待安装完成，然后会重启进入系统，用你上面配置的用户名和密码登录，请注意最好不要登录 **root** ，你可以用 `sudo` 命令来获取相应的权限，下图是展示成果
![FINISH](http://7narru.com1.z0.glb.clouddn.com/image/ubuntu/finish.png)

### Fedora 安装
***
##### 1.新建与加载盘片请参考上面的 Ubuntu 部分
##### 2.安装 Fedora
点击 **启动**，会开机，进入如下界面,如图操作
![STARTLIVECD](http://7narru.com1.z0.glb.clouddn.com/image/fedora/startlivecd.png)
接下来依然是如图操作
![INSTALLING](http://7narru.com1.z0.glb.clouddn.com/image/fedora/install.png)
然后是选择语言，选择完后进入如图界面
![INSTALLING1](http://7narru.com1.z0.glb.clouddn.com/image/fedora/install1.png)
配置安装位置，这里请注意，如果你与笔者一样使用虚拟机，强烈建议选择 **自动配置分区** ，但如果你要装到实体机与 Windows 形成双系统时，请选择 **我要配置分区** ，但这要求你对 Linxu 有一定的了解且具备一定的基础进行分区操作，注意不要覆盖 Windows 的 C 盘，此处由于篇幅原因，不再详述。
![INSTALLING2](http://7narru.com1.z0.glb.clouddn.com/image/fedora/install2.png)
下一步将创建 **root** 和 **日常使用账户** ，**root** 账户有最大的管理权限，你甚至可以将整个系统删除，所以使用 **root** 账户请务必小心，**日常使用账户** 应作为你的习惯使用账户，必要时只需使用 `sudo` 命令暂时提升权限即可，更多说明如图所示
![INSTALLING3](http://7narru.com1.z0.glb.clouddn.com/image/fedora/install3.png)
**root**  配置只需创建密码即可，下图是 **日常使用账户** 配置
![INSTALLING4](http://7narru.com1.z0.glb.clouddn.com/image/fedora/install4.png)
配置完后将回到之前的界面，请耐心等待安装，如图
![INSTALLING5](http://7narru.com1.z0.glb.clouddn.com/image/fedora/install5.png)
安装完成，点击 **退出** 后，进入的依然是 Live CD 环境，请先关机，再执行下一步
![INSTALLING6](http://7narru.com1.z0.glb.clouddn.com/image/fedora/install6.png)
由于 Fedora 未自动卸载盘片，因此需要手动卸载盘片，否则将再次进入 Live CD 环境，请如图操作
![INSTALLING7](http://7narru.com1.z0.glb.clouddn.com/image/fedora/install7.png)
接下来则是点击 **启动** 进入你的 Fedora ，使用你上面设置的用户名和密码登录，请注意最好不要登录 **root** ，你可以用 `sudo` 命令来获取相应的权限，下图是展示成果
![INSTALLING8](http://7narru.com1.z0.glb.clouddn.com/image/fedora/install8.png)


### 参考资料
***
  - [Ubuntu-Wikipedia](https://zh.wikipedia.org/wiki/Ubuntu)
  - [Fedora-Wikipedia](https://zh.wikipedia.org/wiki/Fedora)
