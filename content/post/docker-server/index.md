---
title: 服务器 Docker 镜像
slug: docker-server
description: 精选一些常用 Docker 镜像并提供 docker-compose 配置。
image: cover.webp
date: 2022-12-26T19:23:46Z
categories: [Technology]
tags: [server, docker]
---

## 安装 Docker

安装 Docker 详见： [Debian 服务器搭建](../debian-server#docker)

## 轻量静态型容器

轻量型容器或管理容器可直接部署在 openwrt 中。

### Portainer - Docker 管理

Portainer 是一个 Docker 的网页管理工具，
可通过网页访问管理页面控制容器、镜像的启停、更新、创建删除等操作。

```shell
docker run -d \
    --name portainer \
    --restart unless-stopped \
    -p 9443:9443 \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce
```

#### PortainerAgent

Portainer 可通过 Agent 对其他 Docker 环境进行管理。

```shell
docker run -d \
    -p 9001:9001 \
    --name portainer_agent \
    --restart=unless-stopped \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /var/lib/docker/volumes:/var/lib/docker/volumes \
    portainer/agent
```

### Flare - 导航

轻量、快速、美观的个人导航页面。
无任何数据库依赖，
支持在线编辑。

```shell
docker run -d \
    --name flare \
    --restart unless-stopped \
    -p 5005:5005 \
    -v /opt/volume/flare:/app \
    soulteary/flare
```

### Memos - 临时记事

隐私优先的轻量级笔记服务，
轻松捕捉并分享精彩想法。

```shell
docker run -d \
    --name memos \
    --restart unless-stopped \
    -p 5230:5230 \
    -v /opt/volume/memos:/var/opt/memos \
    neosmemo/memos
```

### AriaNG - Aria2 WebUI

AriaNg 是一个让 aria2 更容易使用的现代 Web 前端。
使用纯 html & javascript 开发，
使用响应式布局，支持各种计算机或移动设备。

```shell
docker run -d \
    --name ariang \
    --restart unless-stopped \
    -p 6880:6880 \
    --log-opt max-size=1m \
    p3terx/ariang
```

## 下载型容器

使用 docker compose 进行容器编排。

```shell
mkdir -p ~/docker/download
cd ~/docker/download
mkdir -p alist aria2 qbittorrent autobangumi/config autobangumi/data baidu
vim docker-compose.yml
```

```yml
version: "3"
services:
  # 网络存储聚合
  alist:
    container_name: alist
    image: xhofe/alist
    restart: unless-stopped
    ports:
      - 5244:5244
    volumes:
      - ./alist:/opt/alist/data

  # Aria2 下载器
  aria2:
    container_name: aria2
    image: p3terx/aria2-pro
    restart: unless-stopped
    ports:
      - 6800:6800
      - 6888:6888
      - 6888:6888/udp
    volumes:
      - ./aria2:/config
      - /mnt/hdd/downloads:/downloads
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK_SET=022
      - RPC_SECRET=Aria2
      - RPC_PORT=6800
      - LISTEN_PORT=6888
      - DISK_CACHE=64M
      - IPV6_MODE=false
      - UPDATE_TRACKERS=true
      - CUSTOM_TRACKER_URL=
      - TZ=Asia/Shanghai
    logging:
      driver: json-file
      options:
        max-size: 1m

  # qBittorrent 增强版
  qbittorrent:
    container_name: qbittorrent
    image: p3terx/qbittorrent-enhanced
    restart: unless-stopped
    ports:
      - 28080:28080
      - 28888:28888
      - 28888:28888/udp
    volumes:
      - ./qbittorrent:/qBittorrent
      - /mnt/hdd/downloads:/downloads
      - /mnt/hdd/jellyfin/bangumi:/bangumi
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK_SET=022
      - TZ=Asia/Shanghai
      - QBT_WEBUI_PORT=28080

  # 自动追番
  autobangumi:
    container_name: autobangumi
    image: estrellaxd/auto_bangumi
    restart: unless-stopped
    depends_on:
      - qbittorrent
    ports:
      - 7892:7892
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
      - AB_DOWNLOADER_HOST=qbte.lsvr
    volumes:
      - ./autobangumi/config:/app/config
      - ./autobangumi/data:/app/data

  # 百度网盘 的 VNC 客户端
  BaiduNetdisk:
    container_name: baidunetdisk
    image: johngong/baidunetdisk
    restart: unless-stopped
    ports:
      - 5800:5800
      - 5900:5900
    volumes:
      - ./baidu:/config
      - /mnt/hdd/downloads:/downloads
    environment:
      - USER_ID=1000
      - GROUP_ID=1000
```

## 服务型容器

使用 docker compose 进行容器编排。

```shell
mkdir -p ~/docker/service
cd ~/docker/service
mkdir -p calibre jellyfin/config jellyfin/cache navidrome freshrss/extensions freshrss/data
vim docker-compose.yml
```

```yml
version: "3"
services:
  # 电子书库
  calibreWeb:
    container_name: calibre
    image: linuxserver/calibre-web
    restart: unless-stopped
    ports:
      - 8083:8083
    volumes:
      - ./calibre:/config
      - /mnt/ssd/book:/books
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai

  # 流媒体服务器
  jellyfin:
    container_name: jellyfin
    image: nyanmisaka/jellyfin
    user: 1000:1000
    restart: unless-stopped
    ports:
      - 8096:8096
    volumes:
      - ./jellyfin/config:/config
      - ./jellyfin/cache:/cache
      - /mnt/hdd/jellyfin:/media
    devices:
      - /dev/dri/renderD128:/dev/dri/renderD128
      - /dev/dri/card0:/dev/dri/card0

  # 音乐服务器
  navidrome:
    container_name: navidrome
    image: deluan/navidrome
    restart: unless-stopped
    ports:
      - 4533:4533
    environment:
      ND_SCANSCHEDULE: 1h
      ND_LOGLEVEL: info
      ND_BASEURL: ""
    volumes:
      - ./navidrome:/data
      - /mnt/ssd/music:/music:ro

  # Rss 订阅
  rsshub:
    container_name: rsshub
    image: diygod/rsshub
    restart: unless-stopped
    ports:
      - 1200:1200

  # Rss 阅读
  freshrss:
    container_name: freshrss
    image: freshrss/freshrss
    restart: unless-stopped
    ports:
      - 1210:80
    depends_on:
      - rsshub
    logging:
      options:
        max-size: 10m
    volumes:
      - ./freshrss/data:/var/www/FreshRSS/data
      - ./freshrss/extensions:/var/www/FreshRSS/extensions
    environment:
      TZ: Asia/Shanghai
      CRON_MIN: "1,15,31"
      TRUSTED_PROXY: 172.16.0.1/12 192.168.0.1/16

  # 内网测速
  # speedtest:
  #   container_name: speedtest
  #   image: adolfintel/speedtest
  #   restart: unless-stopped
  #   ports:
  #     - 6300:80
```

## 数据库容器

使用 docker compose 进行容器编排。

```shell
mkdir -p ~/docker/database
cd ~/docker/database
mkdir -p mysql postgres
vim docker-compose.yml
```

```yml
version: "3"
services:
  # mysql 数据库
  mysql:
    container_name: mysql
    image: mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: unless-stopped
    ports:
      - 3306:3306
    volumes:
      - ./mysql:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: Ero@Mysql

  # postgres 数据库
  postgres:
    container_name: postgres
    image: postgres
    restart: unless-stopped
    ports:
      - 5432:5432
    volumes:
      - ./postgres:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: Ero@Postgres

  # Graphile
  postgraphile:
    container_name: postgraphile
    image: graphile/postgraphile
    restart: unless-stopped
    ports:
      - 5000:5000
    depends_on:
      - postgres
    command: --connection postgres://study:123456@postgres/pg_test --schema public --watch

  # web 数据库管理
  adminer:
    container_name: adminer
    image: adminer
    restart: unless-stopped
    ports:
      - 6000:8080
```
