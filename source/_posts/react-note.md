---
title: react学习笔记
date: 2020-12-21 18:00:56
tags:
---

#### 样式模块化
```js
import hello from './index.module.css'

<h2 className={hello.title}>Hello React!</h2>
```
#### 有用的插件
- chrome
FeHelper
- VS Code
ES7 React/Redux/GraphQL/React

<!-- more -->

#### 开发环境配置代理
- package.json
```js
proxy: http://localhost:5000
```
- setupProxy.js
适用于配置多个后端端口。
```js
//因为是webpack用，所以只能用COMMJS方式引入
const proxy = require('http-proxy-middlewre')

module.exports = function(app) {
    app.use(
        proxy('/api1', { //遇见/api1前缀的请求，就会触发该代理配置
            target: 'http://localhost:5000', //请求转发给谁
            changeOrigin: true, //控制服务器收到的请求头中的Host的值
            pathRewrite: {'^/api1': ''} //重写请求路径（必须）
        })
    )
}
```


#### 发布订阅库
PubSubJs
```
yarn add pubsub-js
```

基本例子
```js
import PubSub from 'pubsub-js';

var mySubscriber = function(msg, data) {
    console.log(msg, data);
}

void token = PubSub.subscribe("MY TOPIC", mySubscriber);

PubSub.publish("MY TOPIC", "hello world");
```
- 先订阅，再发布
- 适用于任意组件通信
- 要在组件的componentWillUnmount中取消订阅。


#### 获取网络的方式
- XHR
- fetch
    - await async

#### 路由API
https://cdn.bootcss.com/history/4.7.2/history.js
```js
let history = History.createBrowserHistory();

let histroy = Histroy.createHashHistory();

history.push(path);

history.replace(path);

history.goBack();

history.goForward();
```


#### react-router
- \<BrowserRouter/>
- \<HashRouter/>
包裹在App外侧。

路由组件和一般组件的不同
1. 写法不同
- 一般组件：\<Demo/>
- 路由组件：\<Route path="/demo" component={Demo}>
2. 存放位置不同
- 一般组件: components目录
- 路由组件: pages目录
3. 接收到的props不同
- 一般组件: 写组件时传递了什么，就能收到什么
- 路由组件: history, location, match等windows.history路由信息

#### NavLink
NavLink是Link的升级版。匹配路由连接，并带高亮效果，可以设置activeClassName，表示选中之后追加的className，默认"active"
```js
<NavLink activeClassName="active" className='list-group-item' to='/about'>About</NavLink>
```
- NavLink封装
```js
import React, {Component} from 'react';
import {NavLink} from 'react-router';

export default class MyNavLink extends Component {
    render() {
        return (
            <NavLink activeClassName='active' className='list-group-item' {...this.props}>{this.props.children}</NavLink>
        )
    }
}
```
#### Switch
1. 通常情况下，path和component是一一匹配。
2. 一旦匹配上就不继续匹配了，可以提升效率。
```js
<Switch>
    <Route />
    <Route />
    <Redirect />
</Switch>
```
最后的Redirect是默认路由

#### 嵌套路由
1. 去掉exact
2. 内层路由，path要写全，即要写上父路由的path值。

#### 路由组件传递param
1. 注册路由 \<Route path='/demo/test/:name/:age' component={Test}/>
2. 路由跳转 \<Link to='/daemo/test/tom/18'>详情\</Link>
3. 获取参数 const {id, title} = this.props.match.params.

#### 路由组件获取search参数
1. 注册路由 \<Route path='/demo/test' component={Test}/>
2. 路由跳转 \<Link to='/daemo/test?name=tome&age=18'>详情\</Link>
3. 获取参数 const {name, age} = qs.parse(this.props.location.search.slice(1));
```js
//create-react-app 脚手架已经包含
import qs from 'querystring'

let obj = {name: 'tom', age: 18};
qs.stringify(obj); //name=tom&age=18

let str = 'carName=奔驰&price=199';
qs.parse(str);
```
#### 向路由组件传递state参数

```js
<Link to={{pathname:'/home/message/detail', state:{id:1, title:'title'}}}>链接</Link>
```
接收state
```js
const {id, title} = this.props.location.state;
```

#### 开启replace模式
```js
<Link replace={true} to="xxxxx">链接</Link>
```

#### 编程式路由跳转
```js
this.props.histroy.replace('xxxxxx')

//带state
this.props.histroy.replace('xxxxxx', {state: state})

this.props.histroy.push('xxxxxx')
```
#### withRouter
一般组件也可以用BOM属性。
- 注解修饰
```js
@withRouter
```
- 包裹类
```js
export default withRouter(MyComponent)
```

#### BrowserRouter和HashRouter的区别
1. 底层原理不一样
- BrowserRouter使用的H5的History API，不兼容IE9及以下版本。
- HashRouter使用的是URL的hash值。
2. url的表现不一样
- BrowserHistroy的路径中没有#
- HashRouter的路径中包含#
3. 刷新后对路由state参数的影响
- BrouserRouter没有任何影响，因为state保存在histroy对象中。
- HashRouter刷新后会导致路由state参数的丢失
4. 备注：HashRouter可以用于解决一些路径错误相关的问题。