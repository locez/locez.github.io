---
title: Rust：在 Windows 上构建并使用 OpenCV
date: 2023-03-19 09:48:27
categories:
 - rust
 - opencv
 - windows
tags:
 - rust
 - opecv
 - windows
---

##### AUTHOR: [Locez](http://locez.com)
##### VERSION: 2
##### UPDATE: 2023-05-10


### 目标
---
本文目标为在 Windows 上编译 OpenCV，并且利用 [opencv-rust](https://crates.io/crates/opencv) 成功在 rust 项目中使用 OpenCV。

### 前置条件
---
 - 已经完成 `rust` 安装
 - 已经完成 `git` 安装

### 安装 C++ 编译工具链
---
此处推荐直接安装 [Visual Studio](https://visualstudio.microsoft.com/zh-hans/)， 可以省去非常多的麻烦， 勾选以下几个选项即可：
 - 使用 C++ 的桌面开发
 - 右侧勾选 C++ ATL 支持，需要注意是否与 MSVC 版本一致（编译 OpenCV 过程中需要）

如果你曾经已经安装过 Visual Studio，此处只需要打开 Installer 修改一下配置即可完成安装。

<!-- more -->

### 编译 OpenCV
---

#### 安装 LLVM
---
LLVM 为 opencv-rust 编译依赖，此处使用 WINGET 进行安装

``` powershell
WinGet install LLVM.LLVM
```
*注意*： 需要将 llvm 的路径(`C:\Program Files\LLVM\bin`)加入到系统环境变量中的 PATH，方便其它应用找到 Clang 等程序，该步骤为必须，此处不再截图添加环境变量




#### 安装 vcpkg
---
我们使用 vcpkg 来编译 OpenCV，如果此前你已经安装过 vcpkg， 那么这一步可以跳过。

克隆仓库，推荐一个硬盘空间大的磁盘
``` powershell
> git clone https://github.com/microsoft/vcpkg
> cd vcpkg
> .\bootstrap-vcpkg.bat
```

*注意*： 将 vcpkg 的路径加入到系统环境变量中的 PATH，方便在调用， 此处就不截图了（这一步是必须的，为了让 rust 更容易找到 OpenCV 的库路径）


#### 编译 OpenCV
---
编译 OpenCV 的过程中会下载大量的包，为了避免在下载包的时候因为网络原因卡住，powershell 可以使用以下语法配置代理

``` powershell
> $env:HTTPS_PROXY="http://ip:port"
> $env:HTTP_PROXY="http://ip:port"
```

使用 vcpkg 编译安装
``` powershell
> vcpkg.exe install opencv4[contrib,nonfree]:x64-windows-static
```

### 在 Rust 中使用
---
在 rust 项目中
``` powershell
> cargo add opencv
> cargo run
```

以下是 example code

``` rust
use opencv::highgui::{imshow, wait_key};
use opencv::imgcodecs::{imread, IMREAD_COLOR};

fn main() {
    let img = imread("1.png", IMREAD_COLOR).unwrap();
    imshow("Display Window", &img).unwrap();
    println!("Hello, world!");
    wait_key(0).unwrap();
}

```






