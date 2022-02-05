---
title: CSS3实现文字流光渐变动画
date: 2017-03-18 12:29:46
categories: 前端
banner: http://img.yanyuanfe.cn/QQ%E6%88%AA%E5%9B%BE20170318121942.png
tags:
	- CSS
---

> 来自百度前端技术学院的实践任务：有趣的鼠标悬浮模糊效果，参考：http://ife.baidu.com/course/detail/id/14，用CSS3实现了一下，顺便复习下CSS的基础。

![image](http://img.yanyuanfe.cn/QQ%E6%88%AA%E5%9B%BE20170318121942.png)

<!--more-->


### 主要实现

- 最终效果：  

http://yanyuanfe.cn/IFE-task/IFE2017/task7/index.html
- 实现文字的流光渐变动画
- 背景图需要进行模糊处理
- 实现按钮边框的从中间到两边扩展开
下面来一步步实现这些效果。

### 页面结构和基础样式

index.html
``` html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<link rel="stylesheet" type="text/css" href="style.css">
	<title>有趣的鼠标悬浮模糊效果</title>
</head>
<body>
<div class="container">
    <div class="box-wrapper">
        <div class="box">
         <h1 class="title">HTML5、CSS3、ES6</h1>
        </div>
    </div>
  
   <img class="img" src="./404_1x.png" alt="404">
</div>
</body>

</html>
```
style.css


``` css
* {
  margin:0;
  padding:0;
  box-sizing: border-box;
}

.container {
  margin:50px auto;
  width:400px;
  height:300px;
  position: relative;
}

.box-wrapper {
  position: absolute;
  text-align: center;
  width:100%;
  height:100%;
  padding:30px;
}

.box{
  position: relative;
  width:100%;
  height:100%;
  padding-top: 80px;
}
```
现在我们已经实现了页面的基本布局和样式，container容器固定宽高，在页面中水平居中，包含box-wrapper和img两个盒子，box-wrapper充满container，绝对定位使文字悬浮与图片上面。  
现在的效果如下：  

![image](http://img.yanyuanfe.cn/QQ%E6%88%AA%E5%9B%BE20170318121705.png)

### 实现文字的背景渐变

实现文字的流光渐变我们需要用到CSS3的**linear-gradient**绘制背景渐变效果，从左到右，这里我们需要设置0～100%的五种颜色，其中0～40%的色值与50%～100%的色值相同，并且0和100%的色值相同，这样是为了让流光效果如丝般顺滑。  

**CSS linear-gradient()** 函数的第一个参数描述渐变线的起始点位置。它包含两个关键词：第一个指出垂直位置left or right，第二个指出水平位置top or bottom。关键词的先后顺序无影响，且都是可选的。  

to top, to bottom, to left 和 to right这些值会被转换成角度0度、180度、270度和90度。其余值会被转换为一个以向顶部中央方向为起点顺时针旋转的角度。渐变线的结束点与其起点中心对称。


``` css
.title {
  background-image: -webkit-linear-gradient(left,#D81159, #E53A40 10%, #FFBC42 20%, #75D701 30%, #30A9DE 40%,#D81159 50%, #E53A40 60%, #FFBC42 70%, #75D701 80%, #30A9DE 90%,#D81159);
  
}
```
现在可以看到文字背景的渐变效果已经出来了，但是我们需要的是文字颜色渐变的啊。

现在我们需要用到CSS3的background-clip属性。  

**CSS3 background-clip:border-box|padding-box|content-box|text**
 

用于指定background是否包含content之外的border,padding。默认值为border-box，即background从包含border在内的地方开始渲染，IE的默认表现也等同于border-box。

**border-box**:背景裁剪（背景从border(即包括border在内)开始绘制（渲染））;

**padding-box**:背景裁剪（背景从padding(即包括padding在内)开始绘制）;

**content-box**:背景裁剪（背景从content(即内容部分)开始绘制）;
**text**:背景裁剪（将背景裁剪作为文本的填充色);
（当前只有webkit内核浏览器支持）。  
在这里，我们将使用 -webkit-background-clip:text;这个属性来使用文字作为裁剪区域向外裁剪，此时文字的颜色将覆盖在渐变色之上。  

``` css
.title {
  background-image: -webkit-linear-gradient(left,#D81159, #E53A40 10%, #FFBC42 20%, #75D701 30%, #30A9DE 40%,#D81159 50%, #E53A40 60%, #FFBC42 70%, #75D701 80%, #30A9DE 90%,#D81159);
  -webkit-background-clip: text;
}
```

为了让文字显示渐变色，我们需要将文字颜色变为透明，此时可以使用**-webkit-text-fill-color: transparent**; 或者是** color: transparent**;
将字体颜色设置成透明，这样就能将渐变色文字显示出来了。


``` css
.title {
  background-image: -webkit-linear-gradient(left,#D81159, #E53A40 10%, #FFBC42 20%, #75D701 30%, #30A9DE 40%,#D81159 50%, #E53A40 60%, #FFBC42 70%, #75D701 80%, #30A9DE 90%,#D81159);
  -webkit-background-clip: text;
  color:transparent;
}
```
现在的效果如下。  

![image](http://img.yanyuanfe.cn/QQ%E6%88%AA%E5%9B%BE20170318105511.png)  

### 流光动画原理
流光动画的原理是将文字的背景色的宽度拉长至原来的两倍，而在**background-image**中我们设置了两份相同的颜色组，将背景拉长之后便只显示一份颜色组，超出背景宽度的颜色组通过动画改变背景色的位置**background-position**来实现流光效果。

``` css
.title {
  background-image: -webkit-linear-gradient(left,#D81159, #E53A40 10%, #FFBC42 20%, #75D701 30%, #30A9DE 40%,#D81159 50%, #E53A40 60%, #FFBC42 70%, #75D701 80%, #30A9DE 90%,#D81159);
  color:transparent;
  -webkit-background-clip: text;
  background-size: 200% 100%;
}
```
下面通过CSS3的**animation**来实现帧动画。


``` css
.title {
  background-image: -webkit-linear-gradient(left,#D81159, #E53A40 10%, #FFBC42 20%, #75D701 30%, #30A9DE 40%,#D81159 50%, #E53A40 60%, #FFBC42 70%, #75D701 80%, #30A9DE 90%,#D81159);
  color:transparent;
  -webkit-background-clip: text;
  background-size: 200% 100%;
  animation: flowlight 5s linear infinite;
}


@keyframes flowlight {
  0% {
    background-position: 0 0;
  }
  100% {
    background-position: -100% 0;
  }
}
```
现在我们已经实现了流光动画的效果，是不是很酷，鼠标悬浮的效果等会一起实现。

### 实现按钮边框的从中间到两边扩展开

这个效果主要使用了伪元素**::before**、**::after**来实现容器的边框，::before设置容器的左右边框，::after设置容器的上下边框，伪元素边框使用绝对定位，父容器box使用相对定位，伪元素边框位置分别定位到相对父元素的top和left的50%，通过改变其位置和宽高来实现边框从中间到两边扩展的动画效果。


``` css

.box::before {
  content:'';
  border:3px solid #fff;
  border-width: 0 3px;
  position: absolute;
  width:100%;
  height:0;
  top:50%;
  left:0;
  box-sizing: border-box;
}


.box::after {
  content:'';
  border:3px solid #fff;
  border-width: 3px 0;
  position: absolute;
  width:0;
  height:100%;
  top:0;
  left:50%;
  box-sizing: border-box;
}
```
实现边框扩展的的动画效果，并加上过渡效果。  

``` css

.container:hover .box::before{
  height:100%;
  top:0;
}

.container:hover .box::after {
  width: 100%;
  left:0;
}

.box::before {
  transition: all .8s;
}

.box::after {
  transition: all .8s;
}

```
### 背景图的模糊处理

从最终实现我们可以看到，鼠标悬浮时背景图会出现模糊效果，这样的模糊效果我们可以通过CSS3的滤镜属性**filte**r来实现。  

**filter**：CSS滤镜属性，可以在元素呈现之前，为元素的渲染提供一些效果，如模糊、颜色转移之类的。滤镜常用于调整图像、背景、边框的渲染。

在这里我们需要设置**filter**的**blur**来实现对象的模糊效果。


``` css
.container:hover .img {
   filter: blur(2px);
   -webkit-filter: blur(2px);
   -moz-filter: blur(2px);
   -ms-filter: blur(2px);
}
```
然而，加上之后我们的文字和边框哪儿去了？这是因为图片把文字和边框覆盖了，这里我们需要设置**box-wrapper**容器的**z-index**属性，来凸显各个容器的层级。


``` css
.box-wrapper {
  position: absolute;
  text-align: center;
  width:100%;
  height:100%;
  padding:30px;
  z-index: 99;
}
```
这样，背景模糊的效果就实现了。


### 优化细节

细心的你可能注意到，在鼠标悬浮的时候，文字会一个上移的效果还没实现，下面来实现它。  
文字上移的效果我们通过CSS3的变换属性来实现，默认我们将文字向下位移，鼠标悬浮时我们再将其置0就OK了。


``` css
.title {
  background-image: -webkit-linear-gradient(left,#D81159, #E53A40 10%, #FFBC42 20%, #75D701 30%, #30A9DE 40%,#D81159 50%, #E53A40 60%, #FFBC42 70%, #75D701 80%, #30A9DE 90%,#D81159);
  color:transparent;
  -webkit-background-clip: text;
  background-size: 200%;
  transform: translate(0,20px);
  animation: flowlight 5s linear infinite;
}
```
通过**translate**将文字向下位移20px。

``` css
.container:hover .title{
  transform: translate(0);
}
```
鼠标悬浮时取消位移。 

最后，我们的显示隐藏效果还没实现。这里，我们通过设置透明度**opacity**来实现显示隐藏，设置**transtion**来实现过渡效果。

``` css
.box{
  opacity: 0;
  transition: opacity .8s;
}


.title {
  transition: transform .8s,opacity .8s;
}

.container:hover .box{
  opacity: 1;
}

.container:hover .title{
  opacity: 1;
}

```

现在，我们已经实现了所有的动画效果，看起来还是不错吧，CSS3完成动画效果的能力还是很强大的吧。

### 总结

本文用到的CSS3属性：

- linear-gradient
- webkit-background-clip
- -webkit-text-fill-color
- animation
- transform
- transition
- filter

<div class="tip"> 由于CSS3的兼容性，在完成此效果时，只在Chrome浏览器进行测试，如需兼容其他浏览器，请自行加上浏览器前缀。</div>

源码：https://github.com/YanYuanFE/IFE-task/tree/master/IFE2017/task7  

demo：http://yanyuanfe.cn/IFE-task/IFE2017/task7/index.html



