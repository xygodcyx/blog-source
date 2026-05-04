---
title: 在ubuntu上部署nginx
tags:
  - 'ubuntu'
categories:
  - ''
date: 2025-11-03 16:05:53
---

## nginx基础命令

### 首先安装nginx

``` bash
sudo apt install nginx
```

### 启动nginx

``` bash
sudo systemctl start nginx
```

### 检查nginx状态

``` bash
sudo systemctl status nginx
```

### 检查80端口是否被占用

``` bash
sudo lsof -i :80
```

### 配置nginx防火墙

``` bash
sudo ufw allow 'Nginx HTTP'
sudo ufw allow 'Nginx HTTPS'
```

### 测试nginx

``` bash
ip a s
```

如果启动成功可以在ip:80端口看到nginx的默认网页

## 部署自己的网页

### 在www目录下创建自己的网页根目录

``` bash
sudo mkdir /var/www/blog
```

### 把hexo播客生成的文件全部复制到www/blog目录中

``` bash
sudo rm -rf /var/www/blog/*
cp -r /root/code/hexo-blog/public/* /var/www/blog/
```

### 确保 nginx 指向正确路径

``` bash
sudo nano /etc/nginx/sites-available/default
```

把root路径改为

``` bash
root /var/www/blog;
```

### 重启nginx

``` bash
sudo systemctl restart nginx
```

## 本地开发与自动化部署

### 用gitbabash生成sbash

``` bash
sbash-keygen -t ed25519 -C "github-actions-deploy@hexo" -f ~/hexo_deploy_key
```

### 将公钥添加到ubuntu服务器上

``` bash
cat ~/hexo_deploy_key.pub | sbash root@your-server-ip "mkdir -p ~/.sbash && cat >> ~/.sbash/authorized_keys && chmod 600 ~/.sbash/authorized_keys && chmod 700 ~/.sbash"
```

输入密码后添加成功

### 测试是否可以免密登录

``` bash
sbash -i ~/hexo_deploy_key root@your-server-ip
```

### 创建github action

确保可以登录后在博客的根目录创建`.github/workflows/deploy.yml`文件，内容为：

``` yml
name: Deploy Hexo Blog

on:
  pubash:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Deploy to server
        uses: appleboy/sbash-action@v0.1.7
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            cd /root/code/hexo-blog
            git reset --hard
            git pull origin main
            bun install
            hexo clean & hexo g
            sudo rm -rf /var/www/blog/*
            sudo cp -r /root/code/hexo-blog/public/* /var/www/blog/
            sudo systemctl reload nginx
```

其中：
SERVER_HOST是ip地址
SERVER_USER是用户名，如root
SERVER_SSH_KEY是第一步生成的私钥（没有.pub后缀）
script部分就是在github action里通过sbash远程连接对ubuntu服务器执行的远程命令，这里的命令可以根据需要自行修改，我使用bun作为包管理器,可以换成npm或pnpm：

``` bash
cd /root/code/hexo-blog
git fetch origin main
git reset --hard origin/main
bun install
bunx hexo clean
bunx hexo g
sudo rm -rf /var/www/blog/*
sudo cp -r /root/code/hexo-blog/public/* /var/www/blog/
sudo systemctl reload nginx
```

### 工作流

在本地开发 -> pubash到github仓库 -> github会启动github action -> 在github action里的系统会使用sbash远程连接我们自己的ubuntu服务器执行script里的命令继而实现自动化
