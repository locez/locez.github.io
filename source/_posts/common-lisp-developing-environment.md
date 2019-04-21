title: Common Lisp开发环境
id: 205
categories:
  - Common Lisp
date: 2014-02-06 23:06:54
tags:
  - lisp
---

本文在windows xp下测试通过。

工具：sublime text 2、SBCL(Steel Bank Common Lisp)   【自行搜索下载】

## one

安装SBCL，格式为msi，很容易安装。

![sbcl](https://image.locez.com/blog/common-lisp-developing-environment-00.jpg)

<!--more-->

安装完后可以在菜单找到

![sbclmenu](https://image.locez.com/blog/common-lisp-developing-environment-01.png)

点击执行，可以直接输入lisp代码运行：

![sbclexecute](https://image.locez.com/blog/common-lisp-developing-environment-02.png)

查看环境变量可知已经自动配置好了系统PATH.

## two

解压Sublime Text 2

打开Sublime Text 2

``` bash
Toos
Build System
new Build System...
```

弹出的框中输入以下内容：

``` bash
{
"cmd": ["sbcl", "--script","$file"],
"file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
"selector": "source.lisp"
}
```

如图：

![st2lispbuild](https://image.locez.com/blog/common-lisp-developing-environment-03.png)

然后保存名字为lisp即可。配置到此完成。

# three

测试一下吧！新建一个helloworld.lisp文件

然后键入以下代码：

``` lisp
(defun sayHello()
	(format t "hello world~%"))
(sayHello)
```

![st2lispexcute1](https://image.locez.com/blog/common-lisp-developing-environment-04.png)