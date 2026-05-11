---
title: Linux 修改主机名
published: 2025-08-11
description: 记录在 Linux 系统中永久修改主机名的常用方法，以及锁定 hosts 文件的实用技巧
tags:
  - Linux
  - hostname
category: 笔记
draft: false
---
## 方法一：hostnamectl 命令

这是最现代、最推荐的方法，尤其适用于基于 `systemd` 的系统

### 设置新主机名

```bash
sudo hostnamectl set-hostname <新主机名>
```

> **说明：** 此命令会同时更新**静态主机名** (保存在 `/etc/hostname`) 和**临时主机名** (内核当前使用的主机名)，效果立即显现。

### 更新 /etc/hosts 文件

为了确保本地解析正确，需手动更新 `hosts` 文件。

```bash
sudo nano /etc/hosts
```

## 方法二：手动修改配置文件

这种方法适用于较旧的系统，或在 `hostnamectl` 命令不可用时使用。

### 修改主机名配置文件

- **对于 Ubuntu/Debian 及其衍生系统：**

```bash
sudo nano /etc/hostname
```

- **对于 CentOS/RHEL 及其衍生系统：**

```bash
sudo nano /etc/sysconfig/network
```

### 更新 /etc/hosts 文件

为了确保本地解析正确，需手动更新 `hosts` 文件。

```bash
sudo nano /etc/hosts
```

### 使更改立即生效

**重要：** 手动修改配置文件后，更改只会在**下次重启**后生效。要让它立即生效 (仅限当前会话)，请执行以下命令：

```bash
sudo hostname -F /etc/hostname
```

> **注意：** 重新建立 SSH 连接，您应该就能看到更新后的主机名了。

## 额外技巧：防止 hosts 文件被意外修改

在某些情况下，其他服务（如云初始化脚本、DHCP 客户端）可能会覆盖您的 `/etc/hosts` 文件。如果您希望将其锁定，可以使用 `chattr` (change attribute) 命令。

### 将文件设置为不可修改 (immutable)

```bash
sudo chattr +i /etc/hosts
```

### 将文件恢复为可修改

```bash
sudo chattr -i /etc/hosts
```