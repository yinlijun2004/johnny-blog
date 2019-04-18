---
title: 记录一款内网穿透工具[natapp]
date: 2019-04-18 17:13:32
tags:
---

### NATAPP
这几天发现这款内网穿透工具，支持多平台，挺稳定的，但前提是要氪金，不氪金不稳定 - -， 价格大概9元/月。


#### 下载地址

https://natapp.cn/

下载好以后，前往注册，购买隧道。

会生成一个隧道token，隧道ID对应一个外网的域名和端口。

这个域名和端口映射到你指定的域名和端口，一般映射到本机。


#### 运行

```bash
./natapp -authtoken xxxxxxxxxxxxxxxxxxx
```
