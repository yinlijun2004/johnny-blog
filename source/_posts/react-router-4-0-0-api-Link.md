---
title: Link-react router 4.0.0 api中文文档
tags:
  - react
  - react-router
  - 翻译文档
categories: react router 4.0.0 api中文文档
date: 2016-11-29 14:18:08
---

[原文链接](https://react-router.now.sh/Link)

<font size='6em'>&lt;Link&gt;</font>

为你的应用提供可以声明的，可访问的导航功能。
```javascript
<Link to="/about" activeClassName="active">
  About
</Link>
```

<font color='red'>children: node | func</font>

Link组件可以接受一个函数作为子控件，使得允许你使用自定义的组件渲染Link对象。

<!-- more -->

子控件函数的参数是一个对象，包含如下属性：
- isActive: (bool) 当前链接是否激活
- location: 传递给Link的链接
- href: (string) 路由的url
- onClick: (func) dom的onClick处理函数
- transition: (func) 它是router.transitionTo的快捷方式，代表Link对象的to属性。
```javascript
<Link to="/courses">{
  ({isActive, location, href, onClick, transition}) => 
    <RaisedButton label="Courses" onClick={onClick} primary={isActive} href={href} />
}</Link>
```

<font color='red'>to: string | object</font>

链接的描述。
```
<Link to="/courses"/>
<Link to={{
  pathname: '/courses',
  query: { sort: 'name' },
  state: { fromDashboard: true }
}}/>
```

<font color='red'>activeStyle: object</font>

当链接匹配到<font color='grey' size='5em'>**to**</font>属性时的样式对象。它会和tyle对象合并之后生效。
```
<Link
  to="/courses"
  style={{ color: 'blue', background: 'gray' }}
  activeStyle={{ color: 'red' }}
/>
// 总是有灰色背景
// 链接为/foo时字体为blue
// 为/courses为red(激活状态)
```

<font color='red'>activeClassName: string</font>

当匹配到链接时，生效的className。
```javascript
<Link
  to="/courses"
  className="course-link"
  activeClassName="active"
/>
// will always have "course-link"
// at /courses it will be "course-link active"
```

<font color='red'>activeOnlyWhenExact: bool</font>

为true时, 只有当严格匹配，activeClassName和activeStyle才会生效。

```
<Link to="/courses" activeOnlyWhenExact activeClassName="active"/>
// /courses 被激活
// /courses/123 不被激活
```

<font color='red'>isActive: func</font>

允许自定义当前链接是否被激活，激活时返回true，否则返回false。
```javascript
<Link
  to="/"
  activeStyle={{ color: 'red' }}
  isActive={(location) => (
    // 只有当没有query字段时被激活
    !Object.keys(location.query).length
  )}
/>

<Link
  to="/courses"
  activeStyle={{ color: 'red' }}
  isActive={(location, props) => (
    // 匹配到"/courses" 或者 "/course/123"被激活
    // 尽管这不是真正意义上的被激活, it is
    // theoretically for the sake of a navigation menu
    location.pathname.match(/course(s)?/)
  )}
/>
```
<font color='red'>location</font>

如果你不想使用上下文中的location, 可以传递一个location的属性作为替代，这在链接比较深的redux应用中很有用。
```
<Match pattern="/foo" location={this.props.location}/>
```
<font size='6em'>&lt;/Link&gt;</font>