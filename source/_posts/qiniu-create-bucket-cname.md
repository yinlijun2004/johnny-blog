---
title: 七牛存储空间别名创建
date: 2018-04-17 20:36:18
tags: [qiniu, aliyun, ecs]
---

七牛的存储空间会提供一个默认的域名，用来访问里面的资源，但是这个域名使用是受限的，七牛会有如下提示：

>测试域名
>
> 此类测试域名，限总流量，限单 IP 访问频率，限速，仅供测试使用，不能用于自定义域名的 CNAME 相关文档
>
> p71uhtys2.bkt.clouddn.com

我们可以创建一个融合CDN加速域名指向这个空间。

<!--more-->

步骤如下：

### 绑定域名
在存储空间详情页面，点击立即绑定一个域名。
![](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/5e9d840cd7942567816fd657772bfdbe28cd8d836eb51e630c0f3d8190e17406cff3244e7b2c7241768836c0ea7bc52f?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180417204257.png&size=1024)

进来之后，填写你要绑定的域名，和对应的存储空间。
![](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/e17d5d729c824b0da89d182e7cc930735ece4e9ce6db44b5f81b268c715670598e07cf95b1f5ecb22f3e6052bdadd6f5?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180417204544.png&size=1024)
注意这个域名是要经过备案的，备案的经过我之前有介绍过。

其他的地方按需要填写，填写完之后拉到下面点创建按钮。

然后会提示创建成功的消息。
![](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/eaecffc8d117ed56fa2bf60a3e34017b538c25b556529d1f4d154609c76f740b5a6862e98a36b09feeb32e6e1c387ca8?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180417204616.png&size=1024)

然后再回去看，会看到CDN的状态变为等待CNAME。
![](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/937e05f8155fb32f3193a7ad8ec7e131ba2637c5cebf6a3b6aeb2b2e995c5b461423102874ecb9e3cfe15957f30f20c8?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180417205340.png&size=1024)

### 添加域名解析
因为我的域名是在阿里云申请的，需要到阿里云去填写域名解析信息。

在云解析DNS界面，添加一条解析规则。
![](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/2a13b25abc02dc6fe2ea0887557e72cf51918e4780c987217f2ece9dbfb83f3a6d2af12eb16e439c6e450601ade40159?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180417205140.png&size=1024)

记录值就填刚才的CNAME，主机记录就填二级域名。

点击确认，等待十分钟，七牛那边的CDN状态就会变为成功。
![](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/7c7094162ed066f451bfc9d7e12764a95a29ecd5e4b6ad3989008dacbc236df7f7394167c529aaf8b73bf01d326cd219?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180417205804.png&size=1024)

或者windows下可以用命令行查询解析是否成功。
```bash
$ nslookup -qt=mx qiniu.yinlijun.com
▒▒Ȩ▒▒Ӧ▒▒:
▒▒▒▒▒▒:  UnKnown
Address:  192.168.1.1

qiniu.yinlijun.com      canonical name = qiniu.yinlijun.com.qiniudns.com
qiniu.yinlijun.com.qiniudns.com canonical name = largeqiniu.b0.aicdn.com
largeqiniu.b0.aicdn.com canonical name = nm.aicdn.com
nm.aicdn.com    canonical name = nm.ctn.aicdn.com

aicdn.com
        primary name server = ns1.ialloc.com
        responsible mail addr = wtzhu182.163.com
        serial  = 2015102101
        refresh = 3600 (1 hour)
        retry   = 180 (3 mins)
        expire  = 1209600 (14 days)
        default TTL = 10800 (3 hours)
```

这样的话，可以在代码里面，用你自定义的域名，代替原先七牛提供的测试域名了。
