---
title: GitHub Actions 自动部署 Fuwari
published: 2026-10-04
description: 通过 GitHub Actions 构建 Fuwari，将静态文件推送到 release 分支，再通过 SSH 同步到 Linux 服务器的网站目录
tags:
  - Fuwari
  - GitHub Actions
  - Linux
  - 自动化运维
category: 教程
draft: false
---

## 前言

Fuwari 是基于 Astro 的静态博客，每次更新文章后，都需要构建并上传静态文件。把这部分工作交给 GitHub Actions，以后写完文章只需要提交并推送源码，就能自动更新服务器上的网站。

这套流程适用于能够通过 SSH 登录、安装 Git 并提供静态网站服务的 Linux 服务器，不依赖特定管理面板。构建在 GitHub Actions 中完成，服务器只负责同步和提供静态文件，无需安装 Node.js 或 pnpm。

本文使用两个分支：`main` 保存博客源码，`release` 保存构建产物。部署过程如下：

```text
推送源码到 main
              ↓
GitHub Actions 安装依赖并构建
              ↓
将 dist 中的文件推送到 release
              ↓
通过 SSH 让服务器同步 release
              ↓
Web 服务器提供最新的静态页面
```

## 准备工作

- 博客源码已上传到 GitHub，默认分支为 `main`。
- Web 服务器已配置静态网站和 HTTPS，并确定网站根目录。
- 服务器已安装 Git，可以访问 GitHub；部署用户可以通过 SSH 登录并读写网站目录。

本文以 `/var/www/blog` 为示例目录。如果已有网站，直接使用现有目录即可。私有仓库还需为服务器配置 GitHub 只读凭据。

## 第一步：初始化服务器上的网站目录

通过 SSH 登录服务器，进入网站目录：

```bash
cd /var/www/blog
```

确认路径正确并完成备份。该目录只保存构建产物，初始化时清理其中的非隐藏文件和目录：

```bash
rm -rf *
```

初始化 Git，并关联远程仓库：

```bash
git init -b release
git remote add origin https://github.com/用户名/仓库名.git
```

远程 `release` 分支会在第一次 Actions 构建成功后创建。

确认 Web 服务器禁止访问 `.git`。Nginx/OpenResty 未配置拦截时，在网站的 `server` 配置中添加：

```nginx
location ^~ /.git/ {
    return 404;
}
```

其他 Web 服务器配置等效规则。访问 `https://域名/.git/config` 应返回 `404`。

## 第二步：配置 SSH 密钥和 GitHub Secrets

### 准备部署用的 SSH 密钥

SSH 密钥配置请参阅[《Linux 设置 SSH 密钥登录》](https://qiuxiaotao.com/posts/linux-ssh-key-setup/)。将对应私钥保存到 `KEY`，并把公钥添加到 `USERNAME` 对应用户的 `~/.ssh/authorized_keys`。

### 添加 Repository secrets

打开 GitHub 仓库的 **Settings → Secrets and variables → Actions**，点击 **New repository secret**，添加：

| 名称 | 值 |
| --- | --- |
| `SERVER_IP` | 服务器 IP 地址 |
| `PORT` | SSH 端口，例如 `22` |
| `USERNAME` | SSH 部署用户名 |
| `KEY` | 部署私钥的完整内容 |

加密私钥再添加 `PASSPHRASE`，填写密钥口令。

### 添加 Repository variable

在 **Settings → Secrets and variables → Actions → Variables** 中点击 **New repository variable**，添加：

| 名称 | 值 |
| --- | --- |
| `DEPLOY_PATH` | `/var/www/blog`，或实际网站根目录 |

## 第三步：添加 GitHub Actions 工作流

在 Fuwari 根目录创建 `.github/workflows/deploy.yml`：

```yaml
name: 自动构建与部署

on:
  push:
    branches:
      - main

permissions:
  contents: write

concurrency:
  group: deploy-production
  cancel-in-progress: false

env:
  TZ: Asia/Shanghai

jobs:
  deploy:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: 检出源码
        uses: actions/checkout@v4

      - name: 安装 pnpm
        uses: pnpm/action-setup@v4

      - name: 安装 Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "22.x"
          cache: pnpm

      - name: 安装依赖
        run: pnpm install --frozen-lockfile

      - name: 生成静态文件和搜索索引
        run: pnpm run build

      - name: 将产物推送到 release 分支
        working-directory: dist
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          git init -b release
          git config user.name 'github-actions[bot]'
          git config user.email '41898282+github-actions[bot]@users.noreply.github.com'
          git add .
          git commit -m "build: 更新静态文件"
          git push --force --quiet "https://x-access-token:${GH_TOKEN}@github.com/${GITHUB_REPOSITORY}.git" HEAD:release

      - name: 同步服务器上的静态文件
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_IP }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.KEY }}
          passphrase: ${{ secrets.PASSPHRASE }}
          port: ${{ secrets.PORT }}
          script: |
            set -eu
            cd "${{ vars.DEPLOY_PATH }}"
            git fetch --depth=1 origin +refs/heads/release:refs/remotes/origin/release
            git reset --hard origin/release
            echo "博客静态文件同步完成"
```

## 第四步：提交并验证自动部署

保存后提交并推送：

```bash
git add .github/workflows/deploy.yml
git commit -m "feat: 配置博客自动部署"
git push origin main
```

在仓库的 **Actions** 中查看“自动构建与部署”的运行结果，确认：

1. GitHub 仓库中出现 `release` 分支，根目录下能看到 `index.html` 和静态资源。
2. 服务器网站目录下出现对应文件。
3. 访问博客域名，确认新页面和站内搜索正常。

以后推送到 `main` 会自动触发部署。

