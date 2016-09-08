## 第二章 jQuery选择器

### 基本选择器

- `#id` 匹配id,  `$('#test')`
- `.class` 匹配类, `$('.test')`
- `element` 匹配标签, `$('div')`
- `*` 匹配所有元素, `$('*')`
- `selector1, select2,...,selectN` 匹配集合，如$('div, span, p.myClass')

### 层次选择器

- `$('ancestor decendant')` 选择后代元素
- `$('parent > child')` 选择子元素
- `$('prev + next')` 选择紧接在`prev`后的`next`元素，如: `$('.one + div')`，选择`one`之后的下个一个`div`同辈元素
- `$('prev ~ siblings')` 选择prev后的所有siblings元素，如`$('#two ~ div')`选择`two`之后的所有`div`同辈元素

### 过滤选择器

- `:first` 选取第一个元素，它只返回一个元素。 如`$('div:first')`选取第一个div元素。
- `:last` 选取最后一个元素，它只返回一个元素。
- `:not(selector)` 取出所有给定选择器匹配的元素，如`$('input:not(.myClass)')`,选取`class`不是`myClass`的`input`元素
- `:even` 选取索引为偶数的元素，索引从**0**开始
- `:odd` 选取奇数索引元素
- `:eq(index)` 选取指定索引元素
- `:gt(index)` 选取大于索引的元素
- `:lt(index)` 选取小于索引的元素
- `:header` 选取所有标题元素 如`$(':header')`，选取网页中所有`&lt;h1>,&lt;h2>,&lt;h3>;....`
- `:animated` 选取所有正在执行动画的元素 如：`$('div:animated')`选取正在执行动画的所有`div`元素
- `:focus` 选取当前获取焦点的元素 如`$(':focus')`，获取当前获取焦点的元素

### 内容过滤选择器

- `:contains(text)` 选取文本内容含有text的元素, 如`$('div:contains("我")')`,选取文本含有`我`的`div`元素
- `:empty` 选取不包含子元素或者文本为空的元素, 如`$('div:empty')`选取不包含子元素的`div`元素
- `:has(selector)` 选取含有选择器的所匹配的元素的元素，如`$('div:has(p)')`，选取含有`p`元素的`div`元素
- `:parent` 选取含有子元素或文本的元素 如`$('div:parent')`,选取拥有子元素的`div`元素

### 可见性过滤器

- `:hidden` 选取所有不可见元素, 如`$(':hidden')`, 选取所有不可见元素，`$('input:hidden')`选取所有不可见的input元素
- `:visible` 选取所有可见元素

### 属性过滤选择器

- `[attr]` 选取拥有此属性的元素, 如`$('div[id]')`, 选取拥有属性`id`的元素
- `[attr=val]` 选取`attr`属性为`val`的元素如, `$('div[title=test]')`选取`title`为`test`的`div`元素
- `[attr!=val]` 选取`attr`的值不为`val`的元素
- `[attr^=val]` 选取`attr`的值以`val`开始的元素
- `[attr$=val]` 选取`attr`的值以`val`结束的元素
- `[attr*=val]` 选取`attr`的值包含`val`的元素
- `[attr|=val]` 选取`attr`的值为`val`或以`val-`开始的元素
- `[attr~=val]` 选取`attr`的值包含`' val'`的元素（注意val前面要有空格）
- `[attr1][attr2][attrN]` 选取同时满足几个条件的元素，如`$('div[id][title$="test"]')`，选取拥有属性id，并且属性title以test结束的div元素

### 子元素过滤选择器

- `:nth-child(index/even/odd/equation)` 选取每个父元素下的第`index`个或者奇偶子元素
- `:first-child` 选取每个父元素的第一个子元素，如`$('ul li:first-child')`, 选择ul的第一个li元素
- `:last-child` 选取每个父元素的子元素
- `:only-child` 如果父控件一个子元素，那么匹配到，否则不被匹配。`$('ul li:only-child')`返回只有一个子元素且为li的li元素

### 表单对象属性过滤选择器

- `:enabled` 选取所有可用元素。`$('#form1 :enabled')`,选取`id`为`form1`的表单内的所有可用元素。
- `:disbaled` 选取所有不可用元素。
- `:checked` 选取所有被选中元素(单选框，复选框)。`$('input:checked')`,选取所有被选中的`input`元素
- `:selected` 选取所有被选中的选项元素（下拉列表）。`$('select option:seleted')`

### 表单选择器

- `:input` 选取所有`&lt;input>,&lt;textarea>,&lt;select>,&lt;button>`元素。如`$('#form1 :input')`
- `:text` 选取所有单行文本框。
- `:password` 选取所有密码框
- `:radio` 选取所有单选框
- `:submit` 选取所有提交按钮
- `:image` 选取所有的图像按钮
- `:reset` 选取所有的重置按钮
- `:button` 选取所有的按钮
- `:file` 选取所有的上川域
- `:hidden` 选取所有不可见元素