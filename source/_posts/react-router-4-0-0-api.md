---
title: [BrowserRouter] react router 4.0.0 api中文文档
date: 2016-11-24 14:54:00
tags: [react, react-router, 翻译文档]
catagarys: react router 4.0.0 api 中文文档
---
[原文链接](https://react-router.now.sh/BrowserRouter)

<font size='5em'>\<BrowserRouter></font>

保持你的界面与浏览器历史记录保持同步。
```html5
<BrowserRouter>
  <App/>
</BrowserRouter>
```
<font color='#FF0000'>basename</font>

所有路由的根URL，如果你的应用处于服务端的子目录, 你可以设置这个值为该子目录。

```html5
<BrowserRouter basename="/calendar" />

// 假设Link标签如下所示：
<Link to="/today"/>
// 那么超链接指向 "/calendar/today"
</BrowserRouter>
```
<font size='5em'>\</BrowserRouter></font>
