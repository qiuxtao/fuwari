---
title: Linux 手动管理 DNS
published: 2025-04-17
description: 解决 Linux 下 DNS 配置经常被系统服务覆盖的问题
tags:
  - Linux
  - DNS
  - 网络
category: 笔记
draft: false
---
## 检查是否有其他 DNS 管理工具

```bash
ls -l /etc/resolv.conf
```

- 如果输出结果类似 `/etc/resolv.conf -> ../run/resolvconf/resolv.conf`

则需卸载 `resolvconf` 软件包

```bash
sudo apt-get remove resolvconf
```

- 如果输出结果类似 `/etc/resolv.conf -> ../run/systemd/resolve/stub-resolv.conf`

则需停用 `systemd-resolved` 服务

```bash
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
```

移除旧的符号链接：

```bash
sudo rm /etc/resolv.conf
```

## 修改 /etc/resolv.conf 文件

```bash
sudo nano /etc/resolv.conf
```

添加或修改 `nameserver` 行

国际常用 DNS：

```bash
nameserver 8.8.8.8
nameserver 1.1.1.1
```

中国大陆常用 DNS：

```bash
nameserver 119.29.29.29
nameserver 223.6.6.6
```

## 阻止其他服务修改 resolv.conf 文件

为了阻止其他服务修改 `resolv.conf` 文件，可将文件设置为不可修改：

```bash
sudo chattr +i /etc/resolv.conf
```

将文件恢复为可修改：

```bash
sudo chattr -i /etc/resolv.conf
```