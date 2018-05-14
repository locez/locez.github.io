---
title: Yubikey PGP Card
date: 2018-04-24 14:28:21
categories:
 - linux
 - cryptography
tags:
 - gnupg
 - yubikey
 - PGP
---

##### AUTHOR: [Locez](http://locez.com)
##### VERSION: 1

### 本文环境
---

 - OS：Gentoo
 - Kernel：4.9.76
 - gpg (GnuPG): 2.2.4
 - yubikey-manager 0.6.0
 - pcsc-tools 1.4.27

### 什么是 PGP 卡？
---
在加密技术中，PGP 卡是一种智能卡，这种智能卡可以执行加密，解密，数字签名/验证，认证等任务。它允许我们安全地存储密钥，私钥和密码不能用任何命令或功能从卡上读取，但是可以将新密钥写入到卡上覆盖旧密钥，而 Yubikey 里面有 PGP Card 的功能，因此可以将密钥安全地存进去，使得我们的密钥有一个物理设备的载体，类似于银行的 U 盾

<!-- more -->
### 软件安装
---

Yubikey 相关的包都被 Gentoo 标记为 `Masked`，所以首先是要解除掉才能安装

``` bash
# vim /etc/portage/package.accept_keywords/yubikey
```

将以下内容填入

``` bash
# required by app-crypt/yubikey-manager-0.6.0::gentoo
# required by app-crypt/yubikey-manager (argument)
=dev-python/pyusb-1.0.2 ~amd64
# required by app-crypt/yubikey-manager (argument)
=app-crypt/yubikey-manager-0.6.0 ~amd64
# required by app-crypt/yubikey-manager-0.6.0::gentoo
# required by app-crypt/yubikey-manager (argument)
=dev-python/pyscard-1.9.5 ~amd64
# required by sys-apps/pcsc-tools-1.4.27::gentoo
# required by pcsc-tools (argument)
=dev-perl/pcsc-perl-1.4.14 ~amd64
# required by pcsc-tools (argument)
=sys-apps/pcsc-tools-1.4.27 ~amd64
```

#### 安装 yubikey-manager
---

``` bash
# emerge --ask app-crypt/yubikey-manager
```
#### 安装 pcsc-tools
---

``` bash
# emerge --ask  pcsc-tools
```

### 连接设备
---

因为本人是的桌面环境是 awesome，因此需要禁用 `OTP` 功能，只启用 `U2F`、`CCID`

``` bash
$ ykpersonalize -m5
```
**注意：** 启用 3 个功能只需要 `ykpersonalize -m6` 即可

启动 pcscd 守护进程

``` bash
# systemctl start pcscd.socket 
```
测试连接
---

``` bash
$ pcsc_scan
PC/SC device scanner
V 1.4.27 (c) 2001-2011, Ludovic Rousseau <ludovic.rousseau@free.fr>
Compiled with PC/SC lite version: 1.8.22
Using reader plug'n play mechanism
Scanning present readers...
0: Yubico Yubikey 4 U2F+CCID 00 00

Tue Apr 24 14:46:32 2018
Reader 0: Yubico Yubikey 4 U2F+CCID 00 00
...
...
```

``` bash
$ gpg-connect-agent --hex "scd apdu 00 f1 00 00" /bye
D[0000]  04 03 07 90 00                                     .....
OK
```

### 编辑 PGP 卡信息
---

``` bash
$ gpg --card-edit

Reader ...........: Yubico Yubikey 4 U2F CCID 00 00
Application ID ...: D2760001240102010006069500550000
Version ..........: 2.1
Manufacturer .....: Yubico
Serial number ....: 06950055
Name of cardholder: [not set]
Language prefs ...: [not set]
Sex ..............: unspecified
URL of public key : [not set]
Login data .......: [not set]
Signature PIN ....: not forced
Key attributes ...: rsa2048 rsa2048 rsa2048
Max. PIN lengths .: 127 127 127
PIN retry counter : 3 0 3
Signature counter : 0
Signature key ....: [none]
Encryption key....: [none]
Authentication key: [none]
General key info..: [none]

```
设置密码等信息，默认的 PIN 是 `123456`，PUK 是 `12345678`

``` bash
gpg/card> admin
Admin commands are allowed

gpg/card> passwd
gpg: OpenPGP card no. D2760001240102010006069500550000 detected

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 1
PIN changed.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 3
PIN changed.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 4
Reset Code set.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? q

gpg/card>
```
设置个人信息

``` bash
gpg/card> name
Cardholder's surname: Locez
Cardholder's given name: Locez

gpg/card> lang
Language preferences: zh

gpg/card> sex
Sex ((M)ale, (F)emale or space): M

gpg/card> login
Login data (account name): Locez

gpg/card>

Reader ...........: Yubico Yubikey 4 U2F CCID 00 00
Application ID ...: D2760001240102010006069500550000
Version ..........: 2.1
Manufacturer .....: Yubico
Serial number ....: 06950055
Name of cardholder: Locez Locez
Language prefs ...: zh
Sex ..............: male
URL of public key : [not set]
Login data .......: Locez
Signature PIN ....: not forced
Key attributes ...: rsa2048 rsa2048 rsa2048
Max. PIN lengths .: 127 127 127
PIN retry counter : 3 3 3
Signature counter : 0
Signature key ....: [none]
Encryption key....: [none]
Authentication key: [none]
General key info..: [none]

gpg/card>
```

### 生成与导入 key
---
生成 PGP 主密钥

``` bash

$ gpg --full-generate-key
gpg (GnuPG) 2.2.4; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

gpg: directory '/home/locez/.gnupg' created
gpg: keybox '/home/locez/.gnupg/pubring.kbx' created
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Locez
Email address: loki.a@live.cn
Comment: 
You selected this USER-ID:
    "Locez <loki.a@live.cn>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

```

此时可以动动鼠标键盘让他收集足够的随机数据

生成一个用于认证的子密钥

``` bash
$ gpg --expert --edit-key Locez
gpg (GnuPG) 2.2.4; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

gpg> addkey
```


然后跟着向导进行选择就可以了，通常是选择 `(8) RSA (set your own capabilities)` ，然后 `4096` 位密钥
其中子密钥对的类型选择应该如下

``` bash
Possible actions for a RSA key: Sign Encrypt Authenticate
Current allowed actions: Sign Encrypt  ### 此处显示的为该子密钥可以使用的用途，
                                       ### 通过多次选择下面的开关进行调整

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection?
```

然后重复上面的操作再次添加一个用于签名的子密钥，最终效果大概如下，使用 `save` 命令保存退出

``` bash
gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
pub  rsa4096/AAAAAAAAAAAAAAAA
     created: 2018-04-24  expires: never       usage: SC
     trust: ultimate      validity: ultimate
ssb  rsa4096/BBBBBBBBBBBBBBBB
     created: 2018-04-24  expires: never       usage: E
ssb  rsa4096/CCCCCCCCCCCCCCCC
     created: 2018-04-24  expires: never       usage: A
ssb  rsa4096/DDDDDDDDDDDDDDDD
     created: 2018-04-24  expires: never       usage: S
[ultimate] (1). Locez <loki.a@live.cn>
```

#### 备份公钥与私钥
---
当我们把密钥导入 Yubikey 的时候，我们就无法取出密钥，因此在导入之前最好备份
备份主密钥私钥

``` bash
$ gpg --export-secret-key --armor Locez >> master.key
```
备份主密钥公钥

``` bash
$ gpg -a --export Locez >> master.pub
```
当然也可以对单独子密钥进行备份，语法如下

``` bash
gpg --export-secret-subkeys  --armor DDDDDDDDDDDDDDDD >> sign.key
```
`DDDDDDDDDDDDDDDD` 为子密钥的指纹信息 
子密钥公钥当然也可以单独导出，但是在导出主密钥公钥的时候其实已经把子密钥公钥导出了，因此可以不必重复备份

#### 导入进 Yubikey
---
备份做好以后，就可以将 `RSA` 密钥导入进 Yubikey 了，通常不建议直接将主密钥导入，因此在本文除了主密钥外，另外有三个子密钥用于导入进 Yubikey

采用 `key index` 语法选择或者取消选择密钥，主密钥为 0， 其它依次递增，被选中会有星号

``` bash
gpg> key 1
ssb* rsa4096/BBBBBBBBBBBBBBBB
     created: 2018-04-24  expires: never       usage: E
```
然后接着

``` bash
gpg> keytocard
Signature key ....: [none]
Encryption key....: [none]
Authentication key: [none]

Please select where to store the key:
   (2) Encryption key
Your selection? 2

```
取消选择子密钥 1 并选择子密钥 2 

``` bash
gpg> key 1
gpg> key 2
gpg> keytocard
Signature key ....: [none]
Encryption key....: BBBB BBBB BBBB BBBB BBBB  BBBB BBBB BBBB BBBB BBBB 
Authentication key: [none]

Please select where to store the key:
   (3) Authentication key
Your selection? 3
```
重复操作，直至把 3 个子密钥都导入进 Yubikey，最后 `save` 命令保存,当你看到多了这样的 `card-no` 字样即表面导入成功

``` bash
gpg --edit-key Locez
gpg (GnuPG) 2.2.4; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  rsa4096/AAAAAAAAAAAAAAAA
     created: 2018-04-24  expires: never       usage: SC
     trust: ultimate      validity: ultimate
ssb  rsa4096/BBBBBBBBBBBBBBBB
     created: 2018-04-24  expires: never       usage: E
     card-no: 0000 00000001
ssb  rsa4096/CCCCCCCCCCCCCCCC
     created: 2018-04-24  expires: never       usage: A
     card-no: 00000 00000001
ssb  rsa4096/DDDDDDDDDDDDDDDD
     created: 2018-04-24  expires: never       usage: S
     card-no: 000000 00000001
[ultimate] (1). Locez <loki.a@live.cn>

```
#### 删除主密钥私钥
---
通常，为了保证安全，日常操作采用子密钥足以，主密钥私钥应该离线保存在一个非常安全的地方，对的就是刚刚备份的那些东西需要离线存储，例如找个保险柜，此时先删除主密钥私

``` bash
$ gpg --delete-secret-key Locez
gpg (GnuPG) 2.2.4; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


sec  rsa4096/AAAAAAAAAAAAAAAA 2018-04-24 Locez <loki.a@live.cn>

Delete this key from the keyring? (y/N) y
This is a secret key! - really delete? (y/N) y
```
还可通过输入以下命令进行确认, `sec` 后的 `#` 即表明主密钥私钥不可用

``` bash
$ gpg -K
/home/locez/.gnupg/pubring.kbx
------------------------------
sec#  rsa4096 2018-04-24 [SC]     
      AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
uid           [ultimate] Locez <loki.a@live.cn>
ssb>  rsa4096 2018-04-24 [E]
ssb>  rsa4096 2018-04-24 [A]
ssb>  rsa4096 2018-04-24 [S]
```
同样输入 `gpg --edit-key Locez` 会看到 `Secret subkeys are available.` 字样，是子密钥可用，而不是原来的主密钥了

### 简单测试
---
为了验证卡片写入成功，做个简单的测试，先拔掉 Yubikey

``` bash
$ echo "Hello, this is a test" > test
$ gpg --output test.en -se test 
You did not specify a user ID. (you may use "-r")

Current recipients:

Enter the user ID.  End with an empty line: Locez

Current recipients:
rsa4096/BBBBBBBBBBBBBBBB 2018-04-24 "Locez <loki.a@live.cn>"

Enter the user ID.  End with an empty line: 
```
空行结束，然后会要求你插入 Yubikey 并输入 PIN 进行加密

解密如下

``` bash
gpg --decrypt test.en
gpg: encrypted with 4096-bit RSA key, ID BBBBBBBBBBBBBBBB, created 2018-04-24
      "Locez <loki.a@live.cn>"
Hello, this is a test
gpg: Signature made Tue 24 Apr 2018 09:08:28 PM CST
gpg:                using RSA key BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
gpg: Good signature from "Locez <loki.a@live.cn>" [ultimate]
```

### 参考资料
---
 - [https://developers.yubico.com/PGP/](https://developers.yubico.com/PGP/)
 - [https://en.wikipedia.org/wiki/OpenPGP_card](https://en.wikipedia.org/wiki/OpenPGP_card)
 - [https://wiki.archlinux.org/index.php/GnuPG](https://wiki.archlinux.org/index.php/GnuPG)
 - [https://zhuanlan.zhihu.com/p/24103240](https://zhuanlan.zhihu.com/p/24103240)
