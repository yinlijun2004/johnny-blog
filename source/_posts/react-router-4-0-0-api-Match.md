---
title: Match-react router 4.0.0 api中文文档
date: 2016-11-25 13:05:50
tags: [react, react-router, 翻译文档]
---
[原文链接](https://react-router.now.sh/Match)

# &lt;Match&gt;

匹配到地址(location)时进行UI渲染

### <font color='red'>pattern: string</font>

任何[path-to-regexp](https://www.npmjs.com/package/path-to-regexp)可以理解的URL模式表达式
```html
<Match pattern="/users/:id" component={User}/>
```
<!--more-->

### <font color='red'>exactly: bool</font>

当为true时，只有模式表达式严格匹配时，才算匹配上。

| pattern |	location.pathname | exactly? | matches? |
| --- | :---: | :----: | :----: |
| /foo | /foo/bar |	yes |	no
| /foo | /foo/bar |	no | yes
```html
<Match pattern="/foo" exactly component={Foo}/>
```

### <font color='red'>location</font>

如果你不想匹配上下文(context)中的地址(location)时，你可以传入一个location参数来替代。
```html
<Match pattern="/foo" location={{ pathname: '/foo' }}/>
```

### <font color='red'>component</font>
当匹配到地址时渲染的React组件，渲染该组件时，会传入如下属性(props)：
- pattern: (string) 匹配表达式.
- pathname: (string) 匹配后的路径.
- isExact: (bool) 是否严格匹配 (v. partial).
- location: 匹配到的location对象.
- params: 根据匹配模式解析出来的参数

如下所示：
```javascript
class User extends React.Component {
    render() {
        const {params, pattern, pathname, isExact, location} = this.props;
        return (<div>
            <h2>User</h2>
            <div>
                location:{JSON.stringify(location)}
                <br />
                pattern:{pattern}
                <br />
                pathname:{pathname}
                <br />
                isExact:{isExact ? "true" : "false"}
                <br />
                params:{JSON.stringify(params, null, 2)}
                <br />
            </div>
        </div>)
    }
}

<Match pattern="/:user" component={User}/>
```
效果如下
```
location:{"pathname":"/kim","search":"","hash":"","state":null,"query":null,"key":"a21zge"}
pattern:/:user
pathname:/kim
isExact:true
params:{ "user": "kim" }
```

### <font color='red'>render: func</font>

相对于直接渲染一个组件，你可以之间传一个渲染函数，该函数被调用时，会传入组件一样的参数。

这相当于提供了一种内联(inline rendering)渲染方式，或者对Match的封装(wrapping)。
```javascript
// 非常方便的内联渲染
<Match pattern="/home" render={() => <div>Home</div>}/>

// 对Match进行封装
const MatchWithFade = ({ component:Component, ...rest }) => (
  <Match {...rest} render={(matchProps) => (
    <FadeIn>
      <Component {...matchProps}/>
    </FadeIn>
  )}/>
)

<MatchWithFade pattern="/cool" component={Something}/>
```
### <font color='red'>children: func</font>

有时你可能需要根据匹配与否进行不同的渲染，这时，你可以使用children属性，它的类型为function，
它的表现有点象render函数，但是有以下不同：
 
(1) 不管有没有匹配上都会被调用。 

(2) 回传递一个matched属性表示有没有被匹配上。

It seems unlikely you’ll need this for anything besides animating when a component transitions from matching to not matching and back, but who knows?

```javascript
<Match children={({ matched, ...rest}) => (
  {/* 因为Animate 总是能被渲染，所以你可以利用组件的生命周期(lifecycle)来实现动画。*/}
  <Animate>
    {matched && (
      <Something {...rest}/>
    )}
  </Animate>
)}/>
```

# &lt;/Match&gt;