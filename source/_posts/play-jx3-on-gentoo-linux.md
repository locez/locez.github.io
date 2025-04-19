---
title: 在 Gentoo Linux 下畅玩剑网3
date: 2025-04-19 13:00:46
categories:
tags:
---

##### AUTHOR: [Locez](https://locez.com)
##### VERSION: 1

### 系统安装

---

对 Gentoo Linux 感兴趣的同学，可以参考我的一篇博客 [Gentoo Linux with ZFS](https://locez.com/linux/gentoo-linux-with-zfs/)

### Steam 安装

---

Gentoo 里面 Steam 安装需要开启 lib32，需要启用对应的 package use，我们直接参考官方文档 [https://wiki.gentoo.org/wiki/Steam](https://wiki.gentoo.org/wiki/Steam)

### 安装 PROTON-GE

---

使用如下脚本进行安装，安装后重启 steam 即可

``` bash
#!/bin/bash
set -euo pipefail

# make temp working directory
echo "Creating temporary working directory..."
rm -rf /tmp/proton-ge-custom
mkdir /tmp/proton-ge-custom
cd /tmp/proton-ge-custom

# download tarball
echo "Fetching tarball URL..."
tarball_url=$(curl -s https://api.github.com/repos/GloriousEggroll/proton-ge-custom/releases/latest | grep browser_download_url | cut -d\" -f4 | grep .tar.gz)
tarball_name=$(basename $tarball_url)
echo "Downloading tarball: $tarball_name..."
curl -# -L $tarball_url -o $tarball_name --no-progress-meter

# download checksum
echo "Fetching checksum URL..."
checksum_url=$(curl -s https://api.github.com/repos/GloriousEggroll/proton-ge-custom/releases/latest | grep browser_download_url | cut -d\" -f4 | grep .sha512sum)
checksum_name=$(basename $checksum_url)
echo "Downloading checksum: $checksum_name..."
curl -# -L $checksum_url -o $checksum_name --no-progress-meter

# check tarball with checksum
echo "Verifying tarball $tarball_name with checksum $checksum_name..."
sha512sum -c $checksum_name
# if result is ok, continue

# make steam directory if it does not exist
echo "Creating Steam directory if it does not exist..."
mkdir -p ~/.steam/root/compatibilitytools.d

# extract proton tarball to steam directory
echo "Extracting $tarball_name to Steam directory..."
tar -xf $tarball_name -C ~/.steam/root/compatibilitytools.d/
echo "All done :)"

```

### 安装 ntfs 支持

---

``` bash
sudo emerge --ask sys-fs/ntfs3g
```

**注意**：双系统挂载同一块硬盘，需要将 Windows 上的快速启动关了，否则 Linux 下挂载会变成 ReadOnly

### 挂载磁盘

---

``` bash
sudo mkdir -p /data/GameDisk1
sudo mount /dev/nvme1n1p2 /data/GameDisk1/
```

将磁盘加入 fstab，方便开机自动加载

``` bash
sudo genfstab -U / > /etc/fstab
```
具体的挂载参数应该如下 /etc/fstab

``` conf
UUID=AAAAAAAAAAAA   /data/GameDisk1 ntfs            rw,user_id=0,group_id=0,allow_other,blksize=4096        0 0
```

### Steam 设置
---

添加非 Steam 游戏，然后选择到剑网 3 的启动器 `SeasunGame.exe`

启动选项填入：
```
DXVK_ASYNC=1 LANG=zh_CN.UTF-8 LC_ALL=zh_CN.UTF-8 %command%
```

兼容性选择：`GE-Proton-9-*`

### 剑网3配置

---

剑网3中需要添加一个跳过显卡检测的配置，才能正常启动游戏

``` bash
cd /data/GameDisk1/JX3/SeasunGame/Game/JX3/bin/zhcn_hd/
vim config.ini
```
在 `Debug` Section 中添加 `SkipVideoCardScoreUpdate=1`，添加完配置大致如下：

``` ini
[Debug]
3DEngineDebugInfo=0
Console=0
PakFirst=0
EnableProfile=0
EnableDebugBreakCheck=0
TopPoint=0
SkipVideoCardScoreUpdate=1
```

### 配置 dxvk 优化

---

``` bash
mkdir -p ~/.config/dxvk
cat << EOF > ~/.config/dxvk/dxvk.conf
> # 显存与线程优化
dxgi.numBackBuffers = 3
d3d11.maxImplicitDiscardSize = 256
d3d12.enableMapMemoryFlag = True

# 画质增强（减少渲染模糊）
dxgi.syncInterval = 0
d3d11.forceTearing = True
> EOF
```

**注意**: 目前旗舰版电影画质以上直接登录会导致过图直接卡死，经过实测，只需要回到 Windows 下登录，然后选择画质标准，再将标准所有性能参数拉满，即可解决，然后也能得到不错的画质体验

### 引用

---
感谢 [【技术向讨论】关于在Wine (SteamDeck) 下运行剑网3的可行性与实现方法（证实可行！）](https://origin.jx3box.com/tool/39093) 一文中的 SkipVideoCardScoreUpdate=1 参数提供，少走了一些弯路
