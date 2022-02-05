---
title: 深入理解JavaScript之变量提升
date: 2016-07-16 20:29:10
categories: 前端
banner: http://img.yanyuanfe.cn/js.png
tags:
	- JavaScript
---
> Javascript的变量声明具有Hoisting机制(变量提升)，JavaScript引擎在执行的时候，会把所有变量的声明都提升到当前作用域的最前面。

![image](http://img.yanyuanfe.cn/js.png)

<!--more-->

### 从一个栗子说起
在学习JavaScript的时候，你可能会陷入一个误区，以为JavaScript的代码在执行时是一行一行由上到下依次执行的。但是，在实际上，这并不完全正确。

下面是一个简单的示例：


``` js
a=2;

var a;

console.log(a);
```
你可能会以为会输出undefined，因为var a声明在a=2之后，变量被重新赋值，因此a被赋予默认值undefined。但是，你错了，真正的输出结果是2。

再看一个示例：



``` js
console.log(a);

var a=2;
```
看了上一个例子的不寻常的特点，你可能会认为这里也会同样的输出2。还有人可能会以为，由于变量a在使用前没有先进行声明，因此会抛出如下异常：

``` js
Uncaught ReferenceError: a is not defined(…)
```
结局充满意外，两种猜测都不正确，真正的输出结果是undefined。

### 揭开秘密

> 当秘密被揭开的时候，和各种各样的魔术把戏一样，几乎令人失望。

在JavaScript中，当变量被声明时，声明会被提升到它所在函数的顶部，并被赋予undefined值，而初始化仍旧在原来的地方。这就使得在函数的任意位置声明的变量存在于整个函数中，尽管在赋值之前，它的值一直是undefined。

当你看到var a=2;时，可能会认为这是一个声明。但JavaScript实际上会将其看成两个声明：var a;和a=2;。第一个声明是在编译阶段进行的。第二个赋值声明会被留在原地等待执行。

第一个示例将会以如下形式处理：

``` js
var a;

a=2;

console.log(a);
```
结果不言而喻，其中第一部分是编译，第二部分是执行。

同样的，相信你已经知道第二个示例是如何处理的了吧：

``` js
var a;

console.log(a);

a=2;

```
因此，这个过程就好像变量的声明从它们在代码中出现的位置被“移动”到了最上面，这个过程叫做提升。

### 函数提升

上面我们讲了JavaScript的变量提升，除了变量提升，JavaScript还有函数提升。

下面是一个示例：


``` js
foo();

function foo(){

	console.log(a);//undefined
	var a=2;
}
```
理解了变量提升的原理，这个示例的结果也就明显了。foo函数的声明被提升了，因此第一行代码可以正常执行。

在JavaScript中，每个作用域都会进行提升操作。前面的代码都是简化在全局作用域中进行，而这里的foo(...)函数自身也会在内部对var a进行提升（注意：并不是提升到整个程序的最上方）。因此这段代码实际上会按照如下形式进行处理：

``` js


function foo(){
    var a;
    
	console.log(a); //undefined
	
	a=2;
}

foo();
```
请注意，函数声明会被提升，但是函数表达式却不会被提升。

``` js
foo();

var foo=function bar(){
    //...
};
```
输出结果：

``` js
Uncaught TypeError: foo is not a function(…)
```
而不是：

``` js
Uncaught ReferenceError
```
在这段代码中，变量标识符foo()被提升并分配给所在的作用域（此处为全局作用域），因此foo()不会导致ReferenceError。但是foo并没有赋值（如果它是一个函数声明，那么就会赋值）。foo()由于对undefined值进行函数调用而导致非法操作，因此抛出TypeError异常。

同时请注意，即使是具名的函数表达式，名称标识符在赋值之前也无法在所在作用域中使用，如下：

``` js
foo();//TypeError

bar();//ReferenceError

var foo=function bar(){
    //...
};
```
这段代码经过提升后变为如下形式：

``` js
var foo;

foo();//TypeError

bar();//ReferenceError

foo=function bar(){
    //...
};
```
### 优先提升

在JavaScript中，变量声明和函数声明都会被提升，但是，需要注意的一个细节是，函数会优先被提升，然后才是变量。

下面是一个示例：

``` js

foo();

var foo;

function foo(){
    console.log(1);
};

foo=function (){
    console.log(2);
};
```
根据上面的结论我们知道结果会输出1。这段代码会被引擎理解为如下形式：

``` js


function foo(){
    console.log(1);
};

foo(); //1

foo=function (){
    console.log(2);
};
```
在这段代码中，var foo虽然出现在function foo()...的声明之前，但是它是重复声明，会被忽略，因为函数声明会被提升到普通变量之前。

虽然重复的变量声明会被忽略，但是出现在后面的函数声明却可以覆盖前面的。
``` js
foo(); //3

function foo(){
    console.log(1);
};

foo=function (){
    console.log(2);
};

function foo(){
    console.log(3);
};

```
### 结论

当你理解了变量提升，那么这对于Javascript编码意味着什么？最重要的一点是，总是用var定义你的变量。而且，对于一个名称，在一个作用域里面永远只有一次var声明。如果你这么做，你就不会遇到作用域和变量提升问题。

ECMAScript参考文档关于作用域和变量提升的部分：
> 如果变量在函数体类声明，则它是函数作用域。否则，它是全局作用域（作为global的属性）。变量将会在执行进入作用域的时候被创建。块不会定义新的作用域，只有函数声明和程序（译者以为，就是全局性质的代码执行）才会创造新的作用域。变量在创建的时候会被初始化为undefined。如果变量声明语句里面带有赋值操作，则赋值操作只有被执行到的时候才会发生，而不是创建的时候。