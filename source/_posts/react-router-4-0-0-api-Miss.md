---
title: Miss-react router 4.0.0 api中文文档
date: 2016-11-29 13:55:49
tags: [react, react-router, 翻译文档]
categories: react router 4.0.0 api中文文档
---
[原文链接](https://react-router.now.sh/Miss)

<font size='6em'>&lt;Miss&gt;</font>

当没有匹配到当前的地址时，将会渲染Miss。
```javascript
const App = () => (
  <Router>
    <Match pattern="/foo"/>
    <Match pattern="/bar"/>
    <Miss component={NoMatch}/>
  </Router>
)

const NoMatch = ({ location }) => (
  <div>Nothing matched {location.pathname}.</div>
)
```
<!-- more -->
### <font color='red'>component</font>
跟Match一样的，表示将要渲染的组建，但是不同的是只有location一个prop传进去。
```
<Miss component={NoMatch}/>
```
### <font color=red>render: func</font>

跟Match一样的，可以提供一个渲染函数，同样，也只有location一个prop传进去。
```
<Miss render={({ location }) => (
  <div>Nothing matched {location.pathname}.</div>
)}/>
```
<font size='6em'>&lt;/Miss&gt;<font>