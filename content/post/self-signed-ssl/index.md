---
title: 自签 SSL 证书
slug: self-signed-ssl
description: 使用 openssl 生成自签证书并使客户端信任。
image: cover.webp
date: 2024-06-03T22:33:15+08:00
categories: [Technology]
tags: [server, ssl]
---

## 开始之前

由于部分应用强制要求使用 TLS，找了一下自签内网域名证书的办法，
这里以 openwrt 为例，先安装 `openssl-util`，找一个工作目录，如：`/root/ssl/`。

## 创建根证书

```shell
openssl req \
 -x509 \
 -nodes \
 -days 3650 \
 -newkey rsa:2048 \
 -subj "/C=CN/ST=Beijing/L=Beijing/O=Meow" \
 -keyout root_ca.key \
 -out root_ca.crt \
 -reqexts v3_req \
 -extensions v3_ca
```

## 创建域名证书

创建文件夹

```shell
mkdir ${service}
```

### 创建域名私钥

```shell
openssl genrsa -out ${service}/server.key 2048
```

### 域名申请文件

```shell
openssl req \
  -new \
  -key  ${service}/server.key \
  -subj "/C=CN/ST=Beijing/L=Beijing/O=Meow/CN=${service}.meow" \
  -sha256 \
  -out  ${service}/server.csr
```

### 申请配置文件

```shell
vim config.ext
```

```ext
[ req ]
default_bits        = 1024
distinguished_name  = req_distinguished_name
req_extensions      = san
extensions          = san
[ req_distinguished_name ]
countryName         = CN
stateOrProvinceName = Definesys
localityName        = Definesys
organizationName    = Definesys
[ SAN ]
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[ alt_names ]
IP.1 = ${service_ip}
DNS.1 = ${service}.meow
```

### 申请证书

```shell
openssl x509 \
  -req \
  -days 3650 \
  -in ${service}/server.csr \
  -CA root_ca.crt \
  -CAkey root_ca.key \
  -CAcreateserial \
  -sha256 \
  -out ${service}/server.crt \
  -extfile config.ext \
  -extensions SAN
```

## Nginx SSL 配置

拷贝证书到 nginx 配置目录：

```shell
mv ${service} /etc/nginx/conf.d/${service}
```

应用于 luci-nginx 时应注意取消配置自带的自签证书：

```uci
config server '_lan'
    list listen '443 ssl default_server'
    list listen '[::]:443 ssl default_server'
    option server_name '_lan'
    list include 'restrict_locally'
    list include 'conf.d/*.locations'
    option ssl_certificate '/etc/nginx/conf.d/server.crt'
    option ssl_certificate_key '/etc/nginx/conf.d/server.key'
    option ssl_session_cache 'shared:SSL:32k'
    option ssl_session_timeout '64m'
    option access_log 'off; # logd openwrt'
    option ssl_ciphers 'ECDHE-RSA-CNS128-GCM-SHA256:ECDHE:ECDH:CNS:HIGH:!NULL:!aNULL:!MD5:!ADH:!R
    option ssl_protocols 'TLSv1 TLSv1.1 TLSv1.2'
    option ssl_prefer_server_ciphers 'on'
```

其他服务 nginx 配置：

```nginx
server {
  server_name         ${service}.meow;
  set $forward_scheme http;
  set $server         $server_addr;
  set $port           ${service_port};

  listen 443 ssl;
  listen [::]:443 ssl;

  ssl_certificate /etc/nginx/conf.d/${service}/server.crt;
  ssl_certificate_key /etc/nginx/conf.d/${service}/server.key;
  ssl_session_cache shared:SSL:32k;
  ssl_session_timeout 5m;
  ssl_ciphers ECDHE-RSA-CNS128-GCM-SHA256:ECDHE:ECDH:CNS:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_prefer_server_ciphers on;

  include conf.d/proxy_include/location.conf;
}
```
