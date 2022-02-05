---
title: 你不知道的CSS(一)
date: 2016-07-10 17:21:08
categories: 前端
banner: http://img.yanyuanfe.cn/css%20module.png
tags:
	- CSS

	
---



> “像是语言的进化，它让CSS更强大”。                 
— Nicole Sullivan

![image](http://img.yanyuanfe.cn/css%20module.png)

<!--more-->


   CSS（Cascading Style Sheets）指层叠样式表，我相信没有前端开发者不认识它，CSS在网页中负责样式层的展现，它只是网页样式的一种描述方法，不能算是编程语言。因此，众多的开发者都忽视了CSS的能力，认为其太过简单，是一种机械的工作。后来在CSS的发展历程中，出现了越来越高级的应用。比如CSS 3D（硬件加速）、CSS Sprites、媒体查询等等。同时，CSS也在朝着编程语言的方向发展，像LESS、SASS的出现，都表示着，前端开发者再也不能小觑CSS的力量，在某些地方，使用CSS时的性能不输于JavaScript。在这里，我想谈一谈你不知道的CSS，诸如OOCSS、SMACSS、ACSS、BEM。它们是CSS的设计模式，或者叫最佳实践。
   
## 一. OOCSS

  OOCSS（Object-Oriented CSS）即为面向对象的CSS，关于什么是面向对象，乔布斯曾经有一句经典的解释：
    
>  对象就像人一样，也是活生生的生命。他们有知识，知道怎么完成任务；他们有记忆，可以把发生的事情记下来。你和他们不在低层次上交互，而是像我们现在的对话一样，在一个高度抽象的层面上交互。
——乔布斯

从这里这里看到，把所有的这些复杂流程都封装在对象内部，对象对外呈现接口，我们通过接口来使用对象，对象的接口是高层次的，抽象的。

这样，你应该对OOCSS有一定的了解了，OOCSS是一种编写CSS代码的方法，它将页面可重用元素抽象成一个类，用Class加以描述，而与其对应的HTML即可看成是此类的一个实例。OOCSS的核心思想是使用最简单的方式编写干净整洁的CSS代码，从而使CSS代码具有可重用性、可维护性、可扩展性。其关键在于在页面中识别、创建和模块化可重用的对象，并在页面中其他所有需要的地方重用，并扩展其功能。

**类：**

``` css
.container{
    overflow:hidden;
    _overflow:visible;
    zoom:1;
    margin:10px;
}
.container .text{
    display:table-cell;
    zoom:1;
}
.container .img{
    float:left;
    margin-right:10px;
}
.container .img img{
    display:block;
}
```
**实例：**


``` html
<div class="container">
	<a href="" class="img">
		<img src="myimg.png" alt="">
	</a>
	<div class="text">
		Cascading Style Sheets
	</div>
</div>
```






总体来讲，使用OOCSS有这些好处：
1. 加强代码复用以便方便维护。
2. 减小CSS体积。
3. 提升渲染效率。
4. 使用组件库思想、可共用栅格布局、减少选择器、方便扩展。


 那么，该如何使用使用面向对象的CSS？下面通过实践来编写OOCSS代码。

#### 1. 不要直接定义子节点，应把共性声明放到父类。




``` css

.container .inner{...}     //container中的子元素

.inner{...}    //不是很建议的声明
```
在这个demo里面，container定义了很多共有的属性，如color、font-size，应使用.container .inner定义特性声明。将类的属性（内部的子节点）通过联合定义类的形式（形如 .类名 .属性/子节点{}）进行封装，即子节点必须用对象名作为开头，并可通过给根元素设置多个class来实现继承（class="class1 class2"）。

#### 2. 结构和皮肤相分离。


``` html
<div class="container skin">
	<div class="inner">
		<h3>Text</h3>
	</div>
</div>
```
所谓结构和皮肤相分离即是将页面的结构布局和皮肤样式分别又两个class分别控制，在demo里面，container类为控制结构的class，用于控制页面布局结构的样式，如展现元素的位置、浏览器bug和所有比较复杂的问题；skin类为控制皮肤的class，用于控制元素皮肤样式，处理比较简单的问题，如color、border、background等等。
多个皮肤可以重用一种结构，使得复杂的结构可以被重用，皮肤的修改工作变得很简单。

#### 3. 容器和内容相分离。


``` html
<div class="news">
	<h3>今日新闻</h3>
	<ul>
		<li>...</li>
		....
	</ul>
</div>
```
.
```
.news ul{}
.news li{}
```
子元素ul依赖了父容器。


``` html
<div class="news">
	<h3>今日新闻</h3>
	<ul class="list">
		<li>...</li>
		....
	</ul>
</div>
```

``` css
.news{}
.news .list{}
```
子元素list解除了与容器相关的依赖关系，便于重用，子元素可以从一个容器转移到任意容器。以此将容器和内容独立出来。

#### 4. 抽象出可重用的元素，建好组件库，在组件库内寻找可用的元素组装页面。

做一个页面就像建造一座房子一样，造房子之前我们先将房子的每一部分造好，然后一点一点将各个部分组装起来。每一个部分可看作页面的一个个组件，做页面时，需要什么组件就去组件库里面寻找相应的元素，同时页面的每个组件元素又是可重用的。

#### 5. 向你想要扩展的对象本身增加Class而不是它的父节点

``` html
<div class="modal">
	<div class="inner">
	    <h3>...</h3>
		<p>...</p>
	</div>
	
</div>
```

``` css
.modal .inner{...}

.modal h3{...} //h3本身就是可重用对象
```

``` html
<div class="modal">
	<div class="inner">
        <h3 class="list">...</h3>
    	<p>...</p>
	</div>
		
</div>
```

``` css

.modal{...}
.list{...}//如果扩展h3，直接在其本身增加class，而不是通过父节点的层叠来扩展
```
在同一个对象内总是使用层叠关系，一个对象内的所有子节点必须用这个对象的class名作为开头，比如：.modal .inner{...}。对象之间可以嵌套，但不要层叠。
当需要为基础对象扩展其功能的时候，应该根据上下文，通过向父节点增加更多的类名，而不是通过父节点的层叠关系来扩展。


#### 6. 对象应保持独立性


``` html
<div class="modal">
	<div class="mybox">...</div>
</div>
```


``` css
.mybox{...}
.modal .mybox{...}
```

``` html
<div class="modal mybox">
		...
</div>
```

``` css
.mybox{...}
.modal{...}

```
每个对象应该保持其独立性，每个class类名应保持自己的结构，保证自己的功能。自己的问题自己解决，而不是让父母出面。

#### 7. 避免使用ID选择器，权重太高，无法重用。

在CSS选择器的优先级中，ID选择器的权重仅次于内联样式，而且ID选择器在一个页面中只能出现一次，无法进行重用，容易和JS用的id混淆，不便于区分，因此应该避免使用ID选择器。

#### 8. 避免位置相关的样式。

``` css
#header .linkList{...}
#footer .linkList{...}
#header ul{...}
.news ul{...}
.weather h3{...}
.tab h4{...}
```
而应该写成

``` css
.linkList{...}
.mainNav{...}
.subNav{...}
h3,.h3{...}
```
当对象的class不能满足其要求时，通过class直接在对象上面进行扩充，避免通过层叠关系来改变对象的显示。
 
``` css
#weather h3{color:red;}
#tab h3{color:blue}
```
在这里，由于没有定义全局的h3，所以每一个新的模块里面的h3是没有样式的，开发者必须针对同样的样式写许多的CSS。


``` css
h3{color:black}
#weather h3{color:red}
#tab h3{color:blue}
```
这里就不一样了，为h3定义了全局的颜色样式，weather和tab重写了默认的h3，而weather和tab里面的h3也无法在外面使用。



#### 9. 保证选择器相同的权重
    

``` html
<div class="classA classB"></div>
```

``` css
.classA{color:red;}
.classB{color:blue;}
```
在HTML中，class类名出现的先后顺序与优先级无关，在CSS样式表中的先后顺序与优先级有关。

**推荐写法：**


``` html
<div class="media mediaExt attribution"></div>
```

``` css
.media{...}
.mediaExt{...}
.attribution{...}
```


``` css
.modal .hd{...}
.modalExt .hd{...}
```


而不是

``` css
html body .modal div .hd{...}
```
样式表中后出现的覆盖掉前面的，保证选择器有相同的权重。

``` css
.mod .hd{color:red;_zoom:1;}
.weather .hd{...}
```

而不是


``` css
.mod .hd{...}
.ie .mod .hd{...}
.weather .hd{...}
```
这样是为了有节制地使用hack，不要让hack改变你的权重。






#### 10. 类名要简短、清晰、语义化，OOCSS的名字并不影响HTML的语义化

**Class命名的几个目标：**

简短——每一个字节都很重要，尽可能简短

清晰——根据名称可以很快知道它的功能

语义——对象的外观不重要，重要的是它是什么，它的功能是什么

大众化——过于特殊的名字会减少它的应用场景或导致语义化的class以非语义化的方式使用

使用OOCSS不妨碍HTML标签的语义，在具体的SEO场景中，搜索引擎主要看HTML代码是否语义化。但也在语义和可维护性之间做了权衡。




