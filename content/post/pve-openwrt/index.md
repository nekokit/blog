---
title: PVE 中 Openwrt 安装
slug: pve-openwrt
description: 在 Proxmox VE 中安装 openwrt 路由。
image: cover.webp
date: 2022-06-30T19:28:44Z
categories: [Technology]
tags: [PVE, LXC, 虚拟机, openwrt]
---

## 简介

PVE 运行 Openwrt 有两种方式：虚拟机或 LXC 方式。

虚拟机方式能够更好地控制内外网的管理，能方便的直通硬盘和网卡，缺点是效率不如 LXC 方式，故适用于作为控制内外网的网关主路由模式；
LXC 方式资源浪费较小，配置较为麻烦，更适用于作为旁路由模式使用。

## 虚拟机安装

首先准备 openwrt 固件镜像，
可在 [supes.top](https://supes.top/) 进行构建，或使用原版、恩山论坛第三方固件。
推荐使用 `x86-64-generic-squashfs-combined-efi` 固件，拥有以下特性：

- x86_64
- squashfs
- efi

| 变量名          | 示例                                 | 描述               |
| :-------------- | :----------------------------------- | :----------------- |
| `${IMG_NAME}`   | x86-64-generic-squashfs-combined-efi | openwrt 镜像文件名 |
| `${VM_ID}`      | 100                                  | 虚拟机 ID          |
| `${OPENWRT_IP}` | 10.0.0.1                             | openwrt IP         |
| `${MARSK}`      | 255.0.0.0                            | 子网掩码           |

解压得到 `${IMG_NAME}.img` 镜像，将其通过 PVE WebUI 上传到 `Local` - `ISO 镜像` 中。

创建如下虚拟机：

- 不使用 ISO 介质
- q35 机型 UEFI
- 不添加 EFI 磁盘
- 硬盘按需添加
- host CPU
- 512M - 1G 内存

使用命令将 img 镜像转化为硬盘：

```shell
qm importdisk ${VM_ID} /var/lib/vz/template/iso/${IMG_NAME}.img local-lvm
```

在 PVE WebUI 中编辑虚拟机的硬件配置，编辑 `未引用的磁盘` 设置到 `SCSI 总线` 上。
在 PVE WebUI 中添加好需要直通的网卡，这里设置直通两个网口，连接情况如下：

- 虚拟网卡 eth0 -> 虚拟网桥 vmbr0 -> 交换机
- 直通 eth1 -> AP
- 直通 eth2 -> 外部网络

## 配置

通过固件给出的默认 ip 和密码，通过 WebUI 或 ssh 登录到 openwrt 中。
ip 未与本地网络对应的情况，可通过控制台或 ssh 编辑 `/etc/config/network` 修改 ip 地址：

```text
config interface 'lan'
  option device 'br-lan'
  option proto 'static'
  option ipaddr '${OPENWRT_IP}'
  option netmask '${MARSK}'
...
```
