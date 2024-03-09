---
title: 网络环境配置
slug: network-config
description: 家僮网络基础配置，并记录一部分 hosts 和 nginx 配置。
image: cover.webp
date: 2023-11-18T10:29:17Z
categories: [Technology]
tags: [openwrt, nginx, network]
---

## 路由

### 软件包

| 包名               | 描述     |
| ------------------ | -------- |
| luci-app-dockerman | Docker   |
| luci-nginx         | Nginx    |
| luci-app-openclash | 科学上网 |
| luci-app-vlmcsd    | KMS 激活 |
| luci-app-zerotier  | 异地组网 |

之后关闭 系统 - 启动项 中的 `uhttpd` 自动启动。

### dnsmasq

```text
# 配置内网域名
address=/lsvr.meow/192.168.11.2
address=/svr.local/192.168.12.2
```

### network

```text
# 访问桥接光猫
config interface 'modem'
  option proto 'static'
  option device 'wan'
  option ipaddr '192.168.1.2'
  option netmask '255.255.255.0'
```

### hosts

```hosts
# cloudflare
104.18.11.201        www.kisssub.org
104.18.11.201        www.cangku.moe
104.18.11.201        api.cangku.moe
104.18.11.201        tracker.cangku.moe
104.18.11.201        file.cangku.moe
104.18.11.201        s1.cangku.moe
104.18.11.201        gallery.cangku.moe
104.18.11.201        cangku.moe
104.18.11.201        pic.cangku.moe
104.18.11.201        bdrapid.cangku.moe
104.18.11.201        live.cangku.moe

104.18.21.214        www.kisssub.org
104.18.21.214        www.cangku.moe
104.18.21.214        api.cangku.moe
104.18.21.214        tracker.cangku.moe
104.18.21.214        file.cangku.moe
104.18.21.214        s1.cangku.moe
104.18.21.214        gallery.cangku.moe
104.18.21.214        cangku.moe
104.18.21.214        pic.cangku.moe
104.18.21.214        bdrapid.cangku.moe
104.18.21.214        live.cangku.moe

# themoviedb
54.192.18.58         themoviedb.org
54.192.18.100        themoviedb.org
54.192.18.58         www.themoviedb.org
54.192.18.100        www.themoviedb.org
13.224.167.10        api.themoviedb.org
52.85.151.28         api.themoviedb.org
13.224.167.16        api.themoviedb.org

# thetvdb
108.159.14.107       thetvdb.com
13.225.101.95        thetvdb.com
13.226.125.88        api.thetvdb.com
52.84.98.62          api.thetvdb.com
18.164.130.112       api.thetvdb.com

# weiyun
101.89.39.11         weiyun.com
109.244.157.5        share.weiyun.com
109.244.173.140      share.weiyun.com

# adult
104.21.45.234        www.hacg.mov
104.21.45.234        hacg.mov
#157.240.7.8         blog.reimu.net
#172.67.197.95       blog.reimu.net
31.13.88.26          www.javbus.com

# blocked
127.0.0.1            geo2.adobe.com
127.0.0.1            fpdownload2.macromedia.com
127.0.0.1            fpdownload.macromedia.com
127.0.0.1            macromedia.com
```

## nginx

### meow

`/etc/nginx/conf.d/meow.conf`

```nginx
# Nav
server {
  server_name         nav.meow;
  set $forward_scheme http;
  set $server         $server_addr;
  set $port           5005;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# Proxmox
server {
  server_name         pve.meow;
  set $forward_scheme https;
  set $server         192.168.12.11;
  set $port           8006;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# Clash
server {
  server_name         clash.meow;
  set $forward_scheme http;
  set $server         $server_addr;
  set $port           9090;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# Docker
server {
  server_name         docker.meow;
  set $forward_scheme https;
  set $server         $server_addr;
  set $port           9443;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# Ariang
server {
  server_name         ariang.meow;
  set $forward_scheme http;
  set $server         $server_addr;
  set $port           6880;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# Memos
server {
  server_name         memos.meow;
  set $forward_scheme http;
  set $server         $server_addr;
  set $port           5230;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}
```

### lsvr

`/etc/nginx/conf.d/lsvr.conf`

```nginx
# --- Service ---
# Calibre
server {
  server_name         book.lsvr;
  set $forward_scheme http;
  set $server         192.168.11.2;
  set $port           8083;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# Jellyfin
server {
  server_name         jellyfin.lsvr;
  set $forward_scheme http;
  set $server         192.168.11.2;
  set $port           8096;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# Navidrome
server {
  server_name         music.lsvr;
  set $forward_scheme http;
  set $server         192.168.11.2;
  set $port           4533;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# Rsshub
server {
  server_name         rsshub.lsvr;
  set $forward_scheme http;
  set $server         192.168.11.2;
  set $port           1200;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# Freshrss
server {
  server_name         rss.lsvr;
  set $forward_scheme http;
  set $server         192.168.11.2;
  set $port           1210;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# Speedtest
server {
  server_name         speed.lsvr;
  set $forward_scheme http;
  set $server         192.168.11.2;
  set $port           6300;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}


# --- Download ---
# Alist
server {
  server_name         alist.lsvr;
  set $forward_scheme http;
  set $server         192.168.11.2;
  set $port           5244;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# qBittorrent
server {
  server_name         qbte.lsvr;
  set $forward_scheme http;
  set $server         192.168.11.2;
  set $port           28080;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# AutoBangumi
server {
  server_name         bangumi.lsvr;
  set $forward_scheme http;
  set $server         192.168.11.2;
  set $port           7892;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# BaiduNetDisk
server {
  server_name         baidu.lsvr;
  set $forward_scheme http;
  set $server         192.168.11.2;
  set $port           5800;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}


# --- Database ---
# Mysql
server {
  server_name         mysql.lsvr;
  set $forward_scheme http;
  set $server         192.168.11.2;
  set $port           3306;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# Postgres
server {
  server_name         postgres.lsvr;
  set $forward_scheme http;
  set $server         192.168.11.2;
  set $port           5432;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# Graphile
server {
  server_name         graphile.lsvr;
  set $forward_scheme http;
  set $server         192.168.11.2;
  set $port           5000;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# Adminer
server {
  server_name         database.lsvr;
  set $forward_scheme http;
  set $server         192.168.11.2;
  set $port           6000;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}
```

### proxy_include

`/etc/nginx/conf.d/proxy_include/location.conf`

```nginx
location / {
  add_header       X-Served-By        $host;
  proxy_set_header Host               $host;
  proxy_set_header X-Forwarded-Scheme $scheme;
  proxy_set_header X-Forwarded-Proto  $scheme;
  proxy_set_header X-Forwarded-For    $proxy_add_x_forwarded_for;
  proxy_set_header X-Real-IP          $remote_addr;
  proxy_pass       $forward_scheme://$server:$port$request_uri;
}
```
