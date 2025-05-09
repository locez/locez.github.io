---
title: 在 RK3588 上安装 Gentoo
date: 2025-05-09 23:27:39
categories:
   - linux
tags:
   - rk3588
   - gentoo
   - uboot
---

##### AUTHOR: [Locez](https://locez.com)
##### VERSION: 1

###
---

买了一块 rk3588 的 nanopi M6 用来做家用服务器，由于 Arch Linux 的 arm 版维护不够活跃，最终还是选择使用 Gentoo 操作系统。

本文主要记载整个折腾过程

### 官方镜像
---

下载官方 sd 卡镜像，我下载的是 debian 的 image，解压后 `dd` 进 sd 卡

```
gunzip rk3588-sd-debian-bookworm-core-6.1-arm64-20250123.img.gz
sudo dd if=rk3588-sd-debian-bookworm-core-6.1-arm64-20250123.img of=/dev/mmcblk0 bs=1M status=progress conv=fsync
```
### 登录 ssh
---

通过 ssh 登录机器，用户名和密码都是 `pi`

```bash
ssh pi@ip
```

### 分区
---

安装必要的工具

``` bash
apt update
apt install gdisk btrfs-prog dosfstools
```

``` bash
sudo su -
cgdisk /dev/nvme0n1
```

分区情况，我在此处使用 `btrfs` 文件系统

```
Part. #     Size        Partition Type            Partition Name
----------------------------------------------------------------
            1007.0 KiB  free space
   1        512.0 MiB   EFI system partition      EFI System Partition
   2        32.0 GiB    Linux swap                Swap Partition
   3        899.0 GiB   Linux filesystem          Btrfs
```

### 文件系统
---

创建文件系统

``` bash
mkfs.vfat /dev/nvme0n1p1
mkswap /dev/nvme0n1p2
mkfs.btrfs /dev/nvme0n1p3
```

创建基于 `btrfs` 的 `rootfs` 子卷

```
mount /dev/nvme0n1p3 /mnt/
btrfs subvolume create /mnt/rootfs
umount /mnt
```

### 系统安装
---

挂载 rootfs 卷和 boot 分区

``` bash
mkdir -p /mnt/gentoo
mkdir -p /mnt/gentoo/boot
mount -o subvol=/rootfs /dev/nvme0n1p3 /mnt/gentoo/
```

下载并解压 stage3

``` bash
cd /mnt/gentoo
wget https://mirrors.ustc.edu.cn/gentoo/releases/arm64/autobuilds/20250427T235504Z/stage3-arm64-systemd-20250427T235504Z.tar.xz
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```

### 配置交叉编译环境
---

以下命令我会使用 `PC`，`RK` 来做命令行 `prompt` 代表在不同的地方执行语句

``` bash
PC $ sudo emerge --ask sys-devel/crossdev sys-devel/distcc
PC $ sudo emerge --ask app-eselect/eselect-repository
PC $ sudo eselect repository create crossdev
PC $ sudo crossdev --stable --target aarch64-unknown-linux-gnu
```

配置 `distccd` 服务，允许局域网客户端连接

``` bash
PC $ sudo systemctl edit --full distccd.service

[Unit]
Description=Distccd: A Distributed Compilation Server
After=network.target

[Service]
Environment="ALLOWED_SERVERS=192.168.1.0/24"
User=distcc
ExecStart=/usr/bin/distccd --no-detach --daemon --port 3632 --jobs 16 -N 15 --allow $ALLOWED_SERVERS

[Install]
WantedBy=multi-user.target
```

``` bash
PC $ sudo systemctl daemon-reload
PC $ sudo systemctl restart distccd
```
后续可以通过以下命令查看 distccd 的状态

``` bash
PC $ sudo systemctl status distccd
```

接下来配置 RK3588，首先安装 distcc，配置 distcc HOST，这里就是 PC 的 IP 地址了，建议使用静态 IP

``` bash
RK # emerge --ask sys-devel/distcc
RK # distcc-config --set-hosts "192.168.1.3"
```

同时需要更改 `/etc/distcc/hosts` 文件确保 `IP` 一致

```
RK # cat /etc/distcc/hosts
192.168.1.3
```
修改 `/etc/portage/make.conf`

```bash
COMMON_FLAGS="-mcpu=cortex-a76.cortex-a55+crc+crypto -O3 -pipe -fomit-frame-pointer -fPIC"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"
USE="cjk dist-kernel dracut -handbook networkmanager systemd-boot"
CHOST="aarch64-unknown-linux-gnu"
FEATURES="distcc"
MAKEOPTS="-j16"
```

此时已经可以使用 emerge distcc 编译包了

### u-boot 和内核
---

#### 内核
---

在 PC 机上，参照[友善官方文档](https://wiki.friendlyelec.com/wiki/index.php/NanoPi_M6#Build_u-boot_v2017.09)编译 `uboot` 和内核


##### 制作 initramfs

首先到 [alpine linux](https://alpinelinux.org/downloads/) 下载 `minirootfs`

解压后新增 `init` 脚本，用来 switch root

``` bash
#!/bin/sh -e

tty >/dev/null 2>&1 || {
  mkdir -p /proc    && mount -t proc     -o noexec,nosuid,nodev,relatime                          proc     /proc
  mkdir -p /sys     && mount -t sysfs    -o noexec,nosuid,nodev,relatime                          sysfs    /sys
  mkdir -p /dev     && mount -t devtmpfs -o nosuid,size=4096k,nr_inodes=32956400,mode=755,inode64 devtmpfs /dev
  mkdir -p /dev/pts && mount -t devpts   -o nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000    devpts   /dev/pts
  mkdir -p /tmp     && mount -t tmpfs    -o nosuid,nodev,nr_inodes=1048576,inode64                tmpfs    /tmp
  exec setsid sh -c 'exec sh /init </dev/ttyFIQ0 >/dev/ttyFIQ0 2>&1'
}

mkdir -p /newroot
mount -o rw,subvol=rootfs /dev/nvme0n1p3 /newroot


for MOUNT in proc sys dev tmp; do
  mkdir -p "/newroot/${MOUNT}"
  mount --move "/${MOUNT}" "/newroot/${MOUNT}"
done

exec switch_root -c /dev/console /newroot /sbin/init
```

``` bash
chmod +x init
```


``` bash
find . -print0 | cpio --null --create --verbose --format=newc >  /home/locez/rk3588/initramfs.cpio
```

##### 编译内核

``` bash
git clone https://github.com/friendlyarm/kernel-rockchip --depth 1 -b nanopi6-v6.1.y kernel-rk3588

make ARCH=arm64 CROSS_COMPILE=aarch64-unknown-linux-gnu- nanopi6_linux_defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-unknown-linux-gnu- -j16

### 对 .config 进行必要的修改，比如： 根据你的实际情况进行调整
### CONFIG_INITRAMFS_SOURCE="/home/locez/rk3588.cpio
###

ls arch/arm64/boot/Image -alh

### 制作 uImage
mkimage -A arm64 -O linux -T kernel -C gzip -a 0x00400000 -e 0x00400000 -n "Linux Kernel" -d arch/arm64/boot/Image.gz uImage
```

#### uboot
---


```bash
emerge -a dtc
git clone https://github.com/friendlyarm/rkbin --single-branch --depth 1 -b nanopi6
git clone https://github.com/friendlyarm/uboot-rockchip --single-branch --depth 1 -b nanopi6-v2017.09


### 根据需求对 configs/nanopi6_defconfig 进行必要的更改，比如内嵌 dtb
### CONFIG_EMBED_KERNEL_DTB_PATH="../kernel-rk3588/arch/arm64/boot/dts/rockchip/rk3588-nanopi6-rev0a.dtb"
### CONFIG_EMBED_KERNEL_DTB_ALWAYS=y
### CONFIG_EMBED_KERNEL_DTB=y
### CONFIG_NVME=y
### CONFIG_PCI=y
### CONFIG_DM_PCI=y
### CONFIG_DM_PCI_COMPAT=y
### CONFIG_PCIE_DW_ROCKCHIP=y
export ARCH=arm64
export CROSS_COMPILE=aarch64-unknown-linux-gnu-
./make.sh CROSS_COMPILE=$CROSS_COMPILE nanopi6
```

对之前烧录好的 sd 卡进行 uboot 更新

```bash
dd if=uboot.img of=/dev/mmcblk0p1 bs=1M
```

将 `uImage` 放入 `nvme` 的 `/boot` 分区中，然后重启在 `uboot` 输入

```bash
setenv bootcmd "pci enum;nvme scan;load nvme 0:1 ${kernel_addr_c} uImage;bootm ${kernel_addr_c} - ${fdt_addr_r}"
saveenv
boot
```


### 总结
---

没接触过这块，玩起来还是有点吃力，期间也学习和查了不少东西，所以按照本文写的不一定能完全执行，也可能某些细节有遗漏，请酌情阅读，仅供参考。
