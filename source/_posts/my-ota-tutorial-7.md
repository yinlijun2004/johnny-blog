---
title: 从零搭建Android OTA系统——部署到阿里云ECS
date: 2018-3-20 19:55:31
tags: [react, nodejs, ECS， nginx, express]
categories: 从零搭建Android OTA系统
---

本文介绍如何将整个ota系统部署到阿里云ECS上去，没有看过正规军是如何部署的，以下只是我的个人经验。

### 创建远程git目录

登陆阿里云ECS，创建一个bare仓库，存放ota_be代码。
```bash 
git init --bare ~/project/ota_server.git
```

创建post-receive文件，捕获git推送操作，克隆一份到服务器本地，作为运行仓库。
```bash
touch ~/project/ota_server.git/hooks/post-receive
```
填入如下内容
```bash
#!/bin/bash -l

GIT_REPO=/home/yinlijun/project/ota_server.git
TMP_GIT_CLONE=/home/yinlijun/project/ota_server

rm -rf ${TMP_GIT_CLONE}
git clone ${GIT_REPO} ${TMP_GIT_CLONE}
```

#### 本地部署
在本地ota目录下，创建.gitignore文件
```bash
echo ota_be_deploy >> .gitignore
```
这个文件是忽略我们要临时发布目录用的。

#### 创建部署脚本

```bash
touch deploy.sh
```
deploy.sh脚本内容：
```bash
# 构建前端代码
cd ota_fe;
yarn build;
cd ../
echo "ota_fe build done..."

# 拷贝前端构建结果到后端代码
cp ota_fe/build/* ota_be/public/ -rf 
echo "copy ota_fe build result done..."

# 从部署服务器clone一份代码
if [ ! -d "ota_be_deploy" ]; then
    git clone yinlijun@www.yinlijun.com:project/ota_server.git ./ota_be_deploy
    echo "clone remote ota_server done..."
fi

# 将后端代码，拷贝到临时目录
cp -rf ota_be/*  ./ota_be_deploy/
echo "copy ota_be done..."

# 提交后端代码到远程master分支
cd ota_be_deploy; 
git add .;
git commit -m "auto deploy server"; 
git push origin master;
cd ../
echo "deploy done..."

```
运行上述脚本，就会将本地的代码，推送到服务器的仓库上去，同时，因为我们hook了推送操作，就会在服务器上更新运行代码。

#### 启动服务

pm2启动服务
```bash
pm2 start ~/project/ota_server/bin/www --name "ota-server"
```

添加nginx代理
```bash
touch /etc/nginx/ota.conf
```
内容如下：
```
server {
    listen: 80;
    server_name ota.yinlijun.com;

    location / {
        proxy_pass http://127.0.0.1:3020;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host
        proxy_cache_bapass $http_upgrade
    }
}
```

重启nginx
```bash
sudo /etc/init.d/nginx restart
```

#### 预览

[http://ota.yinlijun.com/](http://ota.yinlijun.com/)