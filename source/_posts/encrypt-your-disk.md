---
title: 加密你的磁盘
date: 2018-05-13 16:04:19
categories:
 - linux
 - cryptography
tags:
 - linux
 - luks
---

##### AUTHOR: [Locez](http://locez.com)
##### VERSION: 1

数据的安全，保密性在现在的生活中显得越来越重要。随着数字化的时代的来临，越来越多的数据被数字化，特别是更多有关于我们隐私的数据在不断生成，甚至还有我们需要离线保存的密钥等。而且通常我们使用磁盘，USB 闪存，SD 卡等存储介质进行存储，即便我们已经离线存储，仍然不能保证该存储介质不会丢失，如果丢失那么对于我们来说有可能是灾难性的事件。因此对这些离线存储的重要数据，再次进行进行加密是非常有必要的，本文将告诉你如何加密你的移动存储介质。

在此之前先介绍一下 LUKS：
LUKS （Linux Unified Key Setup）是 Linux 硬盘加密的标准。 通过提供标准的磁盘格式，它不仅可以促进发行版之间的兼容性，还可以提供对多个用户密码的安全管理。 与现有解决方案相比，LUKS 将所有必要的设置信息存储在分区信息首部中，使用户能够无缝传输或迁移其数据。

<!-- more -->
### 1. 环境
---

 - OS: Gentoo
 - Kernel: 4.9.95
 - tools: cryptsetup


### 2. 内核配置
---

 通常来说，大部分发行版的内核都已经配置了相关的加密部分，因此非 gentoo 用户可以跳过此部分

 配置 device mapper 和 crypt target
 
 ``` bash
[*] Enable loadable module support
Device Drivers --->
    [*] Multiple devices driver support (RAID and LVM) --->
        <*> Device mapper support
        <*>   Crypt target support
```

配置加密 API

``` bash
[*] Cryptographic API --->
    <*> XTS support
    <*> SHA224 and SHA256 digest algorithm
    <*> AES cipher algorithms
    <*> AES cipher algorithms (x86_64)
    <*> User-space interface for hash algorithms
    <*> User-space interface for symmetric key cipher algorithms
```

编译新内核并配置应用，然后重启

``` bash
# make -j9 && make modules_install && make install
```


### 3. 安装软件
---

通常的发行版已经预装了该软件包，可以直接使用，下面是 Gentoo 的安装方法

``` bash
# emerge --ask sys-fs/cryptsetup
```


### 4. 创建加密分区
---

注意，该操作会清空你选择分区或设备上的所有数据，请谨慎操作，输入大写的 `YES` 确认

``` bash
# cryptsetup -s 512 luksFormat /dev/sdd

WARNING!
========
This will overwrite data on /dev/sdd irrevocably.

Are you sure? (Type uppercase yes): YES
Enter passphrase: 
Verify passphrase:
```


### 5. 利用 key file 加密分区
---

除了密码之外，还可以选择使用 `key file` 解密你的硬盘，也就是相当于一个密钥，当然可以也可以只使用 key file 或者同时使用密码与 key file



#### 5.1 生成随机 key file
---

``` bash
# dd if=/dev/urandom of=/root/enc.key bs=1 count=4096
```


#### 5.2 添加 key file 作为密码之一
---

``` bash
# cryptsetup luksAddKey  /dev/sdd /root/enc.key
Enter any existing passphrase:
```


#### 6. 移除解密密码
---

移除普通密码

``` bash
# cryptsetup luksRemoveKey /dev/sdd
Enter LUKS passphrase to be deleted: ...
```

移除 key file 密码

``` bash
# cryptsetup luksRemoveKey -d /root/enc.key /dev/sdd
```

**注意**：千万不要将所有密码移除，至少需要留有一个密码访问设备，移除操作不可撤销


### 7. 解密与挂载
--- 

#### 7.1 密码解密
---

``` bash
# cryptsetup luksOpen /dev/sdd myusb
Enter passphrase for /dev/sdd:
```


#### 7.2 key file 解密
---

``` bash
# cryptsetup luksOpen -d /root/enc.key /dev/sdd myusb
```

#### 7.3 创建文件系统
---

在挂载使用之前，我们仍然需要对设备创建文件系统才可以使用，可以选择任何你喜欢的文件系统，例如 `btrfs`，`ext4`，`vfat`，`ntfs` 等

``` bash
# mkfs.ext4 /dev/mapper/myusb
mke2fs 1.43.6 (29-Aug-2017)
Creating filesystem with 488448 4k blocks and 122160 inodes
Filesystem UUID: 995e172a-2bc6-432c-a60f-2d4d7093e748
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
```


#### 7.4 挂载
---

现在可以像正常分区一样挂载我们的加密分区设备了

``` bash
# mount /dev/mapper/myusb /mnt/
# df -h
/dev/mapper/myusb  1.9G  5.7M  1.7G   1% /mnt
```


#### 7.5 卸载挂载点并关闭加密分区
---

``` bash
# umount /mnt
# cryptsetup luksClose myusb
```


### 8. 总结
---

在完成整个步骤以后，您现在需要做的就是妥善保管您的加密存储，可采用同样的方式加密多个设备进行备份，因为谁也不能保证这移动设备会不会在什么时候丢掉。


