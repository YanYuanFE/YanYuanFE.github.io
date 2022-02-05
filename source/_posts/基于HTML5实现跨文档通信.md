---
title: 基于HTML5实现跨文档通信
date: 2017-8-25 20:56:14
banner: http://img.yanyuanfe.cn/Creating-a-HTML5-Flipbook.jpg
tags:
	- HTML5
---


> 当我们谈论Web通信的时候，实际上谈论的是两个略有不同的系统：跨文档通信(cross-document messaging)和通道通信(channel messaging)。跨文档通信就是我们国内更为熟知的HTML5 window.postMessage()应用的那种通信；通道通信也被称为”MessageChannel”. 伴随着server-sent事件以及web sockets, 跨文档通信和通道通信成为HTML5 通信接口“套件”中有用的一部分。

![image](http://img.yanyuanfe.cn/Creating-a-HTML5-Flipbook.jpg)

<!--more-->

### 写在前面
曾经在Web开发中遇到这样一个需求，你的网页中需要嵌入一个跨域的iframe页面，iframe提供了点击的按钮，点击按钮触发事件，需要调用你的网页的一个方法来处理点击事件。但是，基于浏览器的同源策略，Web浏览器不允许窗口间的通信。同源策略指的是：在页面中，从某一个域加载的脚本不能访问从另一个域加载的窗口内容。虽说同源策略在很多情况下能保护我们的信息安全，但是在很多情况下，我们确实需要在网页中嵌入来自其他网站的内容。HTML5利用跨文档通信满足了这种复杂需求。借助跨文档通信API中的postMessage方法与message事件，能在两个页面之间创建一个受控制的通信通道。

### 用法
从MDN(https://developer.mozilla.org/zh-CN/docs/Web/API/Window/postMessage)的文档里可以详细了解postMessage的用法：

otherWindow.postMessage（message, targetOrigin, [transfer]）;

postMessage属于window的一个方法。
otherWindow: 代表其他窗口的一个引用，其值可以是iframe的contentWindow属性、执行window.open返回的窗口对象、命名过或数值索引的window.iframes。

message: 指将要发送的消息本身。大部分浏览器都支持发送各种数据类型的消息。

targetOrigin: 发送消息的目标窗口的域名，它可以是一个URI或者是通配符'*'。消息发送时，如果目标窗口的协议、主机地址或端口三者任意一项与targetOrigin的值不匹配，那么消息将不会被发送。这也是HTML5对于跨文档通信的一种安全防御措施。

transfer：可选参数，基本不会使用到，在此不做说明。

当目标窗口接收到消息时，message事件将会触发。对于DOM事件，可以使用body元素的onmessage属性来进行处理，或者直接利用addEventListener：


``` js
window.addEventListener('message', handleMessage, false);

function handleMessage(event) {
    console.log(event);
    var origin = event.origin || event.originalEvent.orgin;
    if(origin ! == 'http://example.org:8080') return;
}

```

 
事件对象event的属性包括：

data：从其他窗口传递过来的消息。

origin： 调用postMessage时消息发送方窗口的origin。这个字符串由协议、'://'、域名、':端口号'拼接而成。

source: 对发送消息的窗口对象的引用，可以使用它来建立两个不同origin的窗口之间的双向通信。

### 示例
在本地建立app.html文件，内嵌一个位于云服务器上的跨域iframe页面page.html，iframe中包含一个按钮，在示例中，实现iframe页面和本地页面的跨文档通信。

> page.html
 
 
``` html
<body>
    <div class="container">
        
        <div class="column-2">
            
            <div class="button">
                <button class="order-but" onClick="orderNow()">立即预订</button>
            </div>
        </div>
    </div>
</body>
```
``` js
<script type="text/javascript">

var orderNow = function() {  
    console.log(window.parent)
    window.parent.postMessage('order', '*')
}

window.addEventListener('message', handleMessage, false);

function handleMessage(event) {
    console.log(event);
    var origin = event.origin || event.originalEvent.orgin;
    if(origin ! == 'http://example.org:8080') return;
}
</script>
```
 
在page.html中，点击按钮触发点击事件，在点击事件的方法中，将会执行postMessage向iframe的父页面发送一条消息。在这里，window.parent是iframe的父级页面，字符串'order'为将要发送的消息，目标窗口域名为通配符'*'，表示任何域名。
同时，在window上绑定事件监听器，监听其他页面发送的消息。

> app.html


``` html
<body>
    <div class="container">
        <iframe src="http://123.206.14.146/postmessage.page.html" frameborder="0" id="external-frame" width="100%" height="500px"></iframe>
    </div>
    <script>
        window.addEventListener('message', handleMessage, false)

        function handleMessage(event) {
            console.log(event);
            var origin = event.origin;
            window.frames[0].postmessage('ok', origin);
        }
    </script>
</body>
```

app.html位于本地localhost，在此文件中，嵌入位于云服务器上的iframe页面page.html,iframe向app.html发送消息，那么app.html该如何接收消息呢？在window上监听message事件。当接收到消息时，使用postMessage对iframe再发送一个回复消息。

在WebStorm中预览app.html，点击iframe中的按钮，控制台打印出如下信息。
 
从控制台打印出的log可以看到，MessageEvent主要有三个属性：
![image](http://img.yanyuanfe.cn/console.png)

data： 传递的消息；

origin： 发送消息的源，由协议+主机名+端口号组成；

source： 发送消息的窗口对象；

在上面的示例，使用postMessage简单地实现了跨文档的消息传递。
 
### 注意

postMessage是一个很实用的功能，但是使用不当也会暴露许多安全问题，所以在使用的时候需要注意：

1、 otherWindow.postMessage（message, targetOrigin, [transfer]）;
targetOrigin参数最好不要使用通配符"*",应该使用受信任的域名；

2、处理message事件时，需要对发送消息的源event.origin进行校验 ，避免产生安全问题。


``` js
window.addEventListener('message', handleMessage, false);

function handleMessage(event) {
    console.log(event);
    var origin = event.origin || event.originalEvent.orgin;
    if(origin ! == 'http://example.org:8080') return;
}

```

 
### 最后
HTML5的postMessage不仅仅可以实现跨文档通信，跨域通信、多窗口通信、当前页和新窗口之间的通信都可以用如此简单的方式实现。如此强大的功能，那么其兼容性如何呢？如下：
![image](http://img.yanyuanfe.cn/caniuse.png)
 
可以看出，postMessage已经支持大部分浏览器，需要注意的是，在IE8、9和Firefox6及其以下版本只支持字符串作为传递的消息数据。
