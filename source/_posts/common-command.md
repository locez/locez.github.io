title: 新手指南： 新手应该知道的 26 个命令
date: 2015-08-26 18:53:25
categories:
 - Linux
tags:
 - linux
 - command

---

##### AUTHOR: [Locez](http://locez.com)
##### VERSION: 1

![](https://image.locez.com/blog/common-command-00.jpg)
当你进入了 Linux 的世界，在下载、[安装](http://locez.com/Linux/install-ubuntu-and-fedora/) 了某个 Linux 发行版，体验了 Linux 桌面并安装了一些你喜爱和需要的 [软件](http://locez.com/Linux/install-program-on-ubuntu-fedora/) 之后，应该去了解下 Linux 真正的魅力所在：**命令行** 。每一个命令其实就是一个程序，借助这些命令，我们可以办到非常多的事情。下面将会为大家介绍一下几个常用的命令。

<!--more-->
* [如何寻求帮助？](#help)
    * [man](#man)
    * [info](#info)
    * [help](#command_help)
* [如何简单操作？](#operate)
    * [光标](#cursor)
    * [Tab 补全](#tab)
* [常用命令](#common_command)
    * [cd](#cd)
    * [ls](#ls)
    * [pwd](#pwd)
    * [mkdir](#mkdir)
    * [rm](#rm)
    * [cp](#cp)
    * [mv](#mv)
    * [cat](#cat)
    * [more](#more)
    * [less](#less)
    * [nano](#nano)
    * [reboot](#reboot)
    * [poweroff](#poweroff)
    * [ping](#ping)
    * [grep](#grep)
    * [mount](#mount)
    * [umount](#umount)
    * [tar](#tar)
    * [ln](#ln)
    * [chown](#chown)
    * [chmod](#chmod)
    * [useradd](#useradd)
    * [passwd](#passwd)
    * [whereis](#whereis)
    * [find](#find)
    * [wget](#wget)


<h2 id="help">如何寻求帮助？</h2>
---

在 Linux 下遇到问题，最重要的是要自己寻求帮助，下面是三种寻求帮助的方法。

<h4 id="man">man</h4>
---

`man` 是 Linux 的系统手册，即 `manual` 。因为大多数程序都会自带手册，所以可以通过 `man` 命令获取帮助。执行以后，在 **man page** 页面中按 `q` 退出。

获取 `ls` 的帮助

``` zsh
$ man ls
```

查看有多少（针对不同方面的）同名的手册

``` zsh
$ man -f ls
ls (1)               - list directory contents
ls (1p)              - list directory contents
```

查看特定的手册

``` zsh
$ man 1p ls
```


<h4 id="info">info</h4>
---

与 `man` 不同的是，可以像浏览网页一样在各个节点中跳转。
从文档首页开始浏览

``` zsh
$ info
```

获取特定程序的帮助

``` zsh
$ info program
```


<h4 id="command_help">help</h4>
---

除了上面的两种方法外，还有一种简单使用的方法，那就是 `--help` 参数，一般程序都会有这个参数，会输出最简单有用的介绍。

``` zsh
$ man --help       ### 获取 man 的帮助
$ info --help      ### 获取 info 的帮助
```


<h2 id="operate">如何简单操作？</h2>
---

在 `Terminal(终端)` 中，有许多操作技巧，这里就介绍几个简单的。


<h4 id="cursor">光标</h4>
---

    - up(方向键上)      可以调出输入历史执行记录，快速执行命令
    - down(方向键下)    配合 up 选择历史执行记录
    - Home             移动光标到本行开头
    - End              移动光标到本行结尾
    - PgUp             向上翻页
    - PaDN             向下翻页
    - ctrl + c         终止当前程序

<h4 id="tab">Tab 补全</h4>
---

`Tab` 补全是非常有用的一个功能，可以用来自动补全命令或文件名，省时准确。
  - 未输入状态下连按**两次** `Tab` 列出所有可用命令
  - 已输入部分命令名或文件名，按 `Tab` 进行自动补全，多用你就肯定会喜欢的了。
  

<h2 id="common_command"> 常用命令</h2>
---

<h4 id="cd">cd</h4>
---

`cd` 是打开某个路径的命令，也就是打开某个文件夹，并跳转到该处。

``` zsh
$ cd path      ### path 为你要打开的路径。
```

其中 path 有绝对路径和相对路径之分，绝对路径强调从 `/` 起，一直到所在路径。相对路径则，相对于当前路径来说，假设当前家目录有 `etc` 文件夹（绝对路径应为 `/home/username/etc`），如果直接 `cd etc` 则进入此文件夹，但若是 `cd /etc/` 则是进入系统 `etc` ，多琢磨一下就可以理解了。另外在 Linux 中， `.` 代表当前目录， `..` 代表上级目录，因此返回上级目录可以 `cd ..` 。


<h4 id="ls">ls</h4>
---

`ls` 即 **list** ，列出文件。

``` zsh
$ ls       ### 仅列出当前目录可见文件
$ ls -l    ### 列出当前目录可见文件详细信息
$ ls -hl   ### 列出详细信息并以可读大小显示文件大小
$ ls -al   ### 列出所有文件（包括隐藏）的详细信息
```


#### **注意**： Linux 中 以 `.` 开头的文件或文件夹均为隐藏文件或隐藏文件夹。

<h4 id="pwd">pwd</h4>
---

`pwd` 用于返回当前工作目录的名字，为绝对路径名。
``` zsh
$ pwd
/home
```


<h4 id="mkdir">mkdir</h4>
---

`mkdir` 用于新建文件夹。

``` zsh
$ mkdir folder
$ mkdir -p folder/subfolder    ### -p 参数为当父目录存在时忽略，若不存在则建立，用此参数可建立多级文件夹 
```


<h4 id="rm">rm</h4>
---

`rm` 即 **remove** ，删除文件。

``` zsh
$ rm filename    ### 删除 filename
$ rm -i filename   ### 删除 filename 前提示，若多个文件则每次提示
$ rm -rf folder/subfolder/  ### 递归删除 subfolder 下所有文件及文件夹，包括 subfolder 自身
$ rm -d folder     ###  删除空文件夹
```


<h4 id="cp">cp</h4>
---

`cp` 即 **copy** ，复制文件。

``` zsh
$ cp source dest            ### 将 source 复制到 dest
$ cp folder/*  dest         ### 将 folder 下所有文件(不含子文件夹中的文件)复制到 dest
$ cp -r folder  dest        ### 将 folder 下所有文件（包含子文件夹中的所有文件）复制到 dest
```


<h4 id="mv">mv</h4>
---

`mv` 即 **move** ，移动文件。

``` zsh
$ mv source  folder        ### 将 source 移动到 folder 下，完成后则为  folder/source
$ mv -i source folder      ### 在移动时，若文件已存在则提示 **是否覆盖** 
$ mv source dest           ### 在 dest 不为目录的前提下，重命名 source 为 dest
```


<h4 id="cat">cat</h4>
---

`cat` 用于输出文件内容到 Terminal 。

``` zsh
$ cat /etc/locale.gen     ### 输出 locale.gen 的内容 
$ cat -n /etc/locale.gen  ### 输出 locale.gen 的内容并显示行号
```


<h4 id="more">more</h4>
---

`more` 与 `cat` 相似，都可以查看文件内容，所不同的是，当一个文档太长时， `cat` 只能展示最后布满屏幕的内容，前面的内容是不可见的。这时候可用 `more` 逐行显示内容。

``` zsh
$ more /etc/locale.gen
$ more +100 /etc/locale.gen       ### 从 100 行开始显示
```


<h4 id="less">less</h4>
---

`less` 与 `more` 相似，不过 `less` 支持上下滚动查看内容，而 `more` 只支持逐行显示。

``` zsh 
$ less /etc/locale.gen
$ less +100 /etc/locale.gen
```


<h4 id="nano">nano</h4>
---

`nano` 是一个简单实用的文本编辑器，入门简单。

```zsh
$ nano  filename       ### 编辑 filename 文件，若文件不存在，则新打开一个文件，若退出时保存，则创建该文件

```
编辑完后，`ctrl + X` 提示是否保存，按 `y` 确定保存即可。

#### **注意**：在使用过程中可用 `ctrl + G` 获取帮助。


<h4 id="reboot">reboot</h4>
---

`reboot` 为重启命令。

``` zsh
# reboot         ### '$' 和 '#' 的区别在于 '$' 普通用户即可执行，而 '#' 为 root 用户才可执行，或普通用户使用 'sudo'
```


<h4 id="poweroff">poweroff</h4>
---

`poweroff` 为关机命令。

``` zsh
# poweroff  ### 马上关机
```


<h4 id="ping">ping</h4>
---

`ping` 主要用于测试网络连通，通过对目标机器发送数据包来测试两台主机是否连通，及延时情况。

``` zsh
$ ping locez.com    ### 通过域名 ping，若 DNS 未设置好，可能无法 ping 通
$ ping linux.cn
PING linux.cn (211.157.2.94) 56(84) bytes of data.
64 bytes from 211.157.2.94.static.in-addr.arpa (211.157.2.94): icmp_seq=1 ttl=53 time=41.5 ms
64 bytes from 211.157.2.94.static.in-addr.arpa (211.157.2.94): icmp_seq=2 ttl=53 time=40.4 ms
64 bytes from 211.157.2.94.static.in-addr.arpa (211.157.2.94): icmp_seq=3 ttl=53 time=41.9 ms
^C
--- linux.cn ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 40.406/41.287/41.931/0.644 ms

$ ping 211.157.2.94   ### 通过 IP 地址 ping ，若无法 ping 通可能是网络连接出现问题
```


<h4 id="grep">grep</h4>
---

`grep` 主要用于返回匹配的项目，支持正则表达式。

``` zsh
$ grep PATTERN filename      ### 返回所有含有 PATTERN 的行
$ grep zh_CN /etc/locale.gen ### 返回所有含 zh_CN 的行 
```


<h4 id="mount">mount</h4>
---

`mount` 用于挂载一个文件系统，需要 `root` 用户执行。一个磁盘可分为若干个分区，在分区上面可以创建文件系统，而挂载点则是提供一个访问的入口，将一个分区的文件系统挂载到某个目录中，称这个目录为挂载点，并且可以通过这个挂载点访问该文件系统中的内容。
例如一块硬盘在 Linux 中表示为 `/dev/sda` 那么它上面的分区应该表示为 `/dev/sda1` 、`/dev/sda2` 。

``` zsh
# mount                       ### 输出系统目前的挂载信息
# mount /dev/sda1 /mnt        ### 将 sda1 挂载到 /mnt 中
# cd /mnt                     ### 直接通过 /mnt 访问内容
# mount -o remount,rw  /mnt   ### 重新挂载 sda1 到 /mnt 并设置为 可读写 
# mount -a                    ### 挂载 fstab 文件配置好的文件系统
```


<h4 id="umount">umount</h4>
---

`umount` 与 `moung` 相反，是卸载一个挂载点，即取消该入口。

``` zsh
# umount /mnt                 ### 卸载 /mnt 这个挂载点的文件系统
# umount -a                   ### 卸载所有已挂载的文件系统
```


<h4 id="tar">tar</h4>
---

`tar` 主要用于创建归档文件，和解压归档文件，其本身是没有压缩功能的，但可以调用 `gzip` 、 `bzip2` 进行压缩处理。
参数解释：

  - `-c` 创建归档
  - `-x` 解压归档
  - `-v` 显示处理过程
  - `-f` 目标文件，其后必须紧跟 目标文件
  - `-j` 调用 `bzip2` 进行解压缩
  - `-z` 调用 `gzip` 进行解压缩
  - `-t` 列出归档中的文件

``` zsh
$ tar -cvf filename.tar .       ### 将当前目录所有文件归档，但不压缩，注意后面有个 ’.‘ ，不可省略，代表当前目录的意思 
$ tar -xvf filename.tar         ### 解压 filename.tar 到当前文件夹
$ tar -cvjf filename.tar.bz2 .  ### 使用 bzip2 压缩
$ tar -xvjf  filename.tar.bz2   ### 解压 filename.tar.bz2 到当前文件夹
$ tar -cvzf filename.tar.gz     ### 使用 gzip  压缩
$ tar -xvzf filename.tar.gz     ### 解压 filename.tar.gz 到当前文件夹
$ tar -tf   filename            ### 只查看 filename 归档中的文件，不解压
```


<h4 id="ln">ln</h4>
---

`ln` 主要用于在两个文件中创建链接，链接又分为 `Hard Links` (硬链接)和 `Symbolic Links` (符号链接或软链接)，其中默认为创建**硬链接**，使用 `-s` 参数指定创建软链接。
  - 硬链接主要是增加一个文件的链接数，只要该文件的链接数不为 `0` ，该文件就不会被物理删除，所以删除一个具有多个硬链接数的文件，必须删除所有它的硬链接才可删除。
  - 软链接简单来说是为文件创建了一个类似快捷方式的东西，通过该链接可以访问文件，修改文件，但不会增加该文件的链接数，删除一个软链接并不会删除源文件，即使源文件被删除，软链接也存在，当重新创建一个同名的源文件，该软链接则指向新创建的文件。
  - 硬链接只可链接两个文件，不可链接目录，而软链接可链接目录，所以软链接是非常灵活的。
  
``` zsh
$ ln source dest       ### 为 source 创建一个名为 dest 的硬链接
$ ln -s source dest    ### 为 source 创建一个名为 dest 的软链接
```


<h4 id="chown">chown</h4>
---

`chown` 用于改变一个文件的所有者及所在的组。

``` zsh
# chown user filename        ### 改变 filename 的所有者为 user
# chown user:group filename  ### 改变 filename 的所有者为 user，组为 group
# chown -R root folder      ### 改变 folder 文件夹及其子文件的所有者为 root
```


<h4 id="chmod">chmod</h4>
---

`chmod` 永远更改一个文件的权限，主要有 **读取**  、 **写入** 、 **执行** ，三种权限，其中 **所有者** 、 **用户组** 、 **其他** 各占三个，因此 `ls -l` 可以看到如下的信息

    -rwxr--r-- 1 locez users   154 Aug 30 18:09 filename

其中 `r=read` ， `w=write` ， `x=execute` 

``` zsh
# chmod +x filename        ### 为 user ，group ，others 添加执行权限
# chmod -x filename        ### 取消 user ， group ，others 的执行权限
# chmod +w filename        ### 为 user 添加写入权限
# chmod ugo=rwx filename   ### 设置 user ，group ，others 具有 读取、写入、执行权限
# chmod ug=rw filename     ### 设置 user ，group 添加 读取、写入权限
# chmod ugo=--- filename   ### 取消所有权限
```


<h4 id="useradd">useradd</h4>
---

`useradd` 用于添加一个普通用户。

```
# useradd -m -g users -G audio -s /usr/bin/bash newuser     
### -m 创建 home 目录， -g 所属的主组， -G 指定该用户在哪些附加组， -s 设定默认的 shell ，newuser 为新的用户名
```


<h4 id="passwd">passwd</h4>
---

`passwd` 用于改变用户登录密码。

``` zsh
$ passwd                 ### 不带参数更改当前用户密码
# passwd newuser       ### 更改上述新建的 newuser 的用户密码
```


<h4 id="whereis">whereis</h4>
---

`whereis` 用于查找文件、手册等。

``` zsh
$ whereis bash 
bash: /usr/bin/bash /etc/bash.bashrc /etc/bash.bash_logout /usr/share/man/man1/bash.1.gz /usr/share/info/bash.info.gz
$ whereis -b bash       ### 仅查找 binary
bash: /usr/bin/bash /etc/bash.bashrc /etc/bash.bash_logout
$ whereis -m bash       ### 仅查找 manual
bash: /usr/share/man/man1/bash.1.gz /usr/share/info/bash.info.gz
```

<h4 id="find">find</h4>
---

`find` 也用于查找文件，但更为强大，支持正则，并且可将查找结果传递到其他命令。

``` zsh
$ find . -name PATTERN    ### 从当前目录查找符合 PATTERN 的文件
$ find /home -name PATTERN -exec ls -l {} \;  ### 从 /home 文件查找所有符合 PATTERN 的文件，并交由 ls 输出详细信息
```


<h4 id="wget">wget</h4>
---

`wget` 是一个下载工具，简单强大。

``` zsh
$ wget -O newname.md https://github.com/LCTT/TranslateProject/blob/master/README.md     ###下载 README 文件并重命名为 newname.md
$ wget -c url     ### 下载 url 并开启断点续传
```

恭喜你，你已经学习了完了26 个基础的 Linux 命令。虽然这里只是一些最基础的命令，但是熟练使用这些命令就踏出了你从一位 Linux 新手成为 Linux 玩家的第一步！



