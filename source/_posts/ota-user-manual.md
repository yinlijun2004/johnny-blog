---
title: OTA用户使用指南
date: 2018-04-10 10:14:21
tags:
---

## 用户管理

### 用户权限
系统分为四类用户，分别是
- 系统管理员
    - 批准用户注册申请
    - 删除用户
    - 重置用户密码
    - 管理测试SN
    - 管理Android版本
    - 管理版本类别

- 软件工程师
    - 新建版本
    - 修改版本
    - 上传固件包
    - 发布Alpha测试

- 测试工程师
    - 撤回Alpha测试
    - 发布Beta测试

- 运营工程师
    - 撤回Beta测试
    - 发布Release版本

#### 系统管理员权限
- 批准用户注册申请 
用户在OTA主界面注册成为OTA系统用户，注册时需要从以下列表选择一种身份：
    - 软件工程师
    - 测试工程师
    - 运营工程师

![](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/be617eb7472b662e17af2c693abf6022859fe84604c503b38480a183386139a99b110736f96a163c855f3285fc2760a6?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=1523346231%281%29.png&size=1024)

管理员登录之后，在用户管理页面，可以批准或者删除用户的注册申请。
![](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/27099809ab4a1b5194e22e59f458d34c927d7cc627fae348c9d3f1fe2b342abce0968f6c4c740a78cc51cece3d8a1189?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180410154736.png&size=1024)

- 删除用户 用户离职之后，管理员可以选择删除用户。

- 重置用户密码 用户忘记密码之后，管理员可以帮助用户重置用户密码。

- 管理测试SN 管理员可以添加SN号，用来标识对应的版本类别，如果勾选了Alpha，则可以进行Alpha版本状态下的版本升级；如果勾选了Beta，则可以进行Beta版本状态下的版本升级。

- 管理Android版本 管理员可以添加/删除Andriod版本，软件工程师可以从中选择一个版本。

- 管理版本类别 管理员可以添加/删除版本类别，一个版本类别标识一类终端产品，OTA会选择同一版本类别的更高的版本进行升级。

![](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/521e0ddaabd7cd53cdc0f5e1b4ff3864d74919bbe73f5682fa054c57b734ff14d81b3e1b0f2353339b9cf7f0c9d43f1b?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180410162612.png&size=1024)

#### 软件工程师权限
- 新建版本
软件工程师将版本编译出来之后，新建一个版本，需要填写如下信息：
    
    - 版本类别
    - 版本号
    - 版本描述
    - Android版本
    - 更新记录

![](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/b3331a73196a0d3d630155832b3bea3249e262bc3fccb5736bd39ce05a86181f065e320f8f140000cc6698dac00c3f73?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180410164617.png&size=1024)

创建之后，跳转到版本列表界面
![](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/8f8b4b1d6e83a323fcc830f362fd2f2e3b50aefe50a1c1ffd0c80b6367d0899df72c2ee4732bef8420f2cbd240d3e09f?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180410164917.png&size=1024)

点击包管理，进入升级包上传界面
![](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/45c9173a9a48fed2a8a1fe11c36abad36691e158957ca97f01a4acbc005144b515c043cb3069677c5f8e9c6616a22aaa?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180410165043.png&size=1024)

上传完完整包和固件包。

- 修改版本
软件工程师可以在新建状态下更改版本信息，但是不能修改版本类别和版本号。

- 发布Alpha测试
上传完固件包之后，回到版本列表界面，发布alpha测试。如果某台终端被允许Alpha测试，则会收到该版本的更新提示。

#### 测试工程师权限
- 撤回Alpha测试 
若测试工程师在测试过程中发现问题，可以撤回到新建状态，要求软件工程师重新上传固件版本。

- 发布Beta测试
若测试工程师在Alpha测试过程中没有发现问题，则发布Beta测试。

#### 运营工程师权限
- 撤回Beta测试
如果测试过程中发现问题，可以由运营工程师撤回到Alpha测试状态，由测试工程师确认。

- 发布Release版本
如果Beta测试过程中发现问题，可以由运营工程师发布Release状态。

### 版本状态说明
版本的状态分为四个：
- 新建状态 该状态下，可以修改版本信息，上传固件包。
- Alpha测试状态 该状态下，支持Alpha测试的终端将会收到版本提示。
- Beta测试状态 该状态下，支持Beta测试的终端将会收到版本提示。Beta测试状态与Alpha测试状态类似，Beta测试需要更多的终端设备验证。
- Release状态 该状态下，所有终端将会收到版本提示。

版本状态迁移：

![版本状态迁移：](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/f8279f93bdf2d68e2d599d0498b747cfe83430bbd7161ce4a39268a1156af75ff42effdec511f08e0dc4d6489e58dbbe?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=firmware_process.png&size=1024)

### 异常操作
- 如果某台终端，升级了某个Alpha版本或Beta版本，但是该版本最后被撤回，没有发布到Release状态，如何升级到后续版本？

    此时不能进行差分包升级了，需要进行OTA全包升级。

### 使用注意事项
- 上传固件包时，完整包是必选的。
- 与历史版本的差分包是可选的，然后终端有限选择差分包进行OTA升级，如果没有查询到差分包，则下载完整包。

