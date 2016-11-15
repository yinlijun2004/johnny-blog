---
title: git配置user.name和user.email
date: 2016-11-15 18:02:54
tags: git
---

github在统计提交的时候，会判断邮箱是否跟github的登陆邮箱匹配，不匹配则不计算活跃度，即不生成小绿块。

获取配置
```
yinlijun@yinlijun:~/personal_github/johnny-blog$ git config user.email
aaa@aaa.com
yinlijun@yinlijun:~/personal_github/johnny-blog$ git config user.name
aaa
```
<!--more-->

设置当前仓库的user.name/user.email
```
yinlijun@yinlijun:~/personal_github/johnny-blog$ git config user.email aaa
yinlijun@yinlijun:~/personal_github/johnny-blog$ git config user.email aaa@aaa.com
```
设置全局user.name/user.email
```
yinlijun@yinlijun:~/personal_github/johnny-blog$ git config --global user.name yinlijun
yinlijun@yinlijun:~/personal_github/johnny-blog$ git config --global user.email yinlijun2004@gmail.com
```
如果当前仓库未设置user.name/user.email则采用全局的user.name/user.email，否则当前仓库的user.name/user.email会覆盖全局的user.name/user.email。