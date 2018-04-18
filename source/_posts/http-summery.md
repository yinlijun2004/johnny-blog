---
title: HTTP请求头摘要
date: 2018-01-09 11:09:03
tags:
---

### If-Modified-Since
<b>If-Modified-Since</b>是一个条件式请求首部，服务器只在请求的资源在给定的日期时间之后对内容进行过修改的情况下才会返回，状态码<b>200</b>。如果请求的资源从那时起未经修改，则返回一个不带消息主体的<b>304</b>响应，而在<b>Last-Modified</b>首部中会带有上次修改的时间。

语法：If-Modified-Since: <day-name>, <day> <month> <year> <hour>:<minute>:<second> GMT

示例：If-Modified-Since: Wed, 21 Oct 2015 07:28:00 GMT

<!--more-->

### If-Unmodified-Since
只有当资源在指定的时间之后没有进行过修改，服务器在返回请求的资源，或是接受<b>POST</b>或其他non-safe方法的请求。如果所请求的资源在指定的时间之后发生的修改，那么会返回<b>412</b>(Proceondition Failed)错误。

常见应用场景：
- 与non-safe方法如<b>POST</b>搭配使用，可以用来优化并发控制，例如某些wiki应用中的做法：假如在原始副本获取之后，服务器上所存储的文档已经被修改，那么对其做出的编辑会被拒绝提交。

- 与含有<b>If-Range</b>消息头的范围请求搭配使用，用来确保新的请求片段来自于未经修改的文档。

语法：
If-Unmodified-Since: <day-name>, <day> <month> <year> <hour>:<minute>:<second> GMT

示例：
If-Unmodified-Since: Wed, 21 Oct 2015 07:28:00 GMT

### If-None-Match
对于<b>GET</b>和<b>HEAD</b>请求来说，当且仅当服务器上没有任何资源的ETag属性值与这个首部中列出的相匹配的时候，服务器端才会返回所请求的资源，响应码200。对其他方法来说，当且仅当最终确认没有已存在的资源的ETag的属性值与这个首部所列出的相匹配的时候，才会对请求进行相应的处理。


### If-Range
If-Range跟Range一起使用，但字段中的d饿到满足时，Range头字段才会起作用，同时服务器返回(Partial Content)状态码；如果字段值中的条件没有得到满足，服务器返回200状态码，返回完整的请求资源。

字段值中既可以用Last-Modified时间用作验证，也可以用ETag标记作为验证，但不能同时使用。

常见场景：
If-Range头字段通常用于断点续传的下载过程中，用来自上次中断后，确保下载的资源没有发生改变。

语法：
If-Range: <星期>, <日> <月> <年> <时>:<分>:<秒> GMT
If-Range: <etag>

示例：
If-Range: Wed, 21 Oct 2015 07:28:00 GMT
If-Range: "675af34563dc-tr34"

### Range
<b>Range</b>告知服务器返回文件的哪个部分。在一个Range首部中，可以一次性请求多个部分，服务器会以multipart文件的形式返回。如果服务器返回的是范围响应，需要使用206(Partial Content)状态码。加入所请求的范围不合法，那么服务器会返回416（Range Not Satisfiable）状态码，表示客户端错误。服务器允许忽略Range首部，从而返回整个文件，状态码用200。

语法：
```
Range: <unit>=<range-start>-
Range: <unit>=<range-start>-<range-end>
Range: <unit>=<range-start>-<range-end>, <range-start>-<range-end>
Range: <unit>=<range-start>-<range-end>, <range-start>-<range-end>, <range-start>-<range-end>
```
示例：
```
Range: bytes=200-1000, 2000-6576, 19000-
```




### 区别
- <b>If-Modified-Since</b>和<b>If-Unmodified-Since</b>

    <b>If-Mmodified-Since</b>只会出现在<b>GET</b>和<b>HEAD</b>请求中。

- <b>If-Modified-Since</b>和<b>If-None-Match</b>

    <b>If-Modified-Since</b>比<b>If-None-Match</b>优先级要高，但两者都存在时，后者会被忽略，除非服务器不支持<b>If-None-Match</b>


参考资料：
https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/If-Unmodified-Since

https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/If-Range

https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Range

https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/If-None-Match