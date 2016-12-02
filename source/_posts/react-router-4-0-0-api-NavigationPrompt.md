---
title: NavigationPrompt-react router 4.0.0 api中文文档
date: 2016-12-02 19:16:00
tags: [react, react-router, 翻译文档]
categories: react router 4.0.0 api中文文档
---
<font size="6em">&lt;NavigationPrompt&gt;</font>

当你的应用进入一个状态，但是需要防止用户从当前状态离开时（比如填了一半的表单），渲染出一个导航确认（NavigationPrompt）。
```
{formIsHalfFilledOut && (
  <NavigationPrompt message="Are you sure you want to leave?"/>
)}
```
### <font color="red">message: string</font>

<!-- more -->

当用户试图从当前状态离开时显示的信息。
```
<NavigationPrompt message="Are you sure you want to leave?"/>
```
### <font color="red">message: func</font>

这个函数返回一个提示用户跳转的信息，如果返回true，则允许跳转，函数的参数是用户将要跳转的下一个链接，
```
<NavigationPrompt message={(location) => (
  `Are you sure you want to go to ${location.pathname}?`
)}/>
```
### <font color="red">when: bool</font>
when是一个逻辑变量，为true时，将显示message内容的弹窗，为false时，直接跳转。

<NavigationPrompt when={formIsHalfFilledOut} message="Are you sure?"/>
<font size="6em">&lt;/NavigationPrompt&gt;</font>

