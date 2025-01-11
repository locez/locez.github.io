---
title: QNAP NAS 使用心得记录
date: 2025-01-11 06:18:20
categories:
  - linux
tags:
  - qnap
  - nas
  - linux
---

##### AUTHOR: [Locez](https://locez.com)
##### VERSION: 1

### 阅读说明
---

本文为 QNAP NAS 系统使用心得记录，主要记录一些使用过程中遇到的问题和解决方法。但是本文不是 QNAP NAS 系统的入门教程，本文的总体方向是容器方向，因此本文不会介绍 QNAP NAS 系统基本安装和使用方法。

**NOTE**：本文大多数 IP 地址都是随便编造的，请酌情思考，复制粘贴

### 基本信息
---

 - 型号：TS-464c2
 - 内存：40G
 - 硬盘：2T SSD + 64T HDD
 - 系统：QuTS hero 5.2
 - 网络：IPv4 + IPv6

### IPv6 配置
---

QNAP 的 IPv6 默认是关闭的，需要在控制台设置中按以下路径开启

![ipv6-setup](https://image.locez.com/blog/qnap-nas-user-experience-diary/ipv6-setup.png)

**NOTE**: 由于安全原因，不建议对外公网开放 IPv6 服务。本文开启 IPv6 主要是为了后文的内网穿透，此处为可选。

### SSH 配置
---

在 QNAP 的设置中心中开启 `SSH` 配置，后续的许多操作需要 `SSH` 才能进行

### 组网穿透
---

#### 基本配置
---

组网直接选择 `tailscale` 提供的服务即可。在上述的过程中，我们去主动打开 `IPv6` 仅仅是为了内网穿透，后续的操作都可以通过 `tailscale` 来进行。

`tailscale` 提供了 3 位免费用户的服务，并且支持 100 台机器，对个人来说是完全够用了。

首先在 QNAP 的 App 中心中选择 `tailscale` 应用并安装，然后登录你的账号，这个时候机器已经加入到组网中

ssh 登录机器，输入以下命令，先找到 `tailscale` 的安装目录，就可以得到查看 `tailscale` 的状态
```bash
ps aux |grep [t]ailscale
23329 admin     85112 S   /share/ZFS530_DATA/.qpkg/Tailscale/tailscaled --port 41641 --statedir=/share/ZFS530_DATA/.qpkg/Tailscale/state --socket=/tmp/tailscale/tailscaled.sock

# 查看 tailscale 的状态，在这里可以看到节点的连接状态，是否是直连和中继
/share/ZFS530_DATA/.qpkg/Tailscale/tailscale status
1.1.1.1   nas-locez            olocez@      linux   -
1.1.1.2   alarmpi              olocez@      linux   offline
1.1.1.3   iphone-14-1          public.locez@ iOS     offline
1.1.1.4   iphone-14            olocez@      iOS     offline
1.1.1.5   oneplus-in2020       olocez@      android offline

# 检查网络状态， 在这里可以看出是否启用 ipv6 等信息
/share/ZFS530_DATA/.qpkg/Tailscale/tailscale netcheck

Report:
        * UDP: true
        * IPv4: yes, 1.1.1.1:1234
        * IPv6: yes, [::2]:5678
        * MappingVariesByDestIP: false
        * HairPinning: false
        * PortMapping:
        * CaptivePortal: false
        * Nearest DERP: Tokyo
        * DERP latency:
                - tok: 118.1ms (Tokyo)
                - hkg: 148.4ms (Hong Kong)
                - sin: 152.6ms (Singapore)
                - lax: 164.5ms (Los Angeles)
                - sea: 176.9ms (Seattle)
                - sfo: 190.5ms (San Francisco)
                - dfw: 198.9ms (Dallas)
                - den: 204.2ms (Denver)
```

`tailscale` 有一个叫做 `subnet` 的功能，这个功能，可以让你通过一台机器部署访问整个内网，例如我以下的例子就是将 `192.168.1.16` 这个 IP 单独暴露出去子网路由了，组网能连进来的时候只能访问这一个 ip，这样的好处是你的 dns 域名设置只需要设置一次，无论你是直连还是组网都只需要用 `xxx.locez.com` 这一个域名就行了

```
/share/ZFS530_DATA/.qpkg/Tailscale/tailscale up --advertise-routes=192.168.1.16/32 --accept-routes
```
然后还需要到 `tailcale` 面板中设置子网路由，这样组网的设备才能访问到子网路由的设备

![subnet-route-setup](https://image.locez.com/blog/qnap-nas-user-experience-diary/subnet-route-setup.png)



#### 共享配置
---

`tailscale` 默认需要登录才能使用，因此免费用户与其它人共享不是特别安全，我这里采取了利用一个公共账号再加 ACL 配置的方式去实现共享。


首先我们创建一个新的邮箱账号（对于国内用户建议是 outlook），例如我的是 `public.locez@outlook.com`，然后对这个账号进行安全配置，比如验证邮箱，两步验证等，后续是要将这个邮箱账号发送给你信任的朋友直接使用的，因此必要的配置还是要做，要始终将邮箱的所有权掌握在自己手中。

然后在 `tailscale` 中登录这个账号，并用主用户邀请这个 `public.locez` 加入 `tailnet`，设置角色为 `member`。

然后我们打开 `tailscale` 的 `Access Controls` 标签页，添加以下配置，这里没用自己新建分组或者标签，而是直接使用默认的分组和标签，这样比较方便。这样我们只需要在这里控制访客用户能访问哪些 IP 和端口，就可以选择共享的时候要暴露什么给别人了，这个方案除了麻烦一点意外，整体的安全性是非常有保障的。

```json
{
	"acls": [
		// 管理员可以访问所有设备
		{
			"action": "accept",
			"users":  ["autogroup:admin"],
			"ports":  ["*:*"],
		},
		// 默认的访客分组只能访问 192.168.1.16 的 443 端口
		{
			"action": "accept",
			"users":  ["autogroup:member"],
			"ports":  ["192.168.1.16:443"],
		},
	],

	"ssh": [
		// Allow all users to SSH into their own devices in check mode.
		// Comment this section out if you want to define specific restrictions.
		{
			"action": "check",
			"src":    ["autogroup:member"],
			"dst":    ["autogroup:self"],
			"users":  ["autogroup:nonroot", "root"],
		},
	],
}
```

**NOTE**： 一个账户通常对应一个 `tailnet`，因此这个账号的设备都在同一个 `tailnet` 中，因此这个账号的设备都可以互相访问，这个账号的设备也可以访问主用户的设备，但是主用户的设备不能访问这个账号的设备。

接下来就可以让朋友登录这个 `public.locez` 的 `tailscale` 账户，然后使用了，在它的机器加入到组网后，记得还需要到 `tailscale` 的面板上去同意


### 配置代理
---

接下来马上就要用到 `docker` 了，众所周知 `docker` 现在访问困难，`dockerd` 无法拉取镜像，所以需要一个代理，网上的方法很多，改配置文件什么的，其实都不靠谱，你会发现根本不生效，还搞得特别复杂。

首先按照下图配置代理地址

![qnap-http-proxy](https://image.locez.com/blog/qnap-nas-user-experience-diary/qnap-http-proxy.png)

接下来 ssh 到机器上

```bash
# 切到管理员账户下，普通用户执行脚本会有限制，权限没做好，部分会成功部分会失败
sudo -i
# 重启一下整个应用，让进程在启动的时候能配到这个代理地址
/etc/init.d/container-station.sh restart
```

**NOTE**: 通过这样配置以后，ssh 登录的终端也会走这个代理，但是具体的容器仍然是无法走的，因此在需要的容器还得额外配置代理地址

### traefik 网关部署
---

现在服务已经可以对外暴露了，但是所有的都是用 IP 访问的，不容易记忆，因此我们需要用域名来访问我们的服务，这里我们使用 `traefik` 来做反向代理。`traefik` 还能用来做证书管理，用 `traefik` 来为我们提供 `HTTPS` 证书，这样我们就不需要自己去管理证书了，并且可以将所有对外提供的服务都加上 `HTTPS`，假设有一天需要对公网暴露，安全性会增加不少。

**NOTE**：在使用 QNAP 的过程中，我尝试使用了 `IPv6`，为了以后能有机会暴露出服务，碰到 `IPv6` 这部分内容可以选择性跳过，不添加 `IPv6` 相关选项即可

#### IPv6
---

QNAP 的 docker 默认不启用 IPv6，甚至系统也没有带相关的内核模块，要启用这块还是有点麻烦。

用以下命令找到 `docker` 的配置文件，如果没找到将 `awk` 过滤删除
```bash
ps aux |grep [/]docker.json | awk -F ' ' '{print $NF}'
/share/ZFS530_DATA/.qpkg/container-station/etc/docker.json
```
编辑这个文件，将 `ip6tables` 改成 `true`

```bash
"ip6tables": true
```

此时如果去创建容器会发现无法启动了，是因为缺失 ip6tables_nat 内核模块，到 [qnap-ip6tables_nat-module](https://github.com/mammo0/qnap-ip6tables_nat-module) 这个项目下载内核模块，然后放到 NAS 中的目录去，比如我放到了 `/share/Container/container-station-data/application/kernel_mods` 目录

接着参考 QNAP 的官方文档，创建一个并开启一个 `autorun.sh` 脚本，内容如下

```bash
#ip6-tables
/sbin/modprobe ip6_tables
/sbin/modprobe nf_nat
/sbin/modprobe xt_MASQUERADE

insmod /share/Container/container-station-data/application/kernel_mods/ip6t_NPT.ko
insmod /share/Container/container-station-data/application/kernel_mods/nf_reject_ipv6.ko
insmod /share/Container/container-station-data/application/kernel_mods/ip6t_REJECT.ko
insmod /share/Container/container-station-data/application/kernel_mods/ip6table_nat.ko
```

在确定在 `控制面板`-`硬件`-`启动时运行用户定义的进程` 是勾选状态以后，重启 NAS，让内核加载就行了

**NOTE**：请务必使用 `uname -a` 确认系统内核版本，和上述仓库提供的版本的差异有多大


#### 创建专用的 traefik 网络
---

参考 [IPv6](https://docs.docker.com/engine/daemon/ipv6/) 创建 IPv6 网络

```bash
docker network create --ipv6 --subnet 2001:db8::/64 traefik
```

#### cloudflare
---
我们使用 `cloudflare` 来进行 `dns challenge` 验证域名，签发证书，需要你将域名的 `nameserver` 指到 cloudflare，并且申请 cloudflare 的 API token，注意限制好 API token 的作用域和权限

#### docker-compose.yml
---

`.env` 文件格式
```bash
CF_DNS_API_TOKEN=
CF_API_EMAIL=
```

给出以下一个完整的 `docker-compose.yml` 文件，利用容器就可以将自己的服务都通过 `traefik` 来代理，并且自动签发证书

```yaml
services:
  traefik:
    image: traefik:v3.2
    container_name: traefik
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      # 用来提供动态配置，从文件载入非 docker provider 的配置
      - /share/Container/apps/traefik/config/:/dynamic/config/  
      # 证书存储路径,要确保 acme.json 的权限是 600
      - /share/Container/apps/traefik/certs:/certs
    environment:
      - TZ=Asia/Shanghai
    # 存储 CF 的 API token
    env_file: /share/Container/apps/traefik/secrets/.env
    # 指定上述创建的网络
    networks:
      - traefik
    extra_hosts:
      # 手动映射该地址
      - "host.docker.internal:192.168.1.16"
    command:
      - "--api.insecure=true"
      - "--api.dashboard=true"
      - "--log.level=INFO"
      - "--entrypoints.websecure.address=:443"
      - "--entrypoints.web.address=:80"

      # 由 docker 动态发现
      - "--providers.docker.endpoint=unix:///var/run/docker.sock"
      - "--providers.docker.exposedByDefault=false"

      # 从文件加载配置
      - "--providers.file.directory=/dynamic/config"

      # 定义一个 resolver，此处用 cloudflare
      - "--certificatesresolvers.myresolver.acme.dnschallenge=true"
      - "--certificatesresolvers.myresolver.acme.dnsChallenge.resolvers=1.1.1.1:53,8.8.8.8:53"
      - "--certificatesresolvers.myresolver.acme.dnsChallenge.provider=cloudflare"
      - "--certificatesresolvers.myresolver.acme.email=olocez@gmail.com"
      - "--certificatesresolvers.myresolver.acme.storage=/certs/acme.json"
      - "--certificatesresolvers.myresolver.acme.dnsChallenge.delayBeforeCheck=30"

    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik-dashboard.entrypoints=websecure"
      - "traefik.http.routers.traefik-dashboard.tls=true"
      - "traefik.http.routers.traefik-dashboard.tls.certresolver=myresolver"
      - "traefik.http.routers.traefik-dashboard.rule=Host(`traefik.example.com`)"
      - "traefik.http.routers.traefik-dashboard.tls.domains[0].main=example.me"
      - "traefik.http.routers.traefik-dashboard.tls.domains[0].sans=*.example.me"
      - "traefik.http.routers.traefik-dashboard.service=dashboard@internal"
      
      - "traefik.http.routers.traefik-dashboard-api.entrypoints=websecure"
      - "traefik.http.routers.traefik-dashboard-api.tls=true"
      - "traefik.http.routers.traefik-dashboard-api.rule=Host(`traefik.example`) && PathPrefix(`/api`)"
      - "traefik.http.routers.traefik-dashboard-api.service=api@internal"
      - "traefik.http.routers.traefik-dashboard-api.tls.certresolver=myresolver"

# 声明是外部网络
networks:
  traefik:
    enable_ipv6: true
    external: true
```

#### emby
---

让我们来看一个 `emby` 利用 traefik 来代理 emby，并且自动签发证书的例子

```yml
services:
  emby:
    image: emby/embyserver
    container_name: embyserver
    # network_mode: host # Enable DLNA and Wake-on-Lan
    networks:
      - traefik
    environment:
      - UID=1000
      - GID=1000
    volumes:
      - /share/Container/apps/emby/config:/config 
      - /share/Data/Videos:/mnt/Videos
    devices:
      - /dev/dri:/dev/dri
    restart: unless-stopped
    labels:
      - "traefik.enable=true"
      # 替换为你的域名
      - "traefik.http.routers.emby.rule=Host(`emby.example.com`)"  
      - "traefik.http.routers.emby.entrypoints=websecure"
      - "traefik.http.routers.emby.tls=true"
      # 配置上面定义的 resolver
      - "traefik.http.routers.emby.tls.certresolver=myresolver"
      # 后端地址
      - "traefik.http.services.emby.loadbalancer.server.port=8096"
networks:
  traefik:
    external: true
```

#### qnap 的管理面板
---

除了代理容器，我们还可以代理 QNAP 本身 5000 端口的面板，这个就是上述的文件动态加载配出来的原因了  `- /share/Container/apps/traefik/config/:/dynamic/config/`

我们在 `config` 文件夹新建一个 `qnap.yml` 文件，输入以下内容，就可以让 QNAP 的面板也被代理了

```yaml
http:
  # Add the router
  routers:
    router0:
      entryPoints:
      - websecure
      service: qnap
      rule: HOST(`qnap.example.com`)
      tls:
        certResolver: myresolver
  # Add the service
  services:
    qnap:
      loadBalancer:
        servers:
        - url: http://192.168.1.16:5000
```

### qBittorrent
---

`qBittorrent` 需要单独拿出来讲以下，因为我们挂种子下载其实是比较危险的一个动作，所以相应的要做一些比其它容器更加安全的配置

#### 用户创建与文件访问
---

先参照下图创建一个 qbittorrent 用户，指定 `UID`，比如我指定了 `10001`，密码随机一个强密码，并且停用此账户

然后给这个用户创建一个文件夹，用于存放下载的文件，然后编辑取消各种权限，只让这个用户只能访问这个文件夹，我这里没有新建，直接用 `Public` 了，之前这个地方就是空的

![create-user](https://image.locez.com/blog/qnap-nas-user-experience-diary/qnap-create-user.png)


![ban-qbittorrent](https://image.locez.com/blog/qnap-nas-user-experience-diary/ban-qbittorrent.png)

#### docker-compose.yml
---
做好上述操作以后，可以利用下面的 `docker-compose.yml` 启动 qBittorrent

```yaml
services:
  qbittorrent:
    image: linuxserver/qbittorrent
    container_name: qbittorrent
    restart: always
    # 配置容器只读，防止容器被恶意修改
    tmpfs:
      - /run:exec
    read_only: true
    networks:
      - qbittorrent_network
    # 使用创建的用户运行容器
    user: "10001:10001"
    environment:
      - PUID=10001
      - PGID=10001
      - TZ=Asia/Shanghai
      - UMASK_SET=022
      - WEBUI_PORT=8088
      - TORRENTING_PORT=6881
    ports:
      - 8088:8088
      - 6881:6881
      - 6881:6881/udp
    volumes:
      - /share/Public/Downloads/qBittorrent/config:/config
      - /share/Public/Downloads/qBittorrent/downloads:/downloads

# 创建一个专用的网络，如果后续要使用traefik代理的话，用文件提供的方式，假设不准备做种，还可以将出口流量用 iptables drop 掉用来保证安全
networks:
  qbittorrent_network:
    driver: bridge
```

### 总结
---

总体折腾下来以后，家里现在有了一整套的服务，也全上了 `HTTPS`，但是购买 QNAP 的 NAS 似乎并没有给我提供太多的便利，他们家应用不好用，高度封装，有一些常用的东西得找半天，或者不生效，对我这种熟悉 Linux 的人反而是一种负担

如果 Linux 系统的一般故障坏了，我是有能力修复的，但是 QNAP 的我绝对修不了，他们的 App 也确实难用，种类、同类太多，甚至不知道该用哪个做哪件事情

但是也有优点，比如不熟悉 ZFS 的情况下，直接使用也是节省了一种学习成本，某些操作不一定需要去敲命令也算是进步吧，不过我本人的应用场景看起来更像是一个带了大硬盘的服务器，我应该还是要选择我熟悉的操作系统才行