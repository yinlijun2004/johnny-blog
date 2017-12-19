---
title: 部署HEXO博客到阿里云ECS
date: 2017-12-18 17:46:56
tags: [hexo, ECS, git]
---

本文假设你具备如下条件：
- 熟悉hexo操作
- 熟悉linux基本指令
- 熟悉express(nodejs)框架
- 在github上已经部署了hexo博客

## ECS准备
### 购买ECS
前往阿里云购买[ECS](https://ecs-buy.aliyun.com)，根据个人需要和财务状况，选择对应规模的ECS。
我购买的是这个配置，费用是330.00¥。
![img](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/a6b1134939a35cdf34dab8f9d564c777b7aafbb5518fae80c8450e0ba8d5280bf07d7e677b3b3c2028e2f7a5c778e5a8?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=1GE9BH_P%29%600QGAZLQY6E%7D0G.png&size=1024)

### 配置ECS
购买之后，可以远程登录ECS实例，可以选择网页登录，即点击上图的远程连接，此时需要输入远程连接密码。
![img](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/8f23cd09eff9be82c51f1dcbe340257766d02aae3ea6c33a920a3fbdf917426660d0df1eecce1e99548aa7aafad45df3?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=2RK%5D3A%28ZCW9N7%7DRB801Q92V.png&size=1024)
这个密码在创建ECS实例的时候会提供给你，点击确认之后，就可以登录root用户。

我的本地是ubuntu系统，所以可以利用ssh登录。
``` bash
ssh root@xx.xx.xx.xx
```
登录之后，创建一个非root用户。
``` bash
adduser yinlijun
```
切换到该用户
``` bash
su yinlijun
```
### 上传本地公钥，方便部署。
``` bash
ssh-copy-id -i ~/.ssh/id_rsa.pub yinlijun@xx.xx.xx.xx
```

如果需要绑定域名，还需要如下申请域名和备案：

## 购买域名
前往[阿里域名服务](https://wanwang.aliyun.com/domain)选购域名。

## 域名备案
前往[备案专区](https://beian.aliyun.com/)，进行备案，需要时间比较长，我花了12天，备案完成后，阿里云会给你的ECS续费，续费的天数就是你的备案花的天数。
期间要填写资料，上传备案照片等等。

备案完成之后，需要设置域名解析。
![img](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/02ad73354fe97dd378b486349444aeebd3f5fc81ebe1853b1773f70416c90e7878a24b24afa25c27f122c679c20d523b?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=AJTAI%40%601%7DB3GU6Q%60ZWZO%24Z3.png&size=1024)
上图的记录值，填写你的ECS的公网IP。

### 在ECS上创建hexo仓库
````
git init --bare ~/project/hexo.git
````

### 捕获post操作

``` bash
touch ~/project/hexo.git/hooks/post-receive
```

输入如下内容

``` bash
#!/bin/bash -l
GIT_REPO=<到hexo.git的目录>
TMP_GIT_CLONE=<到临时blog的目录>
PUBLIC_WWW=<到blog服务的目录/public/blog>

rm -rf ${TMP_GIT_CLONE}
git clone ${GIT_REPO} ${TMP_GIT_CLONE}
rm -rf ${PUBLIC_WWW}/*
cp -rf ${TMP_GIT_CLONE}/* ${PUBLIC_WWW} 
```

### 本地blog(express)服务
新建express服务
``` bash
express blog-server
```

修改app.js，配置静态文件目录
``` javascript
app.use(express.static(path.join(__dirname, 'public/blog/')));
```
### 启动blog服务
``` bash
pm2 start bin/www
```
### 安装nginx，反向代理二级域名
通过域名访问默认的80端口，blog默认监听3000端口，因此需要配置代理。
创建blog.conf配置文件
``` bash
touch /etc/nginx/conf.d/blog.conf
```
输入如下内容
``` 
server {
    listen 80;
    server_name yinlijun.com www.yinlijun.com bloc.yinlijun.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

## 本地操作

### 把github上的blog目录clone下来
``` bash
git clone git@github.com:yinlijun2004/johnny-blog.git
```
如果你原来没有在github上部署hexo博客，也可以用hexo新建一个，然后托管到某个git仓库。

### 修改_config.yml配置
```
deploy:
  type: git
  repo: yinlijun@www.yinlijun.com:project/hexo.git
  branch: master
```

### 发布博客
撰写好博客后，如下命令部署。
``` bash
hexo clean
hexo g
hexo d
```

### 最终效果
[www.yinlijun.com](http://www.yinlijun.com)

## 有可能碰到的问题

### 外网无法访问端口
需要配置安全组规则。
![img](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/87f1aead6dd1582f56cae6e0179540db4090e83c7f3636e20bd26f24faf1bb04340403773fc999742081809ea9168224?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=3VAK%25~%7DO0ZZZXL9M%60%2963UF7.png&size=1024)
上图是默认开通的端口，如果你的端口不在里面，则需要自行添加安全组规则。

### hexo deploy之后没有反应
- 检查有没有上传本机公钥
- 检查hexo.git仓库的路径是否正确
- 检查备案的域名是否能正常访问




