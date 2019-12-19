---
title: ubuntu 18.04上nginx部署php项目
date: 2019-11-20 11:11:16
tags: [nginx, php]
---

nginx的安装方法之前文章已经提过，在此不再赘述。

将php项目拷贝到/home/xxx/projects/guanwang/web
```
scp -i ~/login.pem -r ~/Desktop/web/ xxx@133.xx.xxx.xx:/home/xxx/projects/guanwang
```

在php项目下：
- 修改config/database.php文件，配置好数据库参数，创建数据库并导入初始数据。

- 修改cache/config/domain.php中的域名

- 修改cache/config/site/1.php中的域名


#### 更新软件库
```
sudo apt-get update
```

#### 安装php
```
sudo apt-get install php-fpm php-cli php-mysql php-gd php-imagick php-recode php-tidy php-xmlrpc
```

#### 创建nginx的站点配置文件
```
touch /etc/nginx/conf.d/guanwang.conf
```
内容如下
```
server {
    listen 80;
    root /home/xxx/projects/guanwang/web;
    index  index.php index.html index.htm;
    server_name test.jjyy.com;

    location / {
        try_files $uri $uri/ =404;       
    }

  
     # pass PHP scripts to FastCGI server
        #
        location ~ \.php$ {
               include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (or other unix sockets):
               fastcgi_pass unix:/run/php/php7.2-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
        }
}
```

#### 重启nginx服务
```
sudo systemctl restart nginx.service
```

#### 创建phpinfo.php文件
```
touch  /var/www/html/phpinfo.php
```
内容如下
```
<?php phpinfo( ); ?>
```


#### 参考
- https://websiteforstudents.com/setup-nginx-web-servers-with-php-support-on-ubuntu-servers/