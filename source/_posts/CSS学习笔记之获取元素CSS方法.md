---
title: CSS学习笔记之获取元素Style方法
date: 2016-07-20 20:30:53
categories: 前端
banner: http://img.yanyuanfe.cn/css.png
tags:
	- CSS

---

> CSS 已经如此壮大,以致于一个普通人已经无法把它完整地装进自己的头脑了。

![image](http://img.yanyuanfe.cn/css.png)

<!--more-->


### 从CSS样式说起

最近一直在使用原生JavaScript，在获取元素的样式的时候发现了一些CSS的小秘密，关于CSS的style、currentStyle、getComputedStyle。什么？后面两个你没见过！！！在看了这篇文章之后你就知道了。

在讲干货之前，我想我们应该来复习下关于CSS的基础。

关于这个层叠样式表（CSS），在HTML文档里有三种表现形式。
1. 内联样式表

内联样式表是指将CSS样式编码写在HTML标签中，如下所示：

``` html
<h1 style="font-size:16px;color:#000FFF">
内联CSS样式。
</h1>
```
内联样式表由HTML元素的HTML元素的style支持，只需将CSS代码用分号隔开写在style=""之中。这是最基本的形式，但是它没有实现表现与内容分离且不能灵活的控制多个页面所以我们很少使用。
2. 内部样式表

内部样式表与内联样式表相似都是把CSS代码写在HTML页面中，稍微不同的是前者可以将样式表放在一个固定的位置，如下所示：

``` html
<html>
<head>
<title>内部样式表</title>
<style type="text/css">
   h1{font-size:16px;
      color:#000FFF
      }
</style>
</head>
<body>
<h1>内部CSS样式。</h1>
</body>
</html>
```
内部样式表编码是初级的应用形式，不能跨页面使用所以不适合使用。


3. 外部样式表

外部样式表是CSS应用中最好的一中形式，它将CSS样式代码单独放在一个外部文件中，再由网页进行调用。多个网页可以调用一个样式文件表，这样能够实现代码的最大限度的重用及网站文件的最优化配置，如下所示：


``` html
<html>
<head>
<title>外部样式表</title>
<link rel="stylesheet" rev="stylesheet" href="style.css">
</head>
<body>
<h1>外部CSS样式。</h1>
</body>
</html>
```
在style.css中的代码为

``` css
@charset "UTF-8";

h1{
    font-size:16px;
    color:#000FFF
}
```
我们在<head>中使用了<link>标签来调用外部样式表文件。将link指定为stylesheet方式，并使用了href="style.css"指明样式表文件的路径便可将该页面应用到在style.css中定义的样式。

现在你已经知道了CSS的三种形式，那么它们和在JavaScript中获取元素又又怎样的关系呢？下面揭晓答案。

### 获取元素样式方法之style

style是通过JavaScript语言来获取CSS的样式的一种最基本的方法，也是最为人所知的方法，它的使用方法是element.style.attr，可读可写。需要注意的是（前方高能），style只能获取元素的内联样式，内部样式和外部样式使用style是获取不到的。
神马？（童话里都是骗人的）。下面是验证代码：

``` html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>CSS样式</title>
    <link rel="stylesheet" type="text/css" href="css/style.css">
    
</head>
<style type="text/css">
  #container{
    width: 300px;
  }
  h1{
    font-size: 13px;
  }
</style>

<body>
   <div id="container" style="width: 400px;">
    <h1 style="font-size: 14px;">CSS学习笔记之获取元素样式</h1>
   </div>
  
  <script src="js/task.js"></script>
</body>

</html>
```
**style.css**

``` css
#container{
	width: 500px;

}
h1{
	font-size: 16px;
}
```
**task.js**

``` js
window.onload = function(){
  var oDiv= document.getElementById('container');
  var h1=document.getElementsByTagName("h1")[0];
  console.log(oDiv.style.width); //400px
  console.log(h1.style.fontSize); //14px
};
```
在上面的代码中，我们分别使用CSS的三种形式对container和h1设置不同的样式，通过js来获取具体值，最后得到的都是内联样式的值。而最终在网页中生效的也是内联样式的值，也表明三种方式中内联样式的优先级最高。

### 获取元素样式方法之currentStyle

说到currentStyle，表示很遗憾，它只兼容IE浏览器，使用currentStyle返回的是元素当前应用的最终CSS属性值（包括外联CSS文件，页面中嵌入的style属性等）。
currentStyle的使用方法是element.currentStyle["attr']或者element.currentStyle.attr。
我们对js代码进行修改如下：


**task.js**

``` js
window.onload = function(){
  var oDiv= document.getElementById('container');
  var h1=document.getElementsByTagName("h1")[0];
  console.log(oDiv.currentStyle.width); //400px
  console.log(h1.currentStyle.fontSize); //14px
};
```
通过测试，IE显示正常，Edge浏览器报错，Chrome报错如下：

``` js
Uncaught TypeError: Cannot read property 'width' of undefined
```

### 获取元素样式方法之getComputedStyle

getComputedStyle是一个可以获取当前元素所有最终使用的CSS属性值。返回的是一个CSS样式声明对象([object CSSStyleDeclaration])，如果没有设置css样式，会读取默认样式，只读。它的使用方法是：
window.getComputedStyle(element, pseudoElt)["attr']或window.getComputedStyle(element, pseudoElt).attr。其中，pseudoElt表示如 :after,:before之类的伪类，如果不用伪类的话设置为null即可，不是必需参数。

getComputedStyle同currentStyle作用相同，但是适用于Firefox、Opera、Safari、Chrome。
关于兼容性，getComputedStyle在IE6-8是不支持的。
我们修改代码如下：

**task.js**

``` js
window.onload = function(){
  var oDiv= document.getElementById('container');
  var h1=document.getElementsByTagName("h1")[0];
  console.log(window.getComputedStyle(oDiv).width); //400px
  console.log(window.getComputedStyle(h1).fontSize); //14px
};
```
经过测试在Chrome和IE9以上正常，IE8报错：

```
对象不支持“getComputedStyle”属性或方法
```

### 获取元素样式方法之关于属性

说点有趣又涨姿势的，当我们使用js获取元素属性值的时候，传入的属性可能跟你想象的不太一样，比如上面驼峰式的fontSize。或者看看下面这个奇葩的浮动属性：
Chrome浏览器下是cssFloat和float，
FireFox浏览器下是cssFloat，IE7浏览器下则是styleFloat，而IE9浏览器下则是cssFloat和styleFloat都有，简直很变态啊有木有。

那我们再来谈谈获取元素属性的另一个方法，getAttribute，它可以获取CSS样式申明对象上的属性值（直接属性名称），可以访问CSS样式对象的属性。
使用getAttribute方法也不需要cssFloat与styleFloat的怪异写法与兼容性处理，但是属性名需要驼峰式写法。比如：


``` js
element.getAttribute("float");
```


``` js
element.getAttribute("fontSize");
```
### 封装getStyle

经过上面的了解，相信你已经可以写出一个封装获取样式的兼容写法，下面是一个简洁版：


``` js
//获取样式
function getStyle(obj, attr) {
	return obj.currentStyle ? obj.currentStyle[attr] : getComputedStyle(obj, false)[attr];
}
```



