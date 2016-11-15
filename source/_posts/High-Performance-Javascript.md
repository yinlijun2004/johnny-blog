---
title: 高性能Javscript - 笔记
date: 2016-11-5 18:04:19
tags: javascript
---

## 第一章 加载和执行

### 减少Javascript加载对性能的影响

- &lt;/body>闭合标签之前，将所有&lt;javascript>标签放到页面底部。这能确保在脚本执行前页面已经完成了渲染
- 合并脚本，减少&lt;javascript>标签
- 使用&lt;javascript>标签的defer属性
```javascript
    <script type='text/javascript' src='file1.js' defer></script>
```

<!--more-->

- 利用动态创建的&lt;javascript>元素来下载并执行代码
```javascript
    var script = document.createElemnet('script');
    script.type = 'text/javascript';
    script.onload = function() {
        alert('Script loaded!');
    }
    script.src = 'file1.js';
    document.getElementByTagName('head')[0].appendChild(script);
```
- 使用XHR对象下载Javascript代码并注入页面中
```javascript
    var xhr = new XMLHttpRequest();
    xhr.open('get', 'file1.js', true);
    xhr.onreadystatechange = function() {
        if(xhr.readyState == 4) {
            if(xhr.status >= 200 && xhr.status < 300 || xhr.status === 304) {
                var script = document.createElemnet('script');
                script.type = 'text/javascript';
                script.text = xhr.responseText;
                document.body.appendChild(script);
            }
        }
    }
```

### 几种类库加载js的方式
- YUI3
```javascript
    <script type='text/javascript' src='http://yui.yahooapis.com/combo?3.0.0/build/yui/yui-min.js'></script>
    <script type='text/javascript'>
        YUI().use('dom', function(Y) {
            Y.DOM.addClass(document.body, 'loaded');
        })
    </script>
```
- LazyLoad
```javascript
    <script type='text/javascript' src='lazyload-min.js'></script>
    <script type='text/javascript'>
        LazyLoad.js('the-rest.js', function() {
            Applicationn.init();
        })
    </script>
```
- LABjs
```javascript
    <script type='text/javascript' src='lab.js'></script>
    <script type='text/javascript'> 
        $LAB.script('first-file.js')
            .wait()
            .script('the-rest.js')
            .wait(function() {
                Applicationn.init();
            })
```

## 第二章 数据存取

在Javascript中，数据存储的位置会对代码整体性能产成重大影响。数据存储共有4中方式：字面量、变量、数组项、对象成员。它们有着各自的性能特点。


- 访问字面量和局部变量的速度最快，相反，访问数组元素和对象成员相对较慢。
- 由于局部变量存在于作用域链的起始位置，因此访问局部变量比访问跨作用域变量更快。变量在作用域中的位置越深，访问所需时间就越长。由于全局变量总处在作用域的最末端，因此访问速度时最慢的。
- 避免使用with语句。
- 嵌套的对象成员会明显影响性能，尽量少用。
- 属性或方法在原型链中的位置越深，访问它的速度也越慢。
- 通常来说，你可以通过把常用的对象成员、数组元素、跨域变量保存在局部变量中来改善Javascript性能，因为局部变量访问速度更快。

## 第三章 DOM编程

访问和操作DOM时现在Web应用的重要部分。但每次穿越链接ECMAScript和DOM都会有性能消耗，为了减少DOM编程带来的消耗，请记住以下几点：

- 最小化DOM访问次数，尽可能在Javascript端处理。
- 如果需要多次访问某个DOM节点，请使用局部变量存储它的引用。
- 小心处理HTML集合，把集合的长度缓存到一个变量中，并在迭代中使用它。如果需要经常操作集合，建议把它靠背到一个数组中。
- 如果可能的话，使用速度更快的API，比如querySelecetAll()和firstElementChild。
- 要留意重排和重绘，批量修改样式时，离线操作DOM树，使用缓存，并减少布局信息的次数。
- 动画中使用绝对定位，使用拖放代理。
- 使用事件委托来减少事件处理器的数量。