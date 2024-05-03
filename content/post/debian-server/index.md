---
title: Debian 服务器搭建
slug: debian-server
description: Debian 系统的安装、配置，安装常用软件，配置 Docker。实现家庭服务器基础环境。
image: cover.webp
date: 2022-05-14T19:50:16+08:00
categories: [Technology]
tags: [server, linux, debian, samba, docker]
---

## 配置 Debian

{{< notice note >}}

Todo:

- 无人值守安装

{{< /notice >}}

使用 netinst ISO 镜像最小化安装 Debian，仅安装 `ssh server` 和 `standard system utilities`。
配置通过以下命令进行。

### 使用 root 修改普通用户 sudo 权限

```shell
su -
apt update
apt install sudo -y
gpasswd -a neko sudo
```

{{< notice info >}}

完成后建议重启系统。

{{< /notice >}}

### 安装基础软件

安装 `vim` `git` `zsh` `curl` `samba`，并安装 `oh-my-zsh` 插件。

```shell
sudo apt install vim git zsh curl samba -y
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

cd ~/.oh-my-zsh/custom/plugins
git clone https://github.com/zsh-users/zsh-autosuggestions.git
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
cd ~

cp ~/.zshrc ~/.zshrc.bak
sed -i 's/ZSH_THEME=\"robbyrussell\"/ZSH_THEME=\"ys\"/g' ~/.zshrc
sed -i 's/plugins=(git)/plugins=(git z sudo extract zsh-autosuggestions zsh-syntax-highlighting)/g' ~/.zshrc
sed -i '/source $ZSH\/oh-my-zsh.sh/a source ~\/.oh-my-zsh\/custom\/plugins\/zsh-syntax-highlighting\/zsh-syntax-highlighting.zsh' ~/.zshrc
sed -i '/source $ZSH\/oh-my-zsh.sh/a source ~\/.oh-my-zsh\/custom\/plugins\/zsh-autosuggestions\/zsh-autosuggestions.zsh' ~/.zshrc

source ~/.zshrc
```

## samba 共享

### 创建 samba 用户

{{< notice note >}}

首先创建 Linux 用户，然后需要转为 Samba 用户才能使用。

{{< /notice >}}

创建 Linux 用户：

```shell
# 添加 Linux 用户
useradd neko
# 设置 Linux 用户的密码
passwd neko
```

将 Linux 用户设置为 samba 用户，并设置 samba 密码

```shell
sudo smbpasswd -a nako
```

删除用户可使用：

```shell
# 删除 Linux 用户
# -r 表示删除主目录
userdel -r ${USER}
# 删除 samba 用户
smbpasswd -x ${USER}
```

### 配置 samba 共享

Samba 的配置文件为 `/etc/samba/smb.conf`，修改此文件增加共享。

```text
[hdd]
    path = /mnt/hdd
    browseable = yes
    available = yes
    writable = yes
    admin users = neko
    valid users = neko
    create mask = 0775
    directory mask = 0775

[ssd]
    path = /mnt/ssd
    browseable = yes
    available = yes
    writable = yes
    admin users = neko
    valid users = neko
    create mask = 0775
    directory mask = 0775
```

修改文件后应测试配置文件是否正确：

```shell
testparm
```

可在 windows 客户机中通过 `Internet选项` 中的 `安全` - `本地 intranet` 添加本地域名，可绕过安全警告。

## Docker

### 安装 Docker

```shell
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release -y

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y

echo '{"registry-mirrors": ["http://hub-mirror.c.163.com"]}' | sudo tee -a /etc/docker/daemon.json
sudo chown neko:docker /etc/docker/daemon.json

sudo gpasswd -a neko docker
newgrp docker
```

管理 Docker 使用 portainer_agent 镜像，通过 Openwrt 中的 portainer-CE 进行连接。

```yaml
version: "3"
services:
  # 容器管理
  agent:
    container_name: portainer_agent
    image: portainer/agent
    restart: unless-stopped
    ports:
      - 9001:9001
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
```

### Docker 容器

详见： [服务器 Docker 镜像](../docker-server)
