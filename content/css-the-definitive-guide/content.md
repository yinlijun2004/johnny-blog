## 第一章 css和文档

-  文档内定义样式

        <style type="text/css">
            @import url(sheet2.css)
            h1 {color: maroon;}
        </style>

-   引入css文件

        <link type="text/css" rel="stylesheet" href="sheet1.css" media="all" />

## 第二章 选择器

- ID选择器和指定id的属性选择器不是一回事，h1#page-title和h1[id="page-title"]之间有差别。
- p.warning和p[class~="warning"]是等价的，但是p[class="warning"]匹配只有一个class的p元素。

    - 类似的还有[foo^="bar"] 选择foo属性值以bar开头的所有元素
    - [foo$="bar"] 选择foo属性值以bar结尾的所有元素
    - [foo*="bar"] 选择foo属性值中含有bar子串的所有元素

- 后代选择器中，两个元素之间的层次间隔可以是无限的，如果想匹配直接相连的父子元素，可以使用子结合符">"

- 兄弟元素选择器 h1 +  p {margin-top: 0}

- 伪类选择器，伪类的顺序很重要 link-visited-focus-hover-active

    - 连接伪类
        - :link 未访问的超连接
        - :visited  已访问的超连接

    - 动态伪类
        - :focus 拥有焦点的元素（可以接受键盘输入或者能已某种方式激活的元素）
        - :hover 鼠标指针悬停的元素
        - :active 用户输入激活的元素

    - 静态伪类
        - :first-child 第一个子元素
        - :first-letter 第一个字母
        - :first-line 第一行
        - :lang(fr) 根据语言选择
    - :first-letter和:first-line只能用于标记或段落之类的块元素，不能用于超连接等行内元素。

## 第三章 结构和层叠

- 选择器的特殊性

    - 给定的各个ID属性值，加0，1，0，0

    - 给形的各个类的属性值，属性选择或者伪类，加0，0，1，0

    - 给定的各个元素和伪元素加0，0，0，1

    - 结合符和通配选择器对特属性没有任何贡献

- 样式继承

    - 文本颜色可被继承
    - border不能被继承，一般的大多数框模型属性不能被继承（包括外边框，内边距，背景和边框）

    - 继承的值没有特殊性，连0特殊性都没有p(61)，因此需要避免不加区别的使用通配选择器。

- 层叠权重大小顺序

    - 读者的重要声明

    - 创作人员的重要声明

    - 创作人员的正常声明

    - 读者的正常声明

    - 用户代理声明

## 第四章 值和单位

- 颜色表示

    - 命名颜色 {color: gray;}

    - 函数式RGB {color: rgb(100%, 50%, 50%);}，或者{color: rgb(52, 95, 153);}

    - 十六进制RGB {color: #ff0000;} 如果3组各自成对，可以简写#f00，相当于#ff0000
- WEB安全颜色
    - RGB百分表示法，能被20整除的颜色，如rgb(60, 40, 120)

    - 十六进制，使用00，33，66，99，cc，ff，如#99ffcc

- 长度单位

    - 绝对长度单位 在打印文档的样式表时更为有用，Web设计中不使用绝对长度单位。

        - 英寸 (in)

        - 厘米(cm) 1in = 2.54cm

        - 毫米(mm) 1in = 25.4mm

        - 点(pt) 1in = 72pt

        - 派卡(pc)  1pc = 12pt, 6pc = 1in

    - 相对长度单位
        - em (em-height) 1em定义未一种给定字体的font-size的值，随元素的不同而不同，国外最常用，也是未来的趋势，更容易适配移动端。
        - ex (x-height) 1ex定位未一种给定字体的小写x的高度，不同字体ex的值不同。
        - px像素

## 第五章 字体
css定义了5中通用字体

- Serif 成比例的有上下短线的字体
- Sans-serif 成比例的，没有上下短线
- Monospace 不成比例，通常用于模拟打字机打出的问题
- Cursive  手写体
- Fantasy 无法用任何特征来定义的字体

特定字体：如Times，Verdana, Helvetica, Arial等，每一种特定的字体都会落入上述通用系列中。

- font-family 如h1 {font-family: Georgia, serif}, Georgia是特定字体，serif是通用字体。
- font-weight
    - normal
    - bold
    - bolder
    - lighter
    - 100/200/300/400/500/600/700/800/900
- font-size
    - xx-small/x-small/small/medium/large/x-large/xx-large/smaller/larger
    - &lt;length>/&lt;percentage>
- font-style字体风格
    - italic/oblique/normal/inherit
- font-variant 字体变形
    - small-caps 首字母大写
    - normal
    - inherit

## 第六章 文本属性

- text-indent 首行缩进
    - &lt;length> 如：p {text-indent: 3em}
    - &lt;percentag> 相对于父元素宽度
    - inhert
- text-align 水平对齐
    - left
    - right
    - center
    - justify 两端对齐，在打印领域很常见
    - inherit
- line-height 行高，定义了基线之间的距离，而不是字体大小，可以控制行间距。行间距=行高-字体大小
    - &lt;length>
    - &lt;percentage>
    - &lt;number>
    - normal
    - inherit
- vertical-align 行内元素和表单元格的垂直方向对齐
    - base-line 基线对齐，元素的底端和副元素的基线对齐
    - sub/super 上标/下标，相对于基线升高/降低
    -  bottom 元素行内框的底端和行框的底端对齐
    - text-bottom 元素行内框的底端和文本的底端对齐
    - top 与bottom相反
    - text-top 与text-bottom相反
    - middle 居中对齐
    - &lt;percentage>
    - &lt;length>
    - inherit
- word-spacing 字间隔
    - &lt;length>
    - normal
    - inherit
- letter-spacing 字母间隔
    - &lt;length>
    - normal
    - inherit
- text-transform
    - uppercase 全大写
    - lowercase 全小写
    - capitalize 单词首字母大写
    - none
    - inherit
- text-decoration 文字效果
    - none
    - underline 下划线
    - overline 上划线
    - line-through 中划线
    - blink
    - inherit
- text-shadow
    - none

    - 颜色值和三个长度值 text-shadow: silver 2px 2px 2px;
- white-space
    - normal 合并空白符，忽略换行符，允许自动换行。
    - nowrap 合并空白符，忽略换行符号，不允许自动换行。
    - pre 保留空白符号，保留换行符号，不允许自动换行。
    - pre-wrap 保留空白符，保留换行符，允许自动换行。
    - pre-line 合并空白符，保留换行符，允许自动换行。
    - inherit
- direction 文本方向
    - ltr
    - rtl
    - inherit

## 第七章 基本视觉格式化

水平格式化
七大水平属性

- margin-left
- border-left
- padding-left
- width
- padding-right
- border-right
- maring-right

这7个属性值加起来就是元素包含块的宽度，这往往是副元素的width值。
width就是左内边距到右内边距的距离。
这7个属性中，只有3个属性可以设置为auto: margin-left, width, margin-right，可以用auto弥补实际值和所需总和的差距。如果三个值都为非auto的某个值，这些格式化属性过分受限，此时会强制设置margin-right为auto.
不止一个auto的情况：

-  margin-left和margin-right为auto，此时元素居中，与text-align:center的区别，text-align只应用块级元素的内联内容，并不能使元素居中。
- margin-left和width为auto，此时margin-left为0，width会填满剩余包块。
- margin-right和width为auto，此时margin-right为0，width填满剩余包块。
- margin-left和margin-right和width都为auto，此时margin-left和margin-right都为0,width会填满包块。这种情况和默认是相同的。

水平外边距不会合并，父元素的内边距，边距和外边距可能影响子元素。

负外边距
   负外边距时，  会使得内容宽度超出包块，因为根据等式父包块width=7大水平之和，margin-left或margin-right为负时，width要增大。

垂直格式化
如果元素的内容的高度，大于元素框的高度，用户代理的具体行为将取决于overflow属性。
垂直格式化的七大属性

- margin-top
- border-top
- padding-top
- height
- padding-bottom
- border-bottom
- margin-bottom

这7个属性值的值和必须等于含块的height，其中，3个值可以设置为auto，margin-top，height，margin-bottom，如果margin-top或margin-bottom设置为auto则自动计算0，这就是为什么不容易设置元素为垂直居中，这与水平时不一样的。对于定位元素，上下边距为auto时，处理不同。

如果没有显式声明包含块的height，百分数高度为重置为auto，如果块级中场元素的height设置为auto，显示时的高度将恰好足以包含其内联内容。
如果块级正常流元素高度设置为auto，而且只有块级子元素，其默认高度将是从最高块级子元素的外边框边界到最低块级子元素外边框边界之间的距离。
不过，如果块级元素有padding或者border，则高度则是最高子元素的上外边距到其最低子元素的下外边距边界之间的距离。

合并垂直外边距
垂直合并只引用与外边距，不会应用于内边距和边框。
负的下外边距会使段落看上去向下拉，负的上边会使段落看上去向上拉。

## 第八章 内边距、边框和外边距

- border-style
    - none
    - hidden
    - dotted
    - dashed
    - solid
    - double
    - groove
    - ridge
    - inset
    - outset

- 设置多个边框样式 p.asize {boder-style:  solid dashed dotted solid;}
- boder-top-style/boder-right-style/border-bottom-style/border-left-style 单独设置边框样式
- border-width边框宽度
    - thin
    - medium
    - thick
    - &lt;length>
- border-top-width/border-right-width/border-bottom-width/border-left-width单独设置宽度
- boder-color边框颜色
    - &lt;color>
    - transparent 透明
- border-top-color/border-right-color/border-bottom-color/border-left-color单独设置颜色
- border-top/boder-right/border-bottom/border-left单独设置边框
    - [&lt;border-width>] [&lt;border-style>][boder-color]
- border 全局边框，应用到四条边
    - [&lt;border-width>] [&lt;border-style>][boder-color]

对于只包含文本的行，能改变行间距离的属性只有line-height, font-size和vertical-align。
行内元素使用正左右外边距，可以多出来水平空间，负的左右外边距会使行内元素与其他内容重叠。
行内元素的边框，不会改变行高。
行内元素使用正的左右边框可以多出来水平空间。

- padding 内边距
    - &lt;length>
    - &lt;percentage>

行内非替换元素使用左右内边距时，可以多出来水平空间，而上下边距不会改变行高。
可以想图像使用内边距,外边距，边框时，它可以改变行高，也可以水平方向留出距离。

- background-color 背景元素
    - &lt;color>
    - transparent
- background-image 如body {background-image: url(bg23.gif)}
    - &lt;uri>
    - none
- background-repeat 有方向的重复
    - repeat
    - repeat-x
    - repeat-y
    - no-repeat
- background-position 背景定位
    - &lt;percentage>
    - &lt;length>
    - left/center/right/top/bottom

background-position单个关键字等价
center: cener center
top: top center或者center top
bottom: bottom center或者center bottom
left: left center 或者center left
right: right center 或者center right

- background-attachment
    - scroll 跟随文档滚动
    - fixed 不跟随文档滚动

- background 简写属性
    - &lt;background-color> || &lt;background-image> || &lt;backgroud-repeat> || &lt;background-attachment> || &lt;background-position>

## 第十章 浮动和定位

- float
    - left
    - right
    - none

浮动元素会从文档的正常流中删除，它还是会影响布局。其他内容会环绕元素，浮动元素的外边距不会合并

- position
    - static 默认 块元素生成一个矩形框，行内元素创建一个或多个行框，置于父元素中
    - relative 元素偏移某个距离，元素仍保持未定位前的形状，它原本占据的空间仍保留
    - absolute 元素框从文档流中完全删除，并相对于其包含块定位。
    - fixed 元素的表现类似于absolute，不过，其包含块是视窗本身。

- 偏移属性，在position为relative,absolute和fixed时，可以设置偏移属性top right bottom left
    - &lt;length>
    - &lt;percentage>
    - auto 初始值

包含块

根元素的包含块，由用户代理建立，即html元素或body元素
对于一个非根元素，如果其position时relative或static，包含块则由最近的块级框，表单元格或行内块祖先框的内容边界构成。
对于一个非根元素，如果其position时absolute，包含块则由最近的position值不是static的祖先元素。

- overflow
    - visible
    - hidden
    - scroll
    - auto