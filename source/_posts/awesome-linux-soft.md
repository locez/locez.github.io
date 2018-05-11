---
title: 轻量、实用的 Linux 软件
date: 2018-05-01 11:06:39
categories:
 - linux
tags:
 - soft
---

##### AUTHOR: [Locez](http://locez.com)
##### VERSION: 1

### Why
---

自从本人将桌面从 `KDE` 转为 `awesomewm` 后，卸载掉了许多繁重的软件依赖，但同时也带来了一些问题，即再无开箱可用的桌面环境，同时本人也不愿意再为了一些 `KDE` 系的软件再次引入许多 `KDE` 框架依赖，例如 `dolphin`，因此有了本文的记录之用，我会不断更新，方向以日常使用同时尽可能轻量美观为主。当然所有的这些都只是我的主观感受，仅供参考，我不会都一一列举，只会对比以后选择我认为较好的

<!-- more -->
### Browser
---
 - `Firefox`
 - `Chrome`

### Terminal
--- 
 - `xfce4-terminal` 轻量简单实用，可设置透明，隐藏菜单等状态栏后很容易与桌面形成一体

### Input Method
---
 - `fcitx` 输入法框架
 - `fcitx-rime`
 - `fcitx-sunpinyin`

### File Manager
---
 - `pcmanfm-qt` qt版本的 pcmanfm 颜值比较高，装完以后在 `Edit` -> `Preferences` -> `Display` -> `Icon Theme` 中选择 `Breeze` ，效果特别不错

### Text Editor
---
 - `vim` 目前的日常编辑工作都使用 vim 进行，只需要在 pcmanfm-qt 中设置对文本的打开命令即可双击利用 vim 打开该文本,还可在偏好设置中选择打开的终端，本人使用的就是 `xfce4-terminal`

### Picture Manager
---

 - `gthumb` gnome 环境下的相册管理器，非常美观

### Music
---
 - `netease-cloud-music` 网易云音乐，但在 `qtcore 5.7` 之后的版本，桌面歌词变为框框，所以谨慎升级 qt

### Others
---
 - `compton` 窗口合成器，`awesomewm` 只是一个窗口管理器，所以额外增加一个窗口合成器，主要用来实现例如 `xfce4-terminal` 的透明效果


