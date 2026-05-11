---
title: Linux 开启 BBR
published: 2025-04-22
description: 记录在 Linux 开启 BBR 拥塞控制算法以提升网络吞吐量、缓解丢包的操作笔记
tags:
  - BBR
  - Linux
  - 网络
category: 笔记
draft: false
---
## 前言

如果你的 Linux 服务器是普通的跨境线路，可能在晚高峰因为高丢包率的影响很难跑起来带宽，或者受大陆运营商网间结算或者省间结算的影响，跨运营商或者跨省也会有高丢包率，导致跑不起来带宽，可以尝试开启 [BBR](https://github.com/google/bbr) 来缓解以上问题

## 前提条件

你的 Linux 内核版本需要 **4.9 或更高**，BBR 模块自该版本起被包含在主线内核中

查看你的内核版本：

```bash
uname -r
```

确保输出的版本号大于 4.9.0 或更高，如果版本过低，你需要先升级你的 Linux 内核

## 开启命令

```bash
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf # 设置默认的队列规程为 fq
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf # 将 TCP 拥塞控制算法设置为 BBR
sysctl -p # 应用配置
```

## 验证 BBR 是否已启用

- 检查默认 TCP 拥塞控制算法是否为 BBR

```bash
sysctl net.ipv4.tcp_congestion_control
```

预期输出：

`net.ipv4.tcp_congestion_control = bbr`

- 检查默认队列规程是否设置为 fq

```bash
sysctl net.core.default_qdisc
```

预期输出：

`net.core.default_qdisc = fq`

## 如何恢复默认配置

```bash
sudo sed -i '/net.core.default_qdisc=fq/d' /etc/sysctl.conf
sudo sed -i '/net.ipv4.tcp_congestion_control=bbr/d' /etc/sysctl.conf
sysctl -p # 应用配置
```