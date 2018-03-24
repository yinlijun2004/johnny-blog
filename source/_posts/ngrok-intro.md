---
title: Windows下Ngrok工具的使用
date: 2018-03-24 10:09:26
tags: [ngrok, nodjs]
---

最近在本机调试七牛云上传，碰到一个问题。七牛云存储完成上传操作后，会有一个回调操作，告诉服务器上传结果。

>[https://developer.qiniu.com/kodo/sdk/1289/nodejs](https://developer.qiniu.com/kodo/sdk/1289/nodejs)
```javascript
var options = {
  scope: bucket,
  callbackUrl: 'http://api.example.com/qiniu/upload/callback',
  callbackBody: 'key=$(key)&hash=$(etag)&bucket=$(bucket)&fsize=$(fsize)&name=$(x:name)'
}
var putPolicy = new qiniu.rs.PutPolicy(options);
var uploadToken=putPolicy.uploadToken(mac);
```

上面的<b>http://api.example.com</b>就是回调的url，在开发模式下肯定是指向本机的，而不是生产服务器。

因此需要一个内网穿透的工具，在网上找到[Ngrok](https://www.ngrok.cc)。

Ngrok会生成一个指向你本机服务的隧道，隧道信息，包含一个三级域名（就是上面的回调url），本地端口（就是你的服务运行的端口），协议类型等信息。
![](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/56d49defca68f6178c2aadc601ea157c78729113241118fa8f5471199a8e707a5a5ee24a8e08fb8e264665971dbf42db?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=QQ%E5%9B%BE%E7%89%8720180324102303.png&size=1024)

隧道有免费的，也有付费的，但免费的貌似不稳定。

然后在server端可以用环境变量区分回调的url。
```javascript
qiniuCallbackDomain: process.env.NODE_ENV === "production" ? 'http://test.xxx.com' :'http://prod.xx.com',
```

Ngrok支持大部分的平台，我的是windows 64位，下载完[客户端](https://www.ngrok.cc/download.html)，解压，运行<b>Sunny-Ngrok启动工具.bat</b>，填入隧道ID，即可实现外网访问。
