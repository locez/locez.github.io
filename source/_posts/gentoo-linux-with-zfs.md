---
title: Gentoo Linux with ZFS
date: 2025-04-13 15:14:05
categories:
    linux
tags:
    zfs
---

##### AUTHOR: [Locez](https://locez.com)
##### VERSION: 1


### 背景

---

在学生时代我就非常喜欢 `Gentoo Linux`，在使用 Gentoo Linux 的过程中学习到了非常多有用宝贵的 Linux 基础知识，这些知识在我的职业生涯早期发挥了相当重要的作用。但是由于工作后的忙碌，以及工作需要使用公司的设备，逐渐没有空维护我的 Gentoo 操作系统。同时也由于笔记本性能后来确实无法跟上，后来将我的 XPS 13 多装了一个 `Arch Linux` 使用至今。

所幸随着我的游戏机纯 Windows 系统的性能逐渐下滑，无法维持正常的游戏质量，我决定新购设备，同时满足我的游戏以及回归 Gentoo 的需求。

本文主要记录这次安装 Gentoo 的过程，有些东西可能不会详细展开。


### 设备配置

---

 - CPU: AMD Ryzen 9 9950X3D 16-Core Processor
 - MainBoard: MPG X870E CARBON WIFI
 - Memroy Size: DDR5 96G
 - Disk: 2 * 2T (ZFS mirror)

### 准备

---

#### 文件

---


 - adminCD，支持 zfs 的 liveCD 安装环境
 - Gentoo Stage3 ISO 文件

我们使用 Gentoo 的 adminCD 来进行安装，我使用的是 [ventoy](https://github.com/ventoy/Ventoy) 进行 USB 启动镜像制作。

Gentoo 相关的文件，可以在 [Downloads](https://www.gentoo.org/downloads/) 中找到，也可以选择一个镜像站进行加速下载。

#### BIOS setup

---

 - 关闭 secure boot

### 安装

---

#### ZFS

----
由于我安装 Windows 时未手动分区，万恶的 Windows 这次只给我分了 100M 的 EFI 分区，所以我在此处需要进行额外的处理。

我要将 2根 SSD 前 16G 切出来做 ESP 分区，都切是为了做对齐，总体的分区布局如下：

```
Part. #     Size        Partition Type            Partition Name
----------------------------------------------------------------
            1007.0 KiB  free space
   1        16.0 GiB    EFI system partition      EFI System Partition
   2        1.8 TiB     Linux filesystem          ZFS0
            327.5 KiB   free space
```

```
Part. #     Size        Partition Type            Partition Name
----------------------------------------------------------------
            1007.0 KiB  free space
   1        16.0 GiB    Linux filesystem
   2        1.8 TiB     Linux filesystem          ZFS1
            327.5 KiB   free space
```

使用如下的命令创建 ZFS Pool，我们在 2 根 SSD 都划分了 16G，但是其中只需要一个 16G 做 ESP 分区，由于我的内存已经相对够大，所以我将另一根未设为 ESP 类型的，用作 ZFS 的 SLOG。根据需求，你也可以作为 SWAP 分区。

其中的 id 可以通过 `ls -al /dev/disk/by-id/` 得到。

``` bash
zpool create -o feature@allocation_classes=enabled \
             -o feature@async_destroy=enabled \
             -o feature@bookmarks=enabled \
             -o feature@bookmark_v2=enabled \
             -o feature@device_rebuild=enabled \
             -o feature@device_removal=enabled \
             -o feature@draid=enabled \
             -o feature@embedded_data=enabled \
             -o feature@empty_bpobj=enabled \
             -o feature@enabled_txg=enabled \
             -o feature@encryption=enabled \
             -o feature@extensible_dataset=enabled \
             -o feature@filesystem_limits=enabled \
             -o feature@hole_birth=enabled \
             -o feature@large_blocks=enabled \
             -o feature@large_dnode=enabled \
             -o feature@livelist=enabled \
             -o feature@log_spacemap=enabled \
             -o feature@obsolete_counts=enabled \
             -o feature@project_quota=enabled \
             -o feature@resilver_defer=enabled \
             -o feature@spacemap_histogram=enabled \
             -o feature@spacemap_v2=enabled \
             -o feature@userobj_accounting=enabled \
             -o feature@zpool_checkpoint=enabled \
             -o feature@zstd_compress=enabled \
             -o ashift=12 \
             -o autoexpand=on \
             -o autoreplace=on \
             -o autotrim=on \
             -O acltype=posixacl \
             -O canmount=off \
             -O devices=off \
             -O normalization=formC \
             -O relatime=on \
             -O xattr=sa \
             -O compression=zstd \
             -O dnodesize=auto \
             -O overlay=off \
             zroot \
               mirror \
                 nvme-ZHITAI_TiPlus7100_2TB_ZTA72T0AB2452406GT-part2 \
                 nvme-ZHITAI_TiPlus7100_2TB_ZTA72T0AB245240A4C-part2 \
               log \
                 nvme-ZHITAI_TiPlus7100_2TB_ZTA72T0AB245240A4C-part1 \

```

创建 rootfs 卷， 注意此处我使用了 zfs 加密，如果不需要可以去掉。

``` bash
zfs create -o mountpoint=legacy \
	-o canmount=noauto \
	-o encryption=aes-256-gcm \
	-o keylocation=prompt \
	-o keyformat=passphrase \
	zroot/root
```

创建 home 卷，此处将加密设置成与父卷一样的方式，但是不添加密码，我们就可以在开机时输入密码后共享同一个密码，无需二次解密。

``` bash
zfs create -o mountpoint=legacy \
	-o canmount=noauto \ 
	-o encryption=aes-256-gcm \
	zroot/root/home
```

挂载对应的卷和分区，注意此处挂载的是 ESP 分区

``` bash
mount -t zfs zroot/root /mnt/gentoo
mount -t zfs zroot/root/home /mnt/gentoo/home
mount /dev/nvme0n1p1 /mnt/gentoo/boot
```

#### 安装 Gentoo
---

下载解压 stage 3，我这里 example 选择 `systemd` 和 `desktop` 的，根据需求选择你要的。

``` bash
cd /mnt/gentoo
wget https://distfiles.gentoo.org/releases/amd64/autobuilds/20250406T165023Z/stage3-amd64-desktop-systemd-20250406T165023Z.tar.xz
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```
配置 `/etc/portage/make.conf`：

``` conf
USE="cjk dist-kernel dracut egl -handbook -kwallet networkmanager sddm systemd-boot wayland wacom"

MAKEOPTS="--jobs 16 --load-average 25"
INPUT_DEVICES="wacom"

CFLAGS="-march=native -O3 -pipe"
CXXFLAGS="${CFLAGS}"
CPU_FLAGS="aes avx avx2 avx512_bf16 avx512_bitalg avx512_vbmi2 avx512_vnni avx512_vp2intersect avx512_vpopcntdq avx512bw avx512cd avx512dq avx512f avx512ifma avx512vbmi avx512vl f16c fma3 mmx mmxext pclmul popcnt rdrand sha sse sse2 sse3 sse4_1 sse4_2 sse4a ssse3 vpclmulqdq"
CPU_FLAGS_X86="${CPU_FLAGS}"

GENTOO_MIRRORS="https://mirrors.ustc.edu.cn/gentoo https://mirrors.tuna.tsinghua.edu.cn/gentoo https://distfiles.gentoo.org"
```
其中的 CPU_FLAGS 可以通过 `cpuid2cpuflags` 得到。

配置必要的 package USE `/etc/portage/package.use/custom`：

``` bash
# required by Locez
*/* CPU_FLAGS_X86: aes avx avx2 avx512_bf16 avx512_bitalg avx512_vbmi2 avx512_vnni avx512_vp2intersect avx512_vpopcntdq avx512bw avx512cd avx512dq avx512f avx51
2ifma avx512vbmi avx512vl f16c fma3 mmx mmxext pclmul popcnt rdrand sha sse sse2 sse3 sse4_1 sse4_2 sse4a ssse3 vpclmulqdq
*/* VIDEO_CARDS: -* amdgpu radeonsi nvidia
```
遵循新的标准，在这里继续配置 `CPU_FLAGS` 和 `VIDEO_CARDS`，其中显卡的需要根据硬件具体进行配置，我这里含有 AMD 的集显和 nvidia 的显卡

准备 chroot

``` bash
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
mount --bind /run /mnt/gentoo/run
mount --make-slave /mnt/gentoo/run

chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
```

配置时区与区域

``` bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localetime

cat /etc/locale.gen | grep -v ^#
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8

locale-gen
```

配置内核与 ZFS 内核模块

``` bash
emerge-webrsync
emerge -av sys-kernel/linux-firmware
emerge -av sys-kernel/gentoo-kernel-bin
emerge -av sys-fs/zfs sys-fs/zfs-kmod
```
配置 `dracut` 生成 `initramfs`，添加 ZFS 支持，以便开机的时候能正确使用加载 ZFS，其中 `systemd-ask-password` 是因为上述创建 ZFS Pool 的步骤中，我们在创建子卷的时候填写了密码，用这个来提示我们输入密码。

**注意**：配置前后的空格是必须的

`/etc/dracut.conf.d/zol.conf`

``` bash
fsck="yes"
add_dracutmodules+=" zfs systemd-ask-password "
```
生成 initramfs

``` bash
emerge --config sys-kernel/gentoo-kernel-bin
```

安装更新系统

``` bash
emerge --ask --verbose --update --deep --newuse @world app-shells/fish
systemctl enable networkmanager
```

安装 KDE 与 SDDM

``` bash
emerge -auDv kde-plasma/plasma-meta sddm
systemctl enable sddm
```

修改 sddm 配置，文章有实效性，最新的配置可以参考： [plasma-wayland.conf](https://github.com/KDE/plasma-workspace/blob/master/sddm-wayland-session/plasma-wayland.conf)

``` conf
[General]
DisplayServer=wayland
GreeterEnvironment=QT_WAYLAND_SHELL_INTEGRATION=layer-shell
InputMethod=

[Theme]
Current=breeze

[Wayland]
CompositorCommand=kwin_wayland --no-global-shortcuts --no-lockscreen --inputmethod maliit-keyboard --locale1
```

创建普通用户，并且配置对应权限分组，其中 sddm 踩坑在登录后需要 daemon group 才能正常运行（截至 20250413 仍然是这样，请注意时间有效性）。

``` bash
useradd -m -G daemon,adm,wheel,audio,video,users -s /usr/bin/fish locez
passwd locez
```

配置启动项，通过 `bootctl` 安装，并且检查 /boot 下的配置是否正确

``` bash
bootctl install
ls /boot

❯ cat /boot/loader/entries/gentoo-6.12.21-gentoo-dist.conf
# Boot Loader Specification type#1 entry
# File created by /usr/lib/kernel/install.d/90-loaderentry.install (systemd 256.10)
title      Gentoo Linux
version    6.12.21-gentoo-dist
sort-key   gentoo
options    dokeymap root=ZFS=zroot/root ro
linux      /gentoo/6.12.21-gentoo-dist/linux
initrd     /gentoo/6.12.21-gentoo-dist/initrd
```

重启进入系统

``` bash
exit
cd
umount -l /mnt/gentoo/dev{/shm,/pts,}
umount -n -R /mnt/gentoo
zpool export zroot
reboot
```

到此为止， Gentoo 的整个安装过程已经结束，顺利的话能直接进入系统，不顺利的话只能 liveCD 继续 chroot 修问题啦。

如果需要 chroot 修问题，这个时候需要在 liveCD 中解密，首次创建的时候输入密码没这个步骤

``` bash
zpool import zroot
zfs load-key zroot/root
```
登录进去普通用户以后如果声音不正常还需要启动 `pipewire` 和 `wireplumber`

```bash
systemctl --user enable --now pipewire-pulse pipewire wireplumber
```

### 双系统启动配置

---

由于使用了 2 个 ESP 分区，双系统启动有两种方式，一种是开机的时候选择启动项，另一种则使用 systemd-boot 管理。

虽然 systemd-boot 无法直接管理另一个 ESP 分区中的启动，但是我们可以通过 UEFI SHELL 进行跳转。

首先安装 `edk2`，将其拷贝到我们的 /boot 中，这样启动选择后可以得到一个 UEFI SHELL，我们可以通过这个 SHELL 得到 Windows 的 `FS alias`

```
emerge -1 -a edk2-bin
cp /usr/share/edk2-ovmf/Shell.efi /boot/shellx64.efi
```
重启然后登录到 UEFI SHELL 中，使用 map 命令得到 Windows 的 UEFI 的 FS alias

然后手动创建以下启动项 `/boot/loader/entries/windows.conf`， 其中 `HD0b` 就是 FS alias
``` conf
title Windows 11
efi     /shellx64.efi
options -nointerrupt -nomap -noversion HD0b:EFI\Microsoft\Boot\Bootmgfw.efi
```

### 引用
---
 - [https://wiki.gentoo.org/wiki/ZFS/rootfs](https://wiki.gentoo.org/wiki/ZFS/rootfs)
 - [https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base)
 - [https://github.com/KDE/plasma-workspace/blob/master/sddm-wayland-session/plasma-wayland.conf](https://github.com/KDE/plasma-workspace/blob/master/sddm-wayland-session/plasma-wayland.conf)
 - [https://openzfs.github.io/openzfs-docs/man/master/7/zfsprops.7.html](https://openzfs.github.io/openzfs-docs/man/master/7/zfsprops.7.html)
 - [https://wiki.archlinux.org/title/Systemd-bootArchWiki-Systemd-boot](https://wiki.archlinux.org/title/Systemd-boot)
