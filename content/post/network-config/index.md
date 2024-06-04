---
title: 网络环境配置
slug: network-config
description: 家庭网络基础配置指南。
image: cover.webp
date: 2024-06-03T21:26:50+08:00
categories: [Technology]
tags: [openwrt, nginx, network]
mermaid: true
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
| luci-app-samba4        | Samba 服务器       |

软件包使用 webui 安装，以方便安装多语言组件。之后关闭 `系统` - `启动项` 中的 `uhttpd` 自动启动。

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
zerotier_address='192.168.192.10'
zerotier_netmask='255.255.255.0'
zerotier_allow_firewall_zones=('lan' 'docker')

# dnsmasq 劫持域名
dnsmasq_addresses=('/server.meow/192.168.10.2' '/pve.meow/192.168.10.3' '/nas.meow/192.168.10.254' '/jellyfin.meow/192.168.10.201' '/lsvr.meow/192.168.10.253' '/nginx.meow/192.168.10.253' '/alist.meow/xiaoya.meow/192.168.10.253' '/rsshub.meow/rss.meow/192.168.10.253' '/book.meow/comic.meow/music.meow/image.meow/192.168.10.253' '/openwrt.meow/nav.meow/clash.meow/docker.meow/file.meow/password.meow/memos.meow/192.168.10.1' '/ariang.meow/qbte.meow/bangumi.meow/baidu.meow/192.168.10.253' '/drawdb.meow/plantuml.meow/jsoncrack.meow/speed.meow/ittools.meow/webp.meow/picsm.meow/pdf.meow/192.168.10.253')


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
  uci add_list system.ntp.server='ntp.aliyun.com'
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

### File Server

`conf.d/file.conf`

```nginx
server {
    server_name  file.meow;
    listen       80;
    listen       [::]:80;
    root /opt/file;
    charset utf-8;

    location / {
        autoindex on;                         # 启用自动首页功能
        autoindex_format html;                # 首页格式为HTML
        autoindex_exact_size off;             # 文件大小自动换算
        autoindex_localtime on;               # 按照服务器时间显示文件时间
        default_type application/octet-stream;# 默认MIME类型设置
        if ($request_filename ~* ^.*?\.(txt|doc|pdf|rar|gz|zip|docx|exe|xlsx|ppt|pptx)$){
            # 当文件格式为上述格式时，将头字段属性Content-Disposition的值设置为"attachment"
            add_header Content-Disposition: 'attachment;';
        }
        sendfile on;                          # 开启零复制文件传输功能
        sendfile_max_chunk 1m;                # 每个sendfile调用的最大传输量为1MB
        tcp_nopush on;                        # 启用最小传输限制功能
        directio 5m;                          # 当文件大于5MB时以直接读取磁盘的方式读取文件
        directio_alignment 4096;              # 与磁盘的文件系统对齐
        output_buffers 4 32k;                 # 文件输出的缓冲区大小为128KB
        max_ranges 4096;                      # 客户端执行范围读取的最大值是4096B
        send_timeout 20s;                     # 客户端引发传输超时时间为20s
        postpone_output 2048;                 # 当缓冲区的数据达到2048B时再向客户端发送
        chunked_transfer_encoding on;         # 启用分块传输标识
    }
}
```

### Password Server

`conf.d/password.conf`

```nginx
server {
  server_name         password.meow;
  set $forward_scheme http;
  set $server         $server_addr;
  set $port           8100;

  listen 443 ssl;
  listen [::]:443 ssl;

  ssl_certificate /etc/nginx/conf.d/password/server.crt;
  ssl_certificate_key /etc/nginx/conf.d/password/server.key;
  ssl_session_cache shared:SSL:32k;
  ssl_session_timeout 5m;
  ssl_ciphers ECDHE-RSA-CNS128-GCM-SHA256:ECDHE:ECDH:CNS:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_prefer_server_ciphers on;

  include conf.d/proxy_include/location.conf;
}
```

### Meow

`conf.d/proxy_include/location.conf`

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

`conf.d/meow.conf`

```nginx
# nav
server {
  server_name         nav.meow;
  set $forward_scheme http;
  set $server         $server_addr;
  set $port           5005;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# clash
server {
  server_name         clash.meow;
  set $forward_scheme http;
  set $server         $server_addr;
  set $port           9090;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# docker
server {
  server_name         docker.meow;
  set $forward_scheme http;
  set $server         $server_addr;
  set $port           9000;

  listen 80;
  listen [::]:80;
  include conf.d/proxy_include/location.conf;
}

# memos
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

## OpenClash

### 内核

只需要用到 Meta 内核，其他内核不需要下载。

### 配置文件

进入 `配置管理` - `配置文件编辑` 粘贴以下内容到编辑框：

```yaml
proxy-providers:
  PQJC:
    type: http
    path: "./proxy_provider/config.yaml"
    url: ${订阅地址}
    interval: 86400
    exclude-filter: "2倍|3倍|5倍|10倍"
    health-check:
      enable: true
      url: https://www.gstatic.com/generate_204
      interval: 86400
  Yahaha:
    type: http
    path: "./proxy_provider/config.yaml"
    url: ${订阅地址}
    interval: 86400
    exclude-filter: "2倍|3倍|5倍|10倍"
    health-check:
      enable: true
      url: https://www.gstatic.com/generate_204
      interval: 86400

proxy-groups:
  - name: PROXY
    type: url-test
    use:
      - PQJC
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
