---
title: Linux 设置时间同步
published: 2026-05-15
description: 记录查看时间同步状态、清理其他时间同步服务、安装启用 systemd-timesyncd 并修改 NTP 服务器地址的常用步骤
tags:
  - Linux
  - NTP
category: 笔记
draft: false
---
## 查看当前是否设置了时间同步

先查看系统时间、时区和 NTP 状态：

```bash
timedatectl
```

重点看这几行：

```text
System clock synchronized: yes
NTP service: active
RTC in local TZ: no
```

- `System clock synchronized: yes` 表示系统时钟已经同步。
- `NTP service: active` 表示已经启用时间同步服务。
- `RTC in local TZ: no` 通常保持默认即可，表示硬件时钟使用 UTC。

如果想确认当前是否由 `systemd-timesyncd` 负责同步：

```bash
systemctl status systemd-timesyncd
```

## 卸载其他时间同步服务

同一台机器上建议只保留一个时间同步服务，否则容易出现互相抢占或配置不生效的问题。

卸载常见的其他时间同步服务：

```bash
sudo apt purge -y chrony ntp ntpsec openntpd
sudo apt autoremove -y
```

如果某些软件包没有安装，提示找不到或未安装可以忽略。

## 安装并启用 systemd-timesyncd

安装 `systemd-timesyncd`：

```bash
sudo apt update
sudo apt install -y systemd-timesyncd
```

开启系统 NTP 同步开关：

```bash
sudo timedatectl set-ntp true
```

确认服务状态：

```bash
systemctl status systemd-timesyncd
timedatectl
```

在 Debian/Ubuntu 上，`systemd-timesyncd` 安装后通常会自动启动。如果检查发现服务没有启动，再执行：

```bash
sudo systemctl enable --now systemd-timesyncd
```

## 修改 NTP 服务器地址

编辑 `systemd-timesyncd` 配置文件：

```bash
sudo nano /etc/systemd/timesyncd.conf
```

找到或添加 `[Time]` 配置段，例如：

```ini
[Time]
NTP=ntp.aliyun.com time1.cloud.tencent.com cn.pool.ntp.org
FallbackNTP=pool.ntp.org time.cloudflare.com
```

说明：

- `NTP=` 是优先使用的时间服务器，可以写多个，使用空格分隔。
- `FallbackNTP=` 是备用时间服务器，当主服务器不可用时使用。

保存后重启服务：

```bash
sudo systemctl restart systemd-timesyncd
```

再次查看同步状态：

```bash
timedatectl
```
