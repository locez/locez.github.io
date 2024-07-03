---
title: Caddy：构建 HTTPS 反向代理，解决跨域访问问题
date: 2024-07-03 22:53:27
categories:
 - linux
tags:
 - caddy
 - linux
---

##### AUTHOR: [Locez](https://locez.com)
##### VERSION: 1

### 目标
---
本文目标为在 Linux 环境中搭建 Caddy HTTPS 反向代理服务器，并解决前端跨域问题

### 前置条件
---
 - Linux 服务器一个

### 背景
---
在前端项目中，我们经常需要访问一些其它域名提供的 API，这种时候我们会碰到跨域问题 [CORS](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS)。 

跨域机制本质上是一种安全机制，浏览器只能访问它自己的同源资源，比如 `example-a.com` 的 JavaScript 默认情况下是不可以访问 `example-b.com` 的资源的，除非 `example-b.com` 的返回包含了正确的 `CORS` 响应头。更多可以参考 [同源策领 Same-origin policy](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy) 。

### Caddy
---
[Caddy](https://github.com/caddyserver/caddy) 是一个使用 go 语言编写的，开源的 Web 服务器。Caddy 支持大多数 Nginx 支持的功能，它对比 Nginx 具有如下的特点：

 - 配置简单
 - 自动签发 HTTPS

在一些性能要求不是特别高的场景下，Caddy 只需简单的配置便能让我们快速搭建起一个具备 HTTPS 的 Web 服务器，这是它最大的优势。

### Linux 服务器配置
---
本文基于国内的腾讯云服务器进行说明，其中会有一些特色的配置，可选择跳过。

#### 将服务器转换成 Arch Linux（可选）
---
使用 [vps2arch](https://github.com/felixonmars/vps2arch) 将服务器转成 Arch Linux

``` bash
### 使用 root 执行
# wget https://felixc.at/vps2arch
# chmod +x vps2arch
# ./vps2arch
```
在转换成功后，需要修改 DNS 为腾讯云内网 DNS 地址

``` bash
$ cat << EOF > /etc/resolv.conf
nameserver 183.60.83.19
nameserver 183.60.82.98
EOF
```
#### docker
---
首先需要安装 docker

``` bash
### 安装 docker
$ sudo pacman -S docker docker-compose
### 开启启动 docker
$ sudo systemctl enable docker
### 启动 docker
$ sudo systemctl start docker
```

##### 配置 docker 镜像加速（可选）
---
由于国内的环境问题，我们需要加速访问 docker 镜像仓库，下列的地址是腾讯云的内网地址，可以根据情况修改成其它的

``` bash
$ sudo mkdir -p /etc/docker

$ cat << EOF > /etc/docker/daemon.json
{
   "registry-mirrors": [
        "https://mirror.ccs.tencentyun.com"
  ]
}
EOF

$ sudo systemctl restart docker
```

### 配置 Caddy
---
我们先来看一下整个的项目结构
``` bash
$ tree .
.
├── Caddyfile
├── certs
│   ├── proxy.locez.com.key
│   └── proxy.locez.com.pem
└── docker-compose.yml
```
`Caddyfile` 是我们最主要的编辑文件，`certs` 是由于国内的环境原因，我们无法在 `80`, `443` 端口中架设未经备案的服务，因此我们无法通过这种方式去自动续签，一种折中的方式是自备证书，下文会介绍如何使用其它方式去做验证。

`Caddyfile` 的内容如下
``` yml
# 代理的域名
proxy.locez.com:443 {
    # 手动上传的证书
    tls /etc/certs/proxy.locez.com.pem /etc/certs/proxy.locez.com.key
    # 反向代理的地址
    reverse_proxy https://example.com {
        header_up Host {upstream_hostport}
    }

    # 指定一个匹配 OPTIONS 请求方法的匹配器
    @cors_preflight {
        method OPTIONS
        header Origin *
    }

    # 处理这个匹配器，返回 204，并且允许跨域
    handle @cors_preflight {
        respond 204
        header Access-Control-Allow-Origin "*"
        header Access-Control-Allow-Methods "GET, POST, OPTIONS"
        header Access-Control-Allow-Headers "*"
    }

    # 设置允许跨域的响应头
    header {
        Access-Control-Allow-Origin *
        Access-Control-Allow-Methods "GET, POST, OPTIONS"
    }
}
```

`docker-compose.yml` 的配置如下，可以修改为你需要的端口，在使用这份配置之前，我们还需要额外创建一个持久的 `volume` 

``` bash
$ sudo docker volume create proxy_caddy_data
```

``` yml
services:
  caddy:
    image: caddy
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
    ports:
      - "443:443"
      - "443:443/udp"
    volumes:
      - $PWD/certs:/etc/certs
      - $PWD/Caddyfile:/etc/caddy/Caddyfile
      - proxy_caddy_data:/data
      - caddy_config:/config

volumes:
  proxy_caddy_data:
    external: true
  caddy_config:
```
证书部分没有解释如何申请，请自己到各自云厂商签一个 3 个月的免费证书就可以了，至此本文关于如何搭建一个反向代理服务器就讲解完了。

### 扩展
---
Caddy 支持自动续签 HTTPS 证书，签发证书是需要验证域名所有权的，得证明这个域名是你的，证书商才会给你签名认证这个证书。目前我们有以下几种认证方式 [challenge-types](https://letsencrypt.org/docs/challenge-types/)

 - HTTP-01 验证，在 80 端口下在指定的路径下放置一个指定的文件，由证书签发者去访问，可以访问成功，就是验证成功，该方法是最常用的方法，但是由于环境问题，我们无法使用该方法进行验证。
 - DNS-01 验证，通过域名系统进行验证，新增一条解析记录 `_acme-challenge.<YOUR_DOMAIN>`，然后域名签发者去查询这条域名记录为特定的值，就是验证成功
 - TLS-SNI-01验证，已经废除
 - TLS-ALPN-01验证，使用 443 端口进行类似 HTTP-01 一样的验证，在国内依然不可用

综上，我们在国内的环境下能用的只有 `DNS-01` 这种方式，但是 `DNS-01` 这种方式是需要 DNS 厂商支持的，DNS 厂商有对应的 API 修改记录值才可以。目前绝大多数厂商都支持，但是仍然有一个问题，那就是默认的 Caddy 是不支持这样的厂商特定的 API 请求的，我们需要自行编译对应的模块，让它支持对应的  DNS Challenge。

我们通过 [xcaddy](https://github.com/caddyserver/xcaddy) 来帮助我们完成这个工作
``` bash
$ go install github.com/caddyserver/xcaddy/cmd/xcaddy@latest
```
这里以 [阿里 DNS 模块](https://github.com/caddy-dns/alidns) 为例，构建支持的版本
``` bash
$ xcaddy build --with github.com/caddy-dns/alidns
```
当然也可以选择在 [caddyserver](https://caddyserver.com/download) 直接下载对应的插件版本，或者使用 go 容器最小化构建一个 caddy 版本

首先我们需要一个 `.env` 文件来存储我们的密钥
```
DOMAIN=https://example.locez.com, https://example-1.locez.com
LOG_FILE=access.log
EMAIL=locez@locez.com
ALIYUN_ACCESS_KEY_ID=
ALIYUN_ACCESS_KEY_SECRET=

```
我们这次的 `Caddyfile` 会如下这样子

``` conf
{$DOMAIN}:443 {
        log {
                level INFO
                output file {$LOG_FILE} {
                        roll_size 10MB
                        roll_keep 10
                }
        }

        # 使用 ACME DNS-01 challenge 获取证书
        tls {
                dns alidns {
                        access_key_id {$ALIYUN_ACCESS_KEY_ID}
                        access_key_secret {$ALIYUN_ACCESS_KEY_SECRET}
                }
        }

        encode gzip

        reverse_proxy example.com:80 {
                header_up X-Real-IP {remote_host}
        }
}
```

`docker-compose.yml`

``` yml
  caddy:
    image: caddy:2
    container_name: caddy
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./caddy:/usr/bin/caddy  # 上述步骤自定义构建的 caddy
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./caddy-config:/config
      - ./caddy-data:/data
    environment:
      DOMAIN: "${DOMAIN}"  # 域名
      EMAIL: "${EMAIL}"        # ACME 认证使用的邮箱
      LOG_FILE: "${LOG_FILE}"
      ALIYUN_ACCESS_KEY_ID: "${ALIYUN_ACCESS_KEY_ID}"
      ALIYUN_ACCESS_KEY_SECRET: "${ALIYUN_ACCESS_KEY_SECRET}"
```




