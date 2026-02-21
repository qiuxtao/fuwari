---
title: Linux 使用 curl 快速查看系统出站公网 IP
published: 2026-02-21
description: Linux 终端下如何使用 curl 命令快速查询服务器的公网出站 IP 地址
tags:
  - Linux
category: 笔记
draft: false
---
在日常的 Linux 服务器管理与运维中，我们经常需要确认当前机器的公网出站 IP。无论是为了配置白名单、检查代理是否生效，还是单纯确认网络环境，以下几个 `curl` 命令都非常实用且高效。

### 国际网络环境

如果你的服务器位于海外，或者你需要测试服务器访问国际互联网时的出口 IP，可以使用 `ipinfo.io`。这个接口不仅会返回你的 IP 地址，通常还会附带 ASN、地理位置和运营商等实用信息：

```
curl ipinfo.io
```

### 大陆网络环境

对于国内的云服务器或本地网络，使用 `ipip.net` 提供的接口响应速度更快，且国内 IP 库定位非常准确：

```
curl myip.ipip.net
```

### 查询 IPv6 地址

随着 IPv6 的逐渐普及，有时我们需要专门确认服务器的 IPv6 出网是否正常。通过给 `curl` 加上 `-6` 参数，强制使用 IPv6 网络去请求 `ip.sb` 即可实现查询：

```
curl ip.sb -6
```