---
title: Fuchsia 编译指南
date: 2018-11-05 15:41:56
categories:
 - fuchsia
tags:
 - fuchsia
 - tutorial
---

##### AUTHOR: [Locez](http://locez.com)
##### VERSION: 1

### Fuchsia 简介
---

在进入本文正文之前，有必要简单介绍一下 Fuchsia 是个什么东西

![](https://image.locez.com/blog/fuchsia-compile-tutorial-00.jpg)

Fuchsia 是 Google 在 2016 年 8 月推出的一个操作系统，是一个实时操作系统。它不再基于 Linux 内核，而是基于名为 `Zircon` 的微内核，并且它被设计为支持多种设备，手机、平板、PC，因此可以猜测其准备在三种终端设备上进行一定的统一

在官方文档中有这么一句话：

``` bash
Pink + Purple = Fuchsia(a new Operating System)
```
可以作为该系统的 slogan

在本文，将介绍从源码编译该系统进行探索体验

<!--more-->
### 环境准备
---

    - go ( >=1.6 )
    - git
    - python (2.7)
    - qemu

在 Fuchsia 官方提供的各种脚本中使用的 python 2 的代码，因此此处请确保你的 python 解释器是 2.7 的

go 则是因为获取源码的工具用到了 go 环境


### 获取源码
---

获取源码的方式非常简单，执行以下命令即可，但是要注意源码有 15 G 左右，下载下来还是非常需要时间的

``` bash
$ mkdir fuchsia
$ cd fuchsia
$ curl -s "https://fuchsia.googlesource.com/scripts/+/master/bootstrap?format=TEXT" | base64 --decode 
```


### 编译 Fuchsia
---

注意，编译的过程非常简单，但是可能需要安装一些额外的依赖，在本人的 gentoo 系统中，编译环境本身是齐全的，如果你按照以下的命令编译不成功，可根据 `Error` 安装相应的依赖包，下面是一条官方给出的 `debian` 系统的依赖安装命令：

``` bash
sudo apt-get install build-essential curl git python unzip
```

Fuchsia 目前支持 `x64` 与 `arm64` 两种架构，此处我们编译 `x64` 版本，从时间上来说编译时间比预期要短的多，本人只编译了 1h 左右，编译的文件有两万六千多个

``` bash
$ cd fuchsia
$ ./scripts/fx set x64
$ ./scripts/fx full-build
```


### 使用 QEMU 启动 Fuchsia
---

Fuchsia 的图形界面需要 `vulkan` 进行硬件加速才可以使用，所以使用 QEMU 进行模拟启动的时候只会有简单的界面而已，使用如下命令启动 tty 界面，其中 `-m` 用于指定使用的内存，单位（MB），本人内存多就随便设置了

``` bash
./scripts/fx run  -m 4096
```

![](https://image.locez.com/blog/fuchsia-compile-tutorial-02.jpg)

另外可用以下命令启动简易版的图形界面

``` bash
/scripts/fx run  -m 4096 -g
```

![](https://image.locez.com/blog/fuchsia-compile-tutorial-03.jpg)

在这个简易的图形界面中，我们可以看到有 `4` 个 `tab` 可用`Alt+Tab` 按键进行切换，第一个是 `debug`，其余 `3` 个是普通终端，可在上面输入命令

![](https://image.locez.com/blog/fuchsia-compile-tutorial-04.jpg)

系统目前含有`198` 个命令，已经实现了大部分我们常用的系统基础命令，甚至还包含 vim 噢

### 简单探索
---

从上面的图中，我们也可以看到 Fuchsia 的根目录包含以下几个目录（由于该系统还没 `tree` 命令，因此我无法给出目录树）

``` bash
blob
boot
config
data
dev
hub
install
pkgfs
svc
system
tmp
volume

```
 - `blob` 文件夹中其实是一些二进制数据，具体该文件夹的作用尚不明确
 - `boot` 顾名思义，启动文件夹，其下有 `bin` `config` `driver` `kernel` `lib` 等子文件夹，有关系统启动的文件放在此处
 - `config` 文件夹下是 `ssl` 证书目录，推测相关配置文件放在此处
 - `data` 尚不清楚具体用途，推测是存放应用数据
 - `dev` 设备文件夹
 - `svc` 文件夹包含一系列 `fuchsia.*` 开头的文件
 - `system` 目录与 `boot` 目录的结构类似，没有 `kernel` `config`
 - `volume` `install` 等目录都是空目录，无法推测其用途

另外值得一提的是虽然 `system` 文件夹中目前不含有 `apps` 文件夹，但是 `PATH` 变量是包含该目录的，因此可以推测安装的应用应该会放到 `/system/apps` 下

PS：最后关机输入 `dm poweroff` 即可


### 总结
---

本文其实整体编译流程与官方文档差不多，如果你的英文阅读能力还可以的话，可以去阅读英文文档，应该会比笔者讲的细节要对，其次就是对 Fuchsia 进行了简单的探索，说实话也不算是探索，因为对该系统最好奇的还是 UI，但是由于没有对应的硬件，无法进行测试体验，这也算是在意料之外的。
