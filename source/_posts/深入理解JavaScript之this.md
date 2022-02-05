---
title: 深入理解JavaScript之this
date: 2016-07-14 16:50:13
categories: 前端
banner: http://img.yanyuanfe.cn/js.png
tags:
	- JavaScript
---



> JavaScript非常重要。这并不总是如此，但现在确实如此

![image](http://img.yanyuanfe.cn/js.png)

<!--more-->

this是JavaScript中的一个很特别的关键字，同时，它也是JavaScript中最复杂的机制之一，重要性同闭包、原型不相伯仲。如果你缺乏对this的清晰认识，this的指向足以让你眼花缭乱，这完全就是一种魔法。

### this是什么

当一个函数被调用时，除了传入了函数的显式参数以外，名为this的隐式参数也被传入了函数。this引用了与该函数调用进行隐式关联的一个对象，被称之为函数上下文。函数上下文包含了函数在哪里被调用（调用栈）、函数的调用方式、传入的参数等信息。this在函数运行时进行绑定，并不是在编写时绑定，它的上下文取决于函数调用时得各种条件。this的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式，而this则指的是调用函数的那个对象。

现在，我们来看看在具体的调用方式中this是怎样的。

### 一. 作为函数进行调用 

当一个函数作为函数进行调用时，是区别于其他调用机制：方法、构造器以及apply、call。

使用这种方式进行调用通常在函数上使用了()操作符，使用了()操作符的表达式并没有将函数作为对象的一个属性。
下面是一个示例：

``` js
function foo(){
	console.log(this.a);
}
var a=2;

foo();//2
```
如你所见，a是声明在全局作用域中的变量，a也是全局对象的一个同名属性。当foo()被调用时，this.a被解析称为了全局变量a，因此this指向全局对象，函数的上下文是全局上下文——window对象。

如果使用严格模式(strict mode)，则this并不能指向全局对象，this此时将会指向undefined：



``` js
function foo(){
	"use strict";
	console.log(this.a);
}
var a=2;

foo();
//TypeError: Cannot read property 'a' of undefined(…)
```


### 二. 作为方法进行调用

当一个函数被赋值为对象的一个属性，并使用引用该函数的这个属性进行调用时，那么函数就是作为对象的一个方法进行调用的。

示例如下：

``` js
var obj={
	a:2
};
obj.foo=function(){
	console.log(this.a);
};
obj.foo();//2

```

如果你对面向对象了解的话，你肯定知道，一个方法所属的对象在该方法体内可以以this的形式进行引用。在这里，将函数作为对象的一个方法进行调用时，该对象就变成了函数的上下文，并且在函数内部可以以this参数的形式进行访问。

请看下面示例：


``` js
function foo(){
	
	console.log(this.a);
}


var obj={
	a:2,
	foo:foo
};
var bar=obj.foo;

var a="two";
bar();//"two"
```
在这里，虽然bar是obj.foo的一个引用，但是实际上，它引用的是foo函数本身，因此此时的bar()其实是一个不带任何修饰的函数调用，因此this指向全局对象。当使用严格模式时，this指向undefined。

### 三. 作为构造器进行调用

将函数进行构造器(constructor)进行调用，我们要在函数进行调用前使用new关键字。 

``` js
function foo(){
	
	return this
}
new foo();
```

将函数作为构造器进行调用，或者说使用new来调用函数，会自动执行下面的操作。

- 创建一个新的空对象。
- 传递给构造器的对象是this参数，从而成为构造器的函数上下文。
- 如果没有显式的返回值，新创建的对象则作为构造器的返回值进行返回。

请看下面代码：


``` js
function foo(a){
	
	this.a=a;
}
var bar=new foo(2);

console.log(bar.a);
```

作为构造函数调用时foo()时，我们会构造一个新对象并把它绑定到foo(...)调用中的this上。

### 四. 使用apply()/call()方法进行调用

在函数调用的时候，JavaScript为我们提供了一种方式，可以显式指定任何一个对象作为其函数上下文。JavaScript的每个函数都有apply()和call()方法，它们的功能相同，仅仅是传入参数不同。

通过函数的apply()方法来调用函数，我们要给apply()传入两个参数：一个是作为函数上下文的对象，另外一个是作为函数参数所组成的数组。call()方法的使用方式类似，唯一不同的是，给函数传入的参数是一个参数列表，而不是单个数组。

请看示例：

``` js
function foo(){
	console.log(this.a);
}

var obj={
	a:2,
};
foo.call(obj);
```
在这段代码中，通过foo.call(...),我们在调用foo时强制把它的this绑定到obj上。

### 总结

如果要判断一个运行中函数的this绑定，就需要找到这个函数的直接调用位置，然后根据其调用方式来判断this的绑定对象。
1. 作为构造器进行调用，this指向新创建的对象。
2. 使用call/apply方法调用时，this指向指定的对象。
3. 作为方法进行调用时，this指向拥有该方法的对象
4. 作为普通函数进行调用时，this指向全局对象window，严格模式下指向undefined。











