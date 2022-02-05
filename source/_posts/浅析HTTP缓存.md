---
title: 浅析HTTP缓存
date: 2018-8-22 22:18:58
banner: http://img.yanyuanfe.cn/photo-1462331940025-496dfbfc7564.jpeg
tags:
	- HTTP
---
> HTTP缓存，面试必问，总结了一波。

![image](http://img.yanyuanfe.cn/photo-1462331940025-496dfbfc7564.jpeg)

<!--more-->

### 强缓存和协商缓存
浏览器缓存包含两种类型，即强缓存（也叫本地缓存）和协商缓存，浏览器在第一次请求发生后，再次请求时：

浏览器在请求某一资源时，会先获取该资源缓存的header信息，判断是否命中强缓存（cache-control和expires信息），若命中直接从缓存中获取资源信息，包括缓存header信息；本次请求根本就不会与服务器进行通信。

如果没有命中强缓存，浏览器会发送请求到服务器，请求会携带第一次请求返回的有关缓存的header字段信息（Last-Modified/If-Modified-Since和Etag/If-None-Match），由服务器根据请求中的相关header信息来比对结果是否协商缓存命中；若命中，则服务器返回新的响应header信息更新缓存中的对应header信息，但是并不返回资源内容，它会告知浏览器可以直接从缓存获取；否则返回最新的资源内容。

强缓存与协商缓存的区别


 类型 | 获取资源形式 | 状态码 | 发送请求到服务器
---|--- |--- |--- 
强缓存 | 从缓存 |  200（from cache) | 否，直接从缓存取
协商缓存 | 从缓存取 | 304（not modified）| 是，正如其名，通过服务器来告知缓存是否可用


### Expires
在HTTP1.0版本，强缓存通过Expires实现。
Expires是HTTP的响应头部，标识未来资源会过期的时间，在响应HTTP请求时告诉浏览器在过期时间前浏览器可以直接从浏览器缓存取数据，如果发起请求的时间超过Expires的时间则会发送请求到服务器重新获取资源。Expires 的一个缺点就是，返回的到期时间是服务器端的时间，这样存在一个问题，如果客户端的时间与服务器的时间相差很大（比如时钟不同步，或者跨时区），那么误差就很大。

### Cache-Control
在HTTP 1.1版开始，强缓存使用响应头Cache-Control: max-age=秒替代。
Cache-Control与Expires的作用一致，都是标识当前资源的有效期，控制浏览器是否直接从浏览器缓存取数据还是重新发请求到服务器获取数据。只是Cache-Control的选择更多，设置更细致，如果同时设置的话，其优先级高于Expires。

### Last-Modifield Date
在HTTP1.0版本中，协商缓存通过Last-Modifield实现。Last-Modifield标识资源的最新修改时间。当第一次请求资源时，服务器返回资源并在响应头中加入资源的最新修改时间Last-Modifield。再次请求时，浏览器在请求头中带上If-Modifield-Since将最新修改日期传回到服务器以进行比较。如果服务器上资源的最新修改日期与浏览器传回的If-Modifield-Since值匹配，会返回一个304响应，浏览器读取本地缓存，否则，服务器返回新的资源，并且更新Last-Modifield。

### Entity Tags
在HTTP1.0版本中，协商缓存通过ETag实现。ETag标识资源的唯一性。当第一次请求资源时，服务器返回资源并在响应头中加入ETag。再次请求时，浏览器在请求头中带上If-None-Match将ETag信息传回到服务器以进行比较。如果服务器上资源标识与浏览器传回的If-None-Match值匹配，会返回一个304响应，浏览器读取本地缓存，否则，服务器返回新的资源和新的ETag。

If-None-Match比If-Modifield-Since具有更高的优先级。根据HTTP规范，如果请求中同时出现了这两个头，则原始服务器禁止返回304。

Last-Modified与ETag一起使用时，服务器会优先验证ETag。

HTTP1.1中Etag的出现主要是为了解决几个Last-Modified比较难解决的问题：
1. Last-Modified标注的最后修改只能精确到秒级，如果某些文件在1秒钟以内，被修改多次的话，它将不能准确标注文件的修改时间
1. 如果某些文件会被定期生成，当有时内容并没有任何变化，但Last-Modified却改变了，导致文件没法使用缓存
1. 有可能存在服务器没有准确获取文件修改时间，或者与代理服务器时间不一致等情形


### 总结
强制缓存只有首次请求才会与服务端进行通信，读取缓存时不会发送请求，响应状态码为200（from disk cache或者from memory cache），HTTP1.1版本的Cache-Control其优先级高于HTTP1.0的Expires。
协商缓存每次请求都会与服务器进行通信，第一次请求数据和标识，第二次询问服务器是否有资源更新。如果命中缓存，响应状态码为304，HTTP1.1版本的ETag优先级高于HTTP1.0的Last-Modifield。


### 相关HTTP头

Cache-Control：
值可以是public、private、no-cache、no- store、no-transform、must-revalidate、proxy-revalidate、max-age
各个消息中的指令含义如下：
- Public指示响应可被任何缓存区缓存。
- Private指示对于单个用户的整个或部分响应消息，不能被共享缓存处理。这允许服务器仅仅描述当用户的部分响应消息，此响应消息对于其他用户的请求无效。
- no-cache指示请求或响应消息不能缓存，该选项并不是说可以设置”不缓存“，容易望文生义~
- no-store用于防止重要的信息被无意的发布。在请求消息中发送将使得请求和响应消息都不使用缓存，完全不存下來。
- max-age指示客户机可以接收生存期不大于指定时间（以秒为单位）的响应。
- min-fresh指示客户机可以接收响应时间小于当前时间加上指定时间的响应。
- max-stale指示客户机可以接收超出超时期间的响应消息。如果指定max-stale消息的值，那么客户机可以接收超出超时期指定值之内的响应消息。

Last-Modified/If-Modified-Since要配合Cache-Control使用。

Last-Modified：标示这个响应资源的最后修改时间。web服务器在响应请求时，告诉浏览器资源的最后修改时间。
If-Modified-Since：当资源过期时（使用Cache-Control标识的max-age），发现资源具有Last-Modified声明，则再次向web服务器请求时带上头 If-Modified-Since，表示请求时间。web服务器收到请求后发现有头If-Modified-Since 则与被请求资源的最后修改时间进行比对。若最后修改时间较新，说明资源又被改动过，则响应整片资源内容（写在响应消息包体内），HTTP 200；若最后修改时间较旧，说明资源无新修改，则响应HTTP 304 (无需包体，节省浏览)，告知浏览器继续使用所保存的cache。

Etag/If-None-Match也要配合Cache-Control使用。

Etag：web服务器响应请求时，告诉浏览器当前资源在服务器的唯一标识（生成规则由服务器决定）。Apache中，ETag的值，默认是对文件的索引节（INode），大小（Size）和最后修改时间（MTime）进行Hash后得到的。
If-None-Match：当资源过期时（使用Cache-Control标识的max-age），发现资源具有Etage声明，则再次向web服务器请求时带上头If-None-Match （Etag的值）。web服务器收到请求后发现有头If-None-Match 则与被请求资源的相应校验串进行比对，决定返回200或304。
