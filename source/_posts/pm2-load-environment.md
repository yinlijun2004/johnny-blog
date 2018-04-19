---
title: pm2启动参数加入环境变量
date: 2018-04-18 17:35:30
tags: [pm2, nodejs]
---
pm2用来管理nodejs应用的时候，可以简单的用命令行来启动，如:
```bash
pm2 start ./ota_server/bin/www --name=ota-server
```

更常用的方法是将启动脚本配置成json，如:

<!-- more -->
start.json
```json
{
    app: [
        {
            name: "ota-server",
            script: "./ota_server/bin/www",
            watch: false,
        }
    ],
}
```

如果你的项目package.json启动参数里面带了环境变量，比如：
```json
"scripts": {
    "start"："cross-env NODE_ENV=production node ./bin/www",
    "start-dev": "node ./bin/www"
}
```

这个时候可以在start.json里面加上相应的环境变量配置。

start.json
```json
{
    app: [
        {
            name: "ota-server",
            script: "./ota_server/bin/www",
            watch: false,
            env_prod: {
                "NODE_ENV": "production",
            },
            env_dev: {
                "NODE_ENV": "development",
            }
        }
    ],
}
```

然后用如下命令加入环境变量了：
```bash
pm2 start start.json --env=prod
```

这样可以在代码里面这样应用环境变量：
```javascript
  domain: process.env.NODE_ENV === "production" ? 'http://ota.yinlijun.com' : 'http://yinlijun.viphk.ngrok.org',

```