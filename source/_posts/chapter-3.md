---
title: 锋利的jQeury 第三章-笔记 
date: 2016-9-3 18:04:19
tags: jQuery
---

## 第三章 jQuery中的DOM操作
 - 查找`DOM`节点

    ```javascript
    var $li = $('ul li:eq(1)'); //获取<ul>里的第二个<li>节点
    ```

<!--more-->

 - 查找属性节点

    ```javascript
    var $para = $('p'); 
    var p_txt = $para.attr('title');
    ```
<!--more-->

 - 创建元素节点

    ```javascript
    var $li = $('<li></li>');
    ```

 - 创建文本节点

    ```javascript
    var $li = $('<li>香蕉</li>');
    ```

 - 创建属性节点

    ```javascript
    var $li = $('<li title="香蕉">香蕉</li>');
    ```
    
 - 插入节点的方法
    - append 追加元素。$('p').append($("&lt;b>你好&lt;/b>"))，A.append(B)之后，A和B是父子关系。
    - appendTo 跟append相反，A.append(B) 相当于 B.appendTo(A)
    - prepend 插入到前面
    - prependTo 跟prepend相反
    - after 在每个匹配元素之后插入内容，A.after(B)之后，A与B是兄弟元素
    - intertAfter跟after相反，A.after(B)相当于B.instertAfter(A)
    - before，跟after相反
    - insertBefore，A.before(B)相当于B.insertBefore(A)
    - 删除节点的方法
    - remove 删除匹配的元素 $('ul li:eq(1)').remove()，删除第二个li元素，remove返回值是删除的元素
    - detach 与remove不同，detach会保留所有该元素绑定的事件，附加的数据等。
    - empty 清空节点

- 复制节点 clone，可以传入一个boolean参数表示是否同时复制元素中所绑定的事件。

- 替换节点
    - replaceWith， A.replaceWith(B)，用B替换A
    - replaceAll 跟replaceWith相反，A.replaceWith(B)相当于B.replaceAll(A)

- 包裹节点
    - wrap $('strong').wrap('&lt;b>&lt;/b>') 用&lt;b>标签将&lt;strong>标签包起来，如果有多个匹配，则每个元素单独包裹。
    - wrapAll，将所有的元素用一个元素来包裹。如果匹配的节点之间有其他节点，其他节点会被放到包裹元素之后。
    - wrapInner 将子内容（包括文本节点）包裹起来。

- 获取属性，$('p').atter('title')

- 设置属性

    - $('p').attr('title', 'your title') 设置单个属性

    - $('p').attr({'title', 'your title', 'name': 'test'})，设置多个属性

- 删除属性 $('p').removeAttr('title');

- 获取样式 $('p').attr('class')获取&lt;p>元素的class

- 设置样式 $('p').attr('class', 'high') 设置样式

- 追加样式 $('p').addClass('another')

- 移除样式

    - $('p').removeClass('another') 移除一个样式

    - $('p').removeClass('another high') 移除多个样式

- 切换样式 $('p').toggleClass('another')，如果another类名不存在则添加之，否则删除之

- 判断是否含有某个样式 $('p').hasClass('another')
- html()方法 获取或设置html内容，相当于javascript的innerHTML属性
- text() 获取或设置文本内容，相当于javascript的innerText属性

    - 获取: var text = $('p').text()

    - 设置 $('p').text('你喜欢的水果是')
- val()方法 获取或设置value，相当于javascript的value属性

- 遍历节点
    - children() 返回子节点（DOM原声节点，非jQuery节点）
    - next()方法，获取紧临的同辈元素，也就是返回下一个兄弟节点。
    - prev()，跟next相反，返回上一个兄弟节点
    - siblings()返回前后所有的同辈元素。
    - closest() 取得最近的匹配元素，往父控件追溯。如$(e.target).closest('li').css('color', 'red');给点击的目标元素附近的li元素添加颜色。
    - parents() 获得集合中每个匹配元素的祖先元素。
    - parent() 获得集合中每个匹配元素的父级元素。
    - find() / filter() / nextAll() / prevAll() 等。

- 获取样式 $('p').css('color') 获取&lt;p>的样式颜色

- 设置样式

    - $('p').css('color', 'red');

    - $('p').css({"fontSize": "30px", "backgroundColor": "#888888"'});

    - 元素定位
        - offset()，获取元素在当前视窗中的相对偏移。
        - position() 获取元素相对于最近的一个position样式为relative或absolute（为啥）的父节点的相对偏移。
    - scrollLeft() 获取或设置水平滚动条的位置
    - scrollRight() 获取或设置垂直滚动条的位置。
