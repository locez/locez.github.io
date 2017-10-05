title: Arch Linux 下的 QQ 解决方案
date: 2015-03-28 21:51:38
categories:
 - Linux

tags:
 - linux
---
##### AUTHOR: [Locez](http://locez.com)
##### VERSION: 4
##### UPDATE: 2015-08-31

##QQ 6.3 由于版本低，已不能登录，以下教程已不适用，有想要折腾 wine QQ 的仅供参考


Linux 上面玩 QQ 一直都是一个问题。Wine 算是一个解决方案，但是也有不少人失败了。由于 QQ 的特殊性，采取了一系列的保护措施，导致 QQ 这个 Windows 程序非常复杂，因此 Wine 在运行 QQ 时表现差强人意。本文将要安装的是 QQ6.3 ，更高的版本除非对 QQ 做出修改，否则很难安装成功，即使成功了，问题也挺多的（笔者已试验过 QQ7.4 安装）。写这个的目的主要是方便有人遇到问题截图提问，毕竟 Linux 的普及工作还得靠大家，对于日常聊天还是建议使用手机QQ 。

### 本文环境：
  - Arch Linux (其他发行版仅供参考)
  - KDE4 & LXDE & GNOME (其它请自测)

### 准备工具
***
　　- Wine
　　- winetricks

### 简介
***
  - `Wine` 是一个在类 Unix 系统中运行 Microsoft Windows 程序的软件， `Wine` 的全称是 `Wine Is Not Emulator` 意为 `Wine` 不是一个模拟器，它通过 API 转换技术做出 Linux 上对应于 Windows 的函数，从而调用 DLL 运行 Windows 程序。
  - `winetricks` 是一个 `script` ，可以用来下载和安装各种在 Wine 运行时需要的部分 DLL 和框架。如 `.NET` ， `Visual C++ runtime library` 或微软和其他公司的闭源程序，使用 `winetricks` 你可以快速安装某些常用的Windows程序。
  
<!--more-->
### 步骤
***
1.安装 `Wine`
``` zsh
$ sudo pacman -S wine
```
**注意**：64 位需启用 `multilib` 仓库才可安装 `Wine` ，去掉 `[multilib]` 及其 `Include的` “#”即可
``` zsh
$ sudo nano /etc/pacman.conf
```

2.安装 `winetricks`
``` zsh
$ sudo pacman -S winetricks
```

3.获取 `winetricks-zh` 的 `verb` 文件，更多详情请到： [winetricks-zh](https://github.com/hillwoodroc/winetricks-zh)

``` zsh
$ mkdir workforwine
$ cd workforwine
$ wget https://github.com/hillwoodroc/winetricks-zh/raw/master/verb/qq.verb
```

4.安装 QQ
``` zsh
$ WINEARCH=win32 winetricks qq
```
接下来是漫长的安装过程，会下载一系列需要的组件，将缓存在 `~/.cache/winetricks` ，请耐心等待。或许你还可以试试 `winetricks-zh` ， `winetricks-zh` 是 `winetricks` 的本地化版本，添加了更多国人可能用到的软件。
``` zsh
$ wget https://github.com/hillwoodroc/winetricks-zh/raw/master/winetricks-zh
$ chmod +x winetricks-zh
$ ./winetricks-zh
```
**注意**:若你觉得 `安装QQ` 这一步安装 `mono` 、`gecko` 太慢，如下图：
![installmono](http://7narru.com1.z0.glb.clouddn.com/image/installmono.png) 
![installingmono](http://7narru.com1.z0.glb.clouddn.com/image/installingmono.png)
![installgecko](http://7narru.com1.z0.glb.clouddn.com/image/installgecko.png)

根据配图我们可以知道 `mono` 是 `.NET` 需要的包，而 `gecko` 则是 `HTML` 需要的包，并且 wine 也更建议我们使用我们发行版中的 `mono` ， `gecko` 包，这有两个好处，一是更加符合自己的发行版，二是不用为每个 `PREFIEX` 单独安装，因此可以尝试以下操作，其他发行版仅供参考：
``` zsh
$ rm -rf ~/.wine
$ sudo pacman -S wine-mono
$ sudo pacman -S wine_gecko
$ WINEARCH=win32 winetricks qq
```

### 需要注意的几点
***
  - 请确保你安装有文泉驿字体 `sudo pacman -S wqy-microhei` 。
  - 用 `winetricks` 和 `winetricks-zh` 安装的区别仅在于安装目录不同， `winetricks` 未指定位置时默认 `～/.wine` ，而 `winetricks-zh` 则安装QQ至 `～/.local/share/wineprefixes/qq` 。
  - 有任何问题都可以直接删除上面提到的两个文件夹重来。
  - `wine` 的不稳定性，导致用 `winetricks` 安装字体有时可以解决，有时不可以，笔者试验了很多次以失败告终，希望有谁解决了可以告诉笔者。


### 其他解决方案
***
  - 虚拟机装个Windows
  - [crossover](https://www.codeweavers.com/products/)

### 参考资料
***
  - [Wine-wiki](http://wiki.winehq.org/FrontPage)
  - [Wine-Wikipedia](https://zh.wikipedia.org/wiki/Wine)
  - [winetricks-wiki](http://wiki.winehq.org/winetricks_cn)
  - [winetricks-zh](https://github.com/hillwoodroc/winetricks-zh)
  - [Wine-ArchWiki](https://wiki.archlinux.org/index.php/Wine_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29)
