---
title: Linux 测试入站端口连接
published: 2025-08-13
description: 解决服务器端口连通性排查问题，介绍如何使用 netcat 开启本地监听并进行 TCP 入站测试
tags:
  - 网络
  - Linux
category: 教程
draft: false
---
### 安装 netcat （Debian / Ubuntu）

```bash
sudo apt install netcat-openbsd
```

### 在本地机器上监听需要测试的端口

```bash
nc -k -l -p <port> -v
```

### 使用 itdog 的 tcping 测试，或者在另一台机器上使用 telnet 命令测试

```bash
telnet <ip> <port>
```