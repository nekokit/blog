---
title: 网络环境配置
slug: network-config
description: 家庭网络基础配置指南。
image: cover.webp
date: 2024-05-03T09:28:02+08:00
categories: [Technology]
tags: [openwrt, nginx, network]
---

## 路由

### 软件包

| 包名                   | 描述               |
| ---------------------- | ------------------ |
| luci-app-dockerman     | Docker             |
| luci-nginx             | Nginx              |
| luci-app-nginx-manager | Nginx 管理（可选） |
| luci-app-openclash     | 科学上网           |
| luci-app-vlmcsd        | KMS 激活           |
| luci-app-zerotier      | 异地组网           |
| luci-app-nfs           | NFS 服务器         |
| luci-app-vsftpd        | SFTP 服务器        |

之后关闭 `系统` - `启动项` 中的 `uhttpd` 自动启动。

### uci-defaults

```shell
# Beware! This script will be in /rom/etc/uci-defaults/ as part of the image.
# Uncomment lines to apply:
#
wlan_name='Eroneko'
wlan_password=''
#
root_password=''
lan_address='192.168.11.1'
lan_netmask='255.255.255.0'
#
pppoe_username=''
pppoe_password=''

# log potential errors
exec >/tmp/setup.log 2>&1

if [ -n "$root_password" ]; then
  (echo "$root_password"; sleep 1; echo "$root_password") | passwd > /dev/null
fi

# Configure LAN
# More options: https://openwrt.org/docs/guide-user/base-system/basic-networking
if [ -n "$lan_address" ]; then
  uci set network.lan.ipaddr="$lan_address"
  uci set network.lan.netmask="$lan_netmask"
  uci commit network
fi

# Configure WLAN
# More options: https://openwrt.org/docs/guide-user/network/wifi/basic#wi-fi_interfaces
if [ -n "$wlan_name" -a -n "$wlan_password" -a ${#wlan_password} -ge 8 ]; then
  uci set wireless.@wifi-device[0].disabled='0'
  uci set wireless.@wifi-iface[0].disabled='0'
  uci set wireless.@wifi-iface[0].encryption='psk2'
  uci set wireless.@wifi-iface[0].ssid="$wlan_name"
  uci set wireless.@wifi-iface[0].key="$wlan_password"
  uci set wireless.@wifi-device[1].disabled='0'
  uci set wireless.@wifi-iface[1].disabled='0'
  uci set wireless.@wifi-iface[1].encryption='psk2'
  uci set wireless.@wifi-iface[1].ssid="$wlan_name"
  uci set wireless.@wifi-iface[1].key="$wlan_password"
  uci commit wireless
fi

# Configure PPPoE
# More options: https://openwrt.org/docs/guide-user/network/wan/wan_interface_protocols#protocol_pppoe_ppp_over_ethernet
if [ -n "$pppoe_username" -a "$pppoe_password" ]; then
  uci set network.wan.proto=pppoe
  uci set network.wan.username="$pppoe_username"
  uci set network.wan.password="$pppoe_password"
  uci commit network
fi

echo "All done!"
```

### 自定义配置

```shell
# 删除全局 ipv6 前缀
delete_ula_prefix=1

# 配置访问桥接光猫服务器
modem_address='192.168.1.2'
modem_netmask='255.255.255.0'
modem_interface_address='192.168.1.1'

# 启动 ntp 服务器
enable_ntp_server=1

# 启动 KMS 服务器
enable_kms_server=1

# ZeroTier 异地组网
zerotier_network=''
zerotier_address='192.168.192.254'
zerotier_netmask='255.255.255.0'
zerotier_allow_firewall_zones=('lan' 'docker')

# dnsmasq 劫持域名
dnsmasq_addresses=('/meow/192.168.11.1' '/lsvr/192.168.11.1' '/devp/192.168.11.1')

# 配置 Docker
enable_docker=1
docker_allow_firewall_zones=('lan' 'zerotier')
docker_registry_mirror='https://hub-mirror.c.163.com'


if [ "$delete_ula_prefix" -eq 1 ]; then
  echo "删除全局 ipv6 前缀..."
  uci del network.globals.ula_prefix
  uci commit network
fi

if [ -n "$modem_address" -a -n "$modem_interface_address" ]; then
  echo "配置桥接光猫访问..."
  uci set network.modem=interface
  uci set network.modem.proto='static'
  uci set network.modem.device='eth1'
  uci set network.modem.ipaddr="$modem_interface_address"
  uci set network.modem.netmask="$modem_netmask"
  uci set network.modem.defaultroute='0'

  uci add firewall zone
  uci add_list firewall.@zone[1].network='modem'

  uci add network route
  uci set network.@route[-1].interface='lan'
  uci set network.@route[-1].target="$modem_address/32"
  uci set network.@route[-1].gateway="$modem_interface_address"

  uci commit network
  uci commit firewall
fi

if [ "$enable_ntp_server" -eq 1 ]; then
  echo "启用 NTP 服务器..."
  uci set system.ntp.enable_server='1'
  uci set system.ntp.interface='lan'
  uci set system.ntp.use_dhcp='0'
  uci add_list system.ntp.server='ntp1.aliyun.com'
  uci add_list system.ntp.server='ntp.ntsc.ac.cn'
  uci add_list system.ntp.server='cn.ntp.org.cn'
  uci commit system
fi

if [ "$enable_kms_server" -eq 1 ]; then
  echo "启用 KMS 服务器..."
  uci set vlmcsd.config.enabled='1'
  uci set vlmcsd.config.autoactivate='1'
  uci commit vlmcsd
fi

if [ -n "$zerotier_network" ]; then
  echo "配置 ZeroTier 异地组网..."
  uci add_list zerotier.sample_config.join="$zerotier_network"
  uci set zerotier.sample_config.enabled='1'
  uci set zerotier.sample_config.nat='1'
  uci commit zerotier

  # 取得 zerotier 接口
  echo "等待 ZeroTier 虚拟接口..."
  /etc/init.d/zerotier restart
  sleep 5
  zerotier_interface="$(ifconfig | grep '^zt' | awk '{print $1}')"

  uci set network.zerotier=interface
  uci set network.zerotier.device="$zerotier_interface"
  uci set network.zerotier.proto='static'
  uci set network.zerotier.ipaddr="$zerotier_address"
  uci set network.zerotier.netmask="$zerotier_netmask"

  uci add firewall zone
  uci set firewall.@zone[-1].name='zerotier'
  uci set firewall.@zone[-1].input='ACCEPT'
  uci set firewall.@zone[-1].output='ACCEPT'
  uci set firewall.@zone[-1].forward='ACCEPT'
  uci add_list firewall.@zone[-1].network='zerotier'

  for zone in ${zerotier_allow_firewall_zones[@]}
  do
    if [ "$zone" -eq 'lan' ]; then
      uci add firewall forwarding
      uci set firewall.@forwarding[-1].src="$zone"
      uci set firewall.@forwarding[-1].dest='zerotier'
    fi
    uci add firewall forwarding
    uci set firewall.@forwarding[-1].src='zerotier'
    uci set firewall.@forwarding[-1].dest="$zone"
  done

  uci commit network
  uci commit firewall
fi

if [ -n "$dnsmasq_addresses" ]; then
  echo "配置 dnsmasq..."
  for address in ${dnsmasq_addresses[@]}
  do
    uci add_list dhcp.@dnsmasq[0].address="$address"
  done
  uci commit dhcp
fi


if [ "$enable_docker" -eq 1 ]; then
  echo "配置 Docker..."

  for zone in ${docker_allow_firewall_zones[@]}
  do
    if [ "$zone" -eq 'lan' ]; then
      uci add firewall forwarding
      uci set firewall.@forwarding[-1].src="$zone"
      uci set firewall.@forwarding[-1].dest='docker'
    fi
    uci add firewall forwarding
    uci set firewall.@forwarding[-1].src='docker'
    uci set firewall.@forwarding[-1].dest="$zone"
  done
  uci commit firewall

  if [ -n "$docker_registry_mirror" ]; then
    uci del dockerd.globals.registry_mirrors
    uci add_list dockerd.globals.registry_mirrors="$docker_registry_mirror"
    uci commit dockerd
  fi
fi
```

### hosts

```hosts
# weiyun
101.89.39.11         weiyun.com
109.244.157.5        share.weiyun.com
109.244.173.140      share.weiyun.com

# blocked
127.0.0.1            geo2.adobe.com
127.0.0.1            fpdownload2.macromedia.com
127.0.0.1            fpdownload.macromedia.com
127.0.0.1            macromedia.com
```

## Nginx

统一反代配置：

```toml
# --- MEOW ---
[meow]
domain="meow"
forward_server="$server_addr"

[[meow.server]]
name="nav"
forward_scheme="http"
port="5005"

[[meow.server]]
name="clash"
forward_scheme="http"
port="9090"

[[meow.server]]
name="docker"
forward_scheme="https"
port="9443"

[[meow.server]]
name="ariang"
forward_scheme="http"
port="6880"

# --- LSVR ---
[lsvr]
domain="lsvr"
forward_server="192.168.11.2"

# --- Service ---
[[lsvr.server]]
name="book"
forward_scheme="http"
port="8083"

[[lsvr.server]]
name="jellyfin"
forward_scheme="http"
port="8096"

[[lsvr.server]]
name="music"
forward_scheme="http"
port="4533"

[[lsvr.server]]
name="rsshub"
forward_scheme="http"
port="1200"

[[lsvr.server]]
name="rss"
forward_scheme="http"
port="1210"

[[lsvr.server]]
name="alist"
forward_scheme="http"
port="5244"

# --- Download ---
[[lsvr.server]]
name="alist"
forward_scheme="http"
port="5244"

[[lsvr.server]]
name="qbte"
forward_scheme="http"
port="28080"

[[lsvr.server]]
name="bangumi"
forward_scheme="http"
port="7892"

[[lsvr.server]]
name="baidu"
forward_scheme="http"
port="5800"

# --- Database ---
[[lsvr.server]]
name="mysql"
forward_scheme="http"
port="3306"

[[lsvr.server]]
name="postgres"
forward_scheme="http"
port="5432"

[[lsvr.server]]
name="graphile"
forward_scheme="http"
port="5000"

[[lsvr.server]]
name="database"
forward_scheme="http"
port="6000"
```

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

## OpenClash

### 内核

只需要用到 Meta 内核，其他内核不需要下载。

### 配置文件

进入 `配置管理` - `配置文件编辑` 粘贴以下内容到编辑框：

```yaml
proxy-providers:
  Yahaha:
    type: http
    path: "./proxy_provider/config.yaml"
    url: ${订阅地址}
    interval: 86400
    exclude-filter: "2倍|5倍|10倍"
    health-check:
      enable: true
      url: https://www.gstatic.com/generate_204
      interval: 86400
proxy-groups:
  - name: PROXY
    type: url-test
    use:
      - Yahaha
rules:
  - DST-PORT,0-442/444-65535,DIRECT
  - GEOIP,CN,DIRECT
  - MATCH,PROXY
```

### 插件设置

`插件设置` - `模式设置`

- 使用 FakeIp 模式
- 使用 Meta 内核
- 运行模式：Fake-IP（增强）模式
- 取消 `UDP 流量转发`

`插件设置` - `流量控制`

- 取消 `路由本机代理`
- 选定 `实验性：绕过中国大陆 IP`
- 大陆域名 DNS 服务器：使用运营商下发 DNS

`覆写设置` - `DNS 设置`

- 选定 `自定义上游 DNS 服务器`
- 选定 `Fake-IP 持久化`
- 设置自定义上游 DNS 服务器：
  - NameServer：应用运营商下发 DNS 和一个公共 DNS
  - FallBack：全部取消
  - Default-NameServer：全部取消
