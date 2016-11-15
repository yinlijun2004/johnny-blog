---
title: 锋利的jQeury 第四章-笔记 
date: 2016-9-5 18:04:19
tags: jQuery
---

## 第四章 jQuery中的事件和动画

- window.onload方法，相当于`jQuery`中的$(window).load(function(){})方法。都是等文档中的所有元素加载完成时调用，包括关联css文件和javascript文件。
- $(document).ready(function() {})则不会等待关联文件下载完，在DOM准备好之后就会调用。
- $(window).load(function(){})可以调用多次，都会执行，而window.onload只会调用最后一次赋予的函数。
- $(document).ready(function() {})可以简写为$(function() {});

<!--more-->

### 事件绑定 
- bind(type [, data], fn);第1个参数是事件类型，包括focus、load、click、dbclik等。，第2个参数可选，作为event.data传递，第3个参数是回调函数。示例：
```javascript
$(function(){
    $('#panel h5.head').bind('click', function(e) {
        $(this).next().show();
    })
})
```
- bind函数可以级联，如
```javascript
$('#panel h5.head').bind('mouseover', function(e) {
    $(this).next().show();
}).bind('mouseout', function(e) {
    $(this).next().hide();
})
```
- bind可以简写，如
```javascript
$('#panel h5.head').click(function(e) {
    $(this).next().toggle();
})
```
- 合成事件

    `jQuery`有两个合成事件，hover()和toggle(),都有两个回调函数作为参数，可以看成是语法糖。hover表示移入移出两个事件，toggle表示前后两次点击事件，如
    ```javascript
    $('#panel h5.head').hover(function() {
        $(this).next().show(); //划过时显示下一个元素
    }, function() {
        $(this).next().hide(); //划出隐藏下一个元素
    })
    ```
    ```javascript
    $('#panel h5.head').toggle(function() {
        $(this).next().show(); //点击时显示下一个元素
    }, function() {
        $(this).next().hide(); //再次点击时隐藏下一个元素
    })
    ```
- 事件对象的属性
    - event.type 事件类型，字符串格式，如'click', 'dbclick', 'load'等。
    - event.preventDefault()阻止默认行为，可以用return false代替。
    - event.stopProgagation()组织事件冒泡，可以用return false代替。
    - event.target 触发事件的元素。
    - event.pageX event.pageY,相对于页面的x，y坐标。
    - event.which 不同事件含义不同，鼠标事件表示那个键，键盘事件表示键值。
    - event.metaKey ctl键是否按下。
- 移除事件 unbind([type],[data]) 
    - 如果没有参数，表示删除所有事件。
    - 如果只提供type，表示移除所有type对应的事件回调。
    - 如果传递了2个参数，表示对应的处理函数会被删除。
- 单次事件 one(type, [data], fn);执行一次后，事件回被移除。
- 事件模拟
    - 原生事件 $('#btn').trigger('click')或者$('#btn').click();
    - 自定义事件
    ```javascript
    $('#btn').bind('myClick', function(){
        $('#test').append('&lt;p>我的自定义事件&lt;/p>')
    });

    $('#btn').trigger('myClick'); //触发事件。
    ```    
    - 传递数据
    ```javascript
    $('#btn').bind('myClick', function(event, msg1, msg2){
        $('#test').append('&lt;p>我的自定义事件&lt;/p>')
    });

    $('#btn').trigger('myClick'. ['参数1'，'参数2']); //触发事件。
    ```   
    - 执行默认操作 triggerHandler，例如，只触发focus事件，但是不获取焦点（浏览器默认行为）
    ```javascript
    $('input').triggerHandler('focus');
    ```
- 事件命名空间
    - 添加命名空间
    ```javascript
    $('#div').bind('mouseover.plugin', function() {

    })
    $('#btn').click(function() {
        $('#div').unbind('.plugin'); //移除上面的事件。
    })
    ```
    
## 动画
 - show(duration) 显示元素，duration不为0时，表示显示动画。将元素display属性从’none‘设置为原来的值
 - hide(duration) 隐藏元素，duration不为0时，表示隐藏动画。将元素display属性设置为'none'
 - fadeIn(duration) 与show一样，但是不改变display样式。
 - fadeOut(duration) 与hide一样，但是不改变display样式。
 - slideUp(duration)和slideDown(duration)，只改变高度。
 - animate(param, speed, callback)
    - param 包含样式属性值，如{left: '400px', top: '400px'}，可累加或累减，如{left: '+=400px'}
    - speed 动画时间
    - callback 动画完成回调
    -累加动画
    ```javascript
    $('#panel').animate({left:'400px', height:'200px', opacity:1}, 3000)
        .animate({top:'200px', width:'200px'}, 3000， function() {
            consle.log('animate done');
        })
        .fadeOut('slow');
    ```
    - 停止动画stop([clearQueue], [gotoEnd])
    - 判断是否处于动画状态 $(element).is(':animated')
    - 延迟动画delay(duration),延迟一段时间开始动画，$(this).animate({left:'400px'}).delay(200);
    - 其他动画方法
        - toggle(speed, [callback]) 显示/隐藏元素
        - slideToggle(speed, [easing], [callback])通过改变高度来显示/隐藏元素
        - fadeTo(speed, opacity, [callback]) 通过改变不透明度来显示/隐藏元素
        - fadeToggle(speed, [easing], [callback]) 通过不透明度来显示/隐藏元素。
