---
title: 服务器 Docker 镜像
slug: docker-server
description: 精选一些常用 Docker 镜像并提供 docker-compose 配置。
image: cover.webp
date: 2024-06-03T21:59:06+08:00
categories: [Technology]
tags: [server, docker]
---

## 安装 Docker

安装 Docker 详见： [Debian 服务器搭建](../debian-server#docker)

## 管理型容器

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

### PortainerAgent

Portainer 可通过 Agent 对其他 Docker 环境进行管理。

```shell
docker run -d \
    -p 9001:9001 \
    --name portainer_agent \
    --restart unless-stopped \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /var/lib/docker/volumes:/var/lib/docker/volumes \
    portainer/agent
```

## 轻量静态型容器

轻量型容器或管理容器可直接部署在 openwrt 中。

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

### VaultWarden - 密码管理

使用 Bitwarden API 的轻量密码管理工具。

```shell
docker run -d \
    --name vaultwarden \
    --restart unless-stopped \
    -p 8100:80 \
    -v /opt/password:/data \
    vaultwarden/server
```

## 服务器容器组织

使用 docker compose 进行容器编排，使用 docker 映射 nfs 卷。

### 下载型容器

```yaml
services:
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
      - nas_downloads:/downloads
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK_SET=022
      - RPC_SECRET=123456
      - RPC_PORT=6800
      - LISTEN_PORT=6888
      - DISK_CACHE=64M
      - IPV6_MODE=true
      - UPDATE_TRACKERS=true
      - TZ=Asia/Shanghai
    logging:
      driver: json-file
      options:
        max-size: 1m

  ariang:
    container_name: ariang
    image: p3terx/ariang
    restart: unless-stopped
    command: --port 6880 --ipv6
    depends_on:
      - aria2
    ports:
      - 6880:6880
    logging:
      driver: json-file
      options:
        max-size: 1m

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
      - nas_downloads:/downloads
      - nas_bangumi:/bangumi
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK_SET=022
      - TZ=Asia/Shanghai
      - QBT_WEBUI_PORT=28080

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
    volumes:
      - ./autobangumi/config:/app/config
      - ./autobangumi/data:/app/data

volumes:
  nas_downloads:
    driver_opts:
      type: "nfs"
      o: "addr=192.168.10.254,nolock,soft,rw"
      device: ":/mnt/buffer/downloads"
  nas_bangumi:
    driver_opts:
      type: "nfs"
      o: "addr=192.168.10.254,nolock,soft,rw"
      device: ":/mnt/buffer/bangumi"
```

因百度网盘流氓行为，单独控制启停。

```yaml
services:
  BaiduNetdisk:
    container_name: baidunetdisk
    image: johngong/baidunetdisk
    restart: unless-stopped
    ports:
      - 5800:5800
      - 5900:5900
    volumes:
      - ./netdisk:/config
      - nas_downloads:/downloads
    environment:
      - USER_ID=1000
      - GROUP_ID=1000

volumes:
  nas_downloads:
    driver_opts:
      type: "nfs"
      o: "addr=192.168.10.254,nolock,soft,rw"
      device: ":/mnt/buffer/downloads"
```

### 资源型容器

```yaml
services:
  alist:
    container_name: alist
    image: xhofe/alist
    restart: unless-stopped
    ports:
      - 5244:5244
    volumes:
      - ./alist:/opt/alist/data
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=022

  xiaoya:
    container_name: xiaoya
    image: xiaoyaliu/alist
    restart: unless-stopped
    ports:
      - 5678:80
    volumes:
      - ./xiaoya:/data
```

### 媒体容器

```yaml
services:
  calibreweb:
    container_name: calibre
    image: linuxserver/calibre-web
    restart: unless-stopped
    ports:
      - 8083:8083
    volumes:
      - ./calibre:/config
      - nas_book:/books
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai

  navidrome:
    container_name: navidrome
    image: deluan/navidrome
    restart: unless-stopped
    ports:
      - 4533:4533
    volumes:
      - ./navidrome:/data
      - nas_music:/music:ro
    environment:
      ND_SCANSCHEDULE: 1h
      ND_LOGLEVEL: info
      ND_BASEURL: ""

  komga:
    container_name: komga
    image: gotson/komga
    restart: unless-stopped
    ports:
      - 25600:25600
    volumes:
      - ./komga:/config
      - nas_comic:/data
      - /etc/timezone:/etc/timezone:ro
    user: 1000:1000

  piwigo:
    container_name: piwigo
    image: linuxserver/piwigo
    restart: unless-stopped
    ports:
      - 8001:80
    volumes:
      - ./piwigo:/config
      - nas_image:/gallery
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai

  mediadb:
    container_name: mediadb
    image: mariadb
    restart: unless-stopped
    ports:
      - 8000:3306
    volumes:
      - ./mediadb/log:/var/log/mysql
      - ./mediadb/data:/var/lib/mysql
      - ./mediadb/conf.d:/etc/mysql/conf.d
      - /etc/localtime:/etc/localtime:ro
    environment:
      - MARIADB_ROOT_PASSWORD=Media@Meow

volumes:
  nas_book:
    driver_opts:
      type: "nfs"
      o: "addr=192.168.10.254,nolock,soft,rw"
      device: ":/mnt/hddpool/book"
  nas_music:
    driver_opts:
      type: "nfs"
      o: "addr=192.168.10.254,nolock,soft,rw"
      device: ":/mnt/hddpool/music"
  nas_comic:
    driver_opts:
      type: "nfs"
      o: "addr=192.168.10.254,nolock,soft,rw"
      device: ":/mnt/hddpool/comic"
  nas_image:
    driver_opts:
      type: "nfs"
      o: "addr=192.168.10.254,nolock,soft,rw"
      device: ":/mnt/hddpool/image"
```

### 工具型容器

```yaml
services:
  #speedtest:
  #  container_name: speedtest
  #  image: adolfintel/speedtest
  #  restart: unless-stopped
  #  ports:
  #    - 8110:80
  #  environment:
  #    MODE: standalone

  ittools:
    container_name: ittools
    image: corentinth/it-tools
    restart: unless-stopped
    ports:
      - 8120:80

  webp2jpg:
    container_name: webp2jpg
    image: wbsu2003/webp2jpg-online
    restart: unless-stopped
    ports:
      - 8130:80

  picsmaller:
    container_name: picsmaller
    image: vimiix/pic-smaller
    restart: unless-stopped
    ports:
      - 8140:3000

  spdf:
    container_name: spdf
    image: frooodle/s-pdf
    restart: unless-stopped
    ports:
      - 8150:8080
    volumes:
      - ./spdf/trainingData:/usr/share/tessdata
      - ./spdf/extraConfigs:/configs
    environment:
      - DOCKER_ENABLE_SECURITY=false
      - INSTALL_BOOK_AND_ADVANCED_HTML_OPS=false
      - LANGS=zh_CN

  drawdb:
    container_name: drawdb
    build: drawdb
    restart: unless-stopped
    ports:
      - 8160:80
```
