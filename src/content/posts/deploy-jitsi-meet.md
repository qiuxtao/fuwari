---
title: 搭建一个开源的视频会议项目——Jitsi Meet
published: 2025-02-24
description: 手把手教你部署 Jitsi Meet
tags:
  - 开源
  - Jitsi
  - 视频会议
category: 教程
draft: false
---
现在免费的视频会议基本上都会限制时长，虽然说腾讯会议和 Microsoft Teams 都支持两人会议不限时，但是他们都会有些缺陷：Teams 是因为服务器不在国内，延迟较大，视频和屏幕共享几乎卡到了不可用的状态；用腾讯会议的时候我的朋友经常会抱怨我共享屏幕的声音太小了，我知道腾讯会议把共享屏幕的声音降低是因为可以防止覆盖掉麦克风的声音，但是为什么不提供一个选项可以开关这个功能呢？

接下来要搭建的这个项目—— Jitsi Meet，就没有这些问题，它开源免费、可以直接在网页加入、支持端到端加密、如果会议里只有两个人还支持p2p（已知的缺点是不支持共享屏幕的时候进行批注）

### 创建并进入安装目录

```
mkdir -p /root/data/docker_data/jitsi
cd /root/data/docker_data/jitsi
```

### 拉去项目文件并进入项目目录

```
git clone https://github.com/jitsi/docker-jitsi-meet
cd docker-jitsi-meet 
```

### 复制示例配置文件并生成密钥

```
cp env.example .env 
./gen-passwords.sh 
```

### 编辑配置文件

```
nano .env
```

### 需要编辑的配置

```
JICOFO_AUTH_LIFETIME=720 hours # 为已认证用户的会话超时值改为720小时
CONFIG=/root/data/docker_data/jitsi/jitsi-meet-cfg # 自定义配置文件目录
HTTP_PORT=8000 # http端口
HTTPS_PORT=8443 # https端口
TZ=Asia/Shanghai # 时区
PUBLIC_URL=https://meeting.qiuxiaotao.cn/ # 公开访问的URL
ENABLE_AUTH=1 # 是否启用身份验证
ENABLE_GUESTS=1 # 是否允许访客
```

如果无法使用 WebSocket，可以设置以下环境变量以回退到 HTTP 轮询和 WebRTC 数据通道

```
ENABLE_SCTP=1
ENABLE_COLIBRI_WEBSOCKET=0
ENABLE_XMPP_WEBSOCKET=0
```

### 启动Jitsi

```
docker compose up -d
```

### 设置反代

```
    location / {
        proxy_pass https://127.0.0.1:8443/;
        proxy_redirect off;
        proxy_set_header Host $host; 
        proxy_set_header X-Forwarded-Proto $scheme; 
        proxy_set_header X-Real-IP $remote_addr; 
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
        proxy_set_header Upgrade $http_upgrade; 
        proxy_set_header Connection $http_connection; 
    }
```

### 自定义内容

```
cd /root/data/docker_data/jitsi/jitsi-meet-cfg/web
```

```
cp interface_config.js custom-interface_config.js
nano custom-interface_config.js
```

**去除 Jitsi 水印**：找到 `SHOW_JITSI_WATERMARK` 将 `true` 改为 `false`  
**修改应用名称：**找到 `APP_NAME` ，自定义名称

```
cp config.js custom-config.js
nano custom-config.js
```

**设置必须输入昵称才能进入会议：**找到 `config.requireDisplayName` 将 `false` 改为 `true`  
**将头像库自定义为** `weavatar.com` **：**找到 `var config = {};` ，在下面加上 `config.gravatarBaseURL = 'https://weavatar.com/avatar/';` 这一行

**修改图标：**将本地 `favicon.svg` 替换容器内的 `favicon.svg`

```
docker cp /root/data/docker_data/jitsi/jitsi-meet-cfg/web/custom/favicon.svg docker-jitsi-meet-web-1:/usr/share/jitsi-meet/images/favicon.svg
```

重启web

```
docker compose restart web
```

### 管理主持人账号

进入容器

```
docker compose exec prosody /bin/bash
```

新增用户

```
prosodyctl --config /config/prosody.cfg.lua register 用户名 meet.jitsi 密码
```

删除用户

```
prosodyctl --config /config/prosody.cfg.lua unregister 用户名 meet.jitsi
```

列出所有已注册用户

```
find /config/data/meet%2ejitsi/accounts -type f -exec basename {} .dat \;
```

部分内容引用自 [Jitsi Meet官方文档](https://jitsi.github.io/handbook/docs/intro)