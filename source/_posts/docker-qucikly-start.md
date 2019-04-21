---
title: Docker 快速入门
date: 2017-10-05 12:34:24
categories:
 - linux
 - docker
tags:
 - linux
 - docker
---

##### AUTHOR: [Locez](http://locez.com)
##### VERSION: 1

![docker](https://image.locez.com/blog/docker-quickly-start-00.jpg)


### 1 Docker 是什么？
---

***Docker*** 是一个开源的容器引擎，而一个容器其实是一个虚拟化的独立的环境，因此开发者可以将应用打包到这样的一个 docker 容器中，然后发布到任何可以运行 docker 容器的机器中，实现一次打包多处部署，解决了因为环境问题而导致的部署难题。


#### 1.1 容器是什么？
---

与 ***容器*** 对应的一个概念就是 ***镜像***,镜像可以看做我们平时装系统的镜像，里面就是一个运行环境。当然我比较喜欢将镜像比作一个我们面向对象编程中的 **类**，而一个容器就是一个类的 **实例**，因此可以根据一个镜像，创建出很多个容器，每一个容器都是具体的，我们可以在容器上面做出更改，然后再把这个容器打包成一个新的镜像，从而以后可以根据改动后的镜像创建出新的容器。而容器本身可以简单理解为是一个虚拟独立的运行环境，我们要做的是中这个环境中打包我们的应用，以便于再次部署。

<!-- more -->
### 2 安装 Docker
---

Gentoo
```
# emerge --ask docker
```

CentOS
```
# yum install docker-ce
```

Ubuntu
```
# apt-get install docker-ce
```

Arch
```
# pacman -S docker
```


#### 2.1 启动 docker 守护进程
---

```
# systemctl start docker
```


### 3 docker 命令介绍
---

``` 
# docker --help
管理命令:
  container   管理容器
  image       管理镜像
  network     管理网络

命令：
  attach      介入到一个正在运行的容器
  build       根据 Dockerfile 构建一个镜像
  commit      根据容器的更改创建一个新的镜像
  cp          在本地文件系统与容器中复制 文件/文件夹
  create      创建一个新容器
  exec        在容器中执行一条命令
  images      列出镜像
  kill        杀死一个或多个正在运行的容器	
  logs        取得容器的日志
  pause       暂停一个或多个容器的所有进程
  ps          列出所有容器
  pull        拉取一个镜像或仓库到 registry
  push        推送一个镜像或仓库到 registry
  rename      重命名一个容器
  restart     重新启动一个或多个容器
  rm          删除一个或多个容器
  rmi         删除一个或多个镜像
  run         在一个新的容器中执行一条命令
  search      在 Docker Hub 中搜索镜像
  start       启动一个或多个已经停止运行的容器
  stats       显示一个容器的实时资源占用
  stop        停止一个或多个正在运行的容器
  tag         为镜像创建一个新的标签
  top         显示一个容器内的所有进程
  unpause     恢复一个或多个容器内所有被暂停的进程
```

在子命令中还有更多丰富的选项，可以使用 `docker COMMAND --help` 查看。例如：
```

# docker run --help
```

### 4 docker 使用实战
---

接下来，我将利用 docker 部署一个 Nginx 服务器，作为讲解的例子。
要创建一个容器，那么我们必须先要有一个用于创建容器的镜像。
在 docker hub 中查找 nginx 相关镜像。

```
# docker search nginx
NAME                                                   DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                                                  Official build of Nginx.                        6959      [OK]
jwilder/nginx-proxy                                    Automated Nginx reverse proxy for docker c...   1134                 [OK]
richarvey/nginx-php-fpm                                Container running Nginx + PHP-FPM capable ...   452                  [OK]
...
...
```

拉取官方镜像，其中上面的非官方镜像是用户们根据自己的需要制作的镜像，方便大家的使用。

```
# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
afeb2bfd31c0: Pull complete
7ff5d10493db: Downloading [===============>                                   ] 6.651 MB/21.87 MB
d2562f1ae1d0: Download complete
```

在这里顺便说一下，docker 镜像是分层存储的，所以上面的 nginx 镜像有 3 层，另外每根据新的容器制作一个新的镜像以后会给这个镜像加上一层。

接下来直接利用这个镜像启动一个新的容器

```
# docker run --name my-nginx -d -p 8080:80 nginx
```

此时中浏览器中输入：http://localhost:8080/ 即可访问 nginx。
参数解释：

 - `--name` 为容器取一个名字
 - `-p` 参数语法为 `-p host port:container port`; `-p 8080:80` 将主机上的8080端口绑定到容器上的80端口，因此在主机中访问8080端口时其实就是访问 nginx 容器的80端口
 - `-d` 后台运行容器

然后我们来看一个更复杂的例子

```
# docker run --name my-nginx \ 
	-v /host/path/nginx.conf:/etc/nginx/nginx.conf:ro \
	-v /some/html:/usr/share/nginx/html:ro \
	-p 8080:80 \
	-d nginx
```

这个例子多了一个参数 `-v`，这个参数的作用是把本地的文件或者文件夹挂载到容器中，其中最后面的 `ro` 或者 `rw` 控制这个挂载是否可写。

 - `-v` 参数语法为 `-v host dir:container dir[:ro|rw]`

上面的命令将本地文件中的 `nginx.conf` 配置文件挂载到容器，并且将要展示的静态页面也挂载到容器。
***注意***：在容器执行一条命令以后，当这个命令结束运行，容器也会结束运行。

接下来让我们看看 docker 别的命令。
查看所有运行中的容器

```
# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
1fe91b5a4cd4        nginx               "nginx -g 'daemon ..."   39 minutes ago      Up 39 minutes       0.0.0.0:8080->80/tcp   my-nginx
# docker ps -a ### 列出包括未运行的容器
```

查看容器运行日志

```
# docker logs my-nginx
172.17.0.1 - - [05/Oct/2017:07:31:11 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0" "-"
2017/10/05 07:31:11 [error] 7#7: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.17.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8080"
```

重启容器

```
# docker restart my-nginx
my-nginx
```

停止运行一个容器

```
# docker stop my-nginx
my-nginx
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

启动一个已经存在的容器

```
# docker start my-nginx
my-nginx
# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
1fe91b5a4cd4        nginx               "nginx -g 'daemon ..."   50 minutes ago      Up 3 seconds        0.0.0.0:8080->80/tcp   my-nginx
```

直接杀死一个运行中的容器

```
# docker kill my-nginx
```

为容器重命名

```
# docker rename my-nginx new-nginx
```

删除容器

```
# docker rm new-nginx
```

查看本机所有镜像

```
# docker imags
```

使用已经存在的容器创建一个镜像

```
# docker commit -m "nothing changed" my-nginx my-nginx-image
sha256:f02483cdb842a0dc1730fe2c653603fa3271e71c31dbb442caccd7ad64860350
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
my-nginx-image      latest              f02483cdb842        6 seconds ago       108 MB
nginx               latest              da5939581ac8        3 weeks ago         108 MB
```

使用我们自己创建的镜像创建容器

```
# docker run --name test -d -p 8081:80 my-nginx-image
12ba86eabdef0121d875bdb547f1101d4eb54ff7cb36e20214fb1388659af83d
# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
12ba86eabdef        my-nginx-image      "nginx -g 'daemon ..."   7 seconds ago       Up 5 seconds        0.0.0.0:8081->80/tcp   test
299f4ede9e79        nginx               "nginx -g 'daemon ..."   11 minutes ago      Up 11 minutes       0.0.0.0:8080->80/tcp   my-nginx
```

如果我们使用的容器的启动命令是一个 shell ，那么我们可以使用 `attach` 命令来介入这个容器，就跟我们使用 `ssh` 连接一台服务器差不多。

```
# docker attach some-container
```

在创建一个容器时直接启动一个可交互式的 shell， 并且退出后自动删除容器

```
# docker run -it --rm  centos
```


### 总结
---

本文介绍了 docker 的简单用法，同时也给出了一个实际的例子用来展示 docker 命令的各种操作。作为一个快速入门的文章，本文写的相对有点简单，很多概念上的东西还需要求查看别的资料理解，但是看完本文应该就具备使用 docker 的基本能力了。此外，后续的文章应该会写一点 `Dockerfile` 与 `docker-compose` 相关的内容。
