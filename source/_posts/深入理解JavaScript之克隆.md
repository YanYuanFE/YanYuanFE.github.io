---
title: 深入理解JavaScript之克隆
date: 2016-09-24 13:43:41
categories: 前端
banner: http://img.yanyuanfe.cn/js001.png
tags:
	- JavaScript
---



> JavaScript中万物皆是对象，这显然是错误的。

![image](http://img.yanyuanfe.cn/js001.png)

<!--more-->

### 动态类型

JavaScript 是一种弱类型或者说动态语言。这意味着你不用提前声明变量的类型，在程序运行过程中，类型会被自动确定。这也意味着你可以使用同一个变量保存不同类型的数据。

### 数据类型

最新的 ECMAScript 标准定义了 7 种数据类型:

- 六种原始类型（Primitive）

- - Boolean
- - Null
- - Undefined
- - Number
- - String
- - Symbol（ECMAScript 6）


- 对象类型（Object）

在 Javascript 里，对象可以被看作是一组属性的集合。用对象字面量语法来定义一个对象时，会自动初始化一组属性。
而后，这些属性还可以被增减。属性的值可以是任意类型，包括具有复杂数据结构的对象。属性使用键来标识，它的键值可以是一个字符串或者符号值（Symbol）。
在对象中，又有两个特殊的对象——函数和数组。

从以上可知JavaScript的数据类型分为两类，而这两种类型在进行复制克隆时候是不同的。原始数据类型存储的是对象的实际数据，而对象类型存储的是对象的引用地址，对象的实际内容单独存放（为了减小数据开销通常存放在内存中）。此外，对象的原型也是引用对象，它将原型的方法和属性放在内存当中，通过原型链的方式来指向这个内存地址。

### 克隆
所谓对象克隆，就是对一个对象生成一个一模一样的对象，但是在JavaScript中，使用简单的复制语句所实现的是对原对象的一个引用，对新复制的对象的任何一个属性方法的改变，都将会影响原对象的属性方法。

根据克隆的特点，克隆有两种类型：
影子克隆（浅复制）：仅仅复制所考虑的对象，而不复制它所引用的对象。对于原始类型，为值传递，对象类型为引用传递。
深度克隆（深复制）：把要复制的对象所引用的对象都复制一遍。所有元素或属性均完全复制，与原对象完全脱离，最终所有对于新对象的修改都不会反映到原对象中。

### 影子克隆

1. 原始类型

1.1 字符串的克隆

``` js
var a="1";
var b=a;
b="2";
console.log(a);//1
console.log(b);//2
```
1.2 数值的克隆
``` js
var a=1;
var b=a;
b=2;
console.log(a);//1
console.log(b);//2
```
1.2 布尔值的克隆
``` js
var a=true;
var b=a;
b=false;
console.log(a);//true
console.log(b);//false
```
由上面的代码可知，原始类型采用普通的克隆方式就能得到正确的结果，因为原始类型中存储的是对象的真实数据。

2. 深度克隆

2.1 函数

函数虽然属于对象类型，但是函数的克隆通过浅复制即可实现。
``` js
var a=function(){
console.log(1)};
var b=a;
b=function(){
console.log(2)};
console.log(a());//1
console.log(b());//2
```
在这里，我们通过普通赋值的形式，就可以实现函数的克隆，并且不会影响之前的对象。原因是函数的克隆会在内容单独开辟一块空间，互不影响。

2.2 对象

在对JavaScript引用类型中的数组进行克隆的时候，直接赋值就会导致一些问题的发生了，请看如下代码。

``` js
var arr1=[1,2,3];
var arr2=arr1;
arr2[1]=1;
console.log(arr1);//[1, 1, 3]
console.log(arr2);//[1, 1, 3]
```
这种克隆方式称之为影子克隆、浅复制，但是这并不是我们想要的结果，我们想要让arr1的值不变。

1. 对象的深度克隆
之前在其他博客上看到过有人讨论数组的深度克隆，提出使用数组的slice方法和concat方法可以实现深度克隆，后来我查看MDN的文档时，看到关于这两个方法的解释：
- Array.prototype.slice()

slice() 方法会浅复制（shallow copy）数组的一部分到一个新的数组，并返回这个新数组。
slice不修改原数组，只会返回一个浅复制了原数组中的元素的一个新数组。原数组的元素会按照下述规则拷贝：
- 如果该元素是个对象引用（不是实际的对象），slice会拷贝这个对象引用到新的数组里。两个对象引用都引用了同一个对象。如果被引用的对象发生改变，则新的和原来的数组中的这个元素也会发生改变。
- 对于字符串、数字及布尔值来说（不是String、Number或者Boolean对象），slice会拷贝这些值到新的数组里。在别的数组里修改这些字符串或数字或是布尔值，将不会影响另一个数组。

如果向两个数组任一中添加了新元素，则另一个不会受到影响。
下面看一个代码示例：

``` js
var myCar = [{color: "red"},5, {wheels: 4}];
var newCar = myCar.slice(0);
newCar[0].color="purple";
newCar[1]=1;
newCar.push("engine");
console.log(myCar);//[{color: "purple"},5,{wheels: 4}]
console.log(newCar);//[{color: "purple"},1,{wheels: 4},"engine"]

```
从这段代码我们隐约可以知道，slice方法对数组元素是
字符串、数字及布尔值等原始类型会执行深拷贝，对引用对象类型，则会执行浅拷贝。

- Array.prototype.concat()
concat() 方法将传入的数组或非数组值与原数组合并,组成一个新的数组并返回.

concat 方法将创建一个新的数组，然后将调用它的对象(this 指向的对象)中的元素以及所有参数中的数组类型的参数中的元素以及非数组类型的参数本身按照顺序放入这个新数组,并返回该数组.

concat 方法并不修改调用它的对象(this 指向的对象) 和参数中的各个数组本身的值,而是将他们的每个元素拷贝一份放在组合成的新数组中.原数组中的元素有两种被拷贝的方式:

- 对象引用(非对象直接量):concat 方法会复制对象引用放到组合的新数组里,原数组和新数组中的对象引用都指向同一个实际的对象,所以,当实际的对象被修改时,两个数组也同时会被修改.
- 字符串和数字(是原始值,而不是包装原始值的 String 和 Number 对象):concat方法会复制字符串和数字的值放到新数组里

``` js
var myCar = [{color: "red"},5, {wheels: 4}];
var newCar = myCar.concat();
newCar[0].color="purple";
newCar[1]=1;
newCar.push("engine");
console.log(myCar);//[{color: "purple"},5,{wheels: 4}]
console.log(newCar);//[{color: "purple"},1,{wheels: 4},"engine"]

```

结果不言而喻。

所谓抛砖引玉，砖已经抛出去了，下面来实现对象的深度克隆。为什么不是数组呢？因为数组也是对象，数组的元素也可能是对象，在深度克隆的过程中，当对象的值也是一个对象时，对对象的值也要进行深度克隆。
下面是一个深度克隆函数：

``` js
function clone(obj) {
    var newobj;
    if (obj === null || obj == undefined || typeof(obj) != "object") {
        return obj;
    }
    if (obj instanceof Array) {
        newobj = [];
        for (var i = 0; i < obj.length; i++) {
            if (typeof obj[i] == "object" && obj[i] != null) {
                newobj[i] = arguments.callee(obj[i]);
            } else {
                newobj[i] = obj[i];
            }

        }
    }else{
        newobj={};
        for(var key in obj){
            if(typeof obj[key]=="object" && obj[key]!=null){
                obj[key]=arguments.calle(obj[key]);
            }else{
                newobj[key]=obj[key];
            }
        }
    }
    return newobj;
    
}

var myCar = [{color: "red"},5, {wheels: 4}];
var newCar=clone(myCar);
newCar[0].color="purple";
newCar[1]=1;
newCar.push("engine");
console.log(myCar);//[{color: "red"},5,{wheels: 4}]
console.log(newCar);//[{color: "purple"},1,{wheels: 4},"engine"]

```
从结果我们看到，这个深度克隆函数是成功的，它不仅可以克隆原始类型，也可以克隆引用类型。说明如下：
1. 判断对象是否为null、undefined或者非object之外的数据类型，如果为真，则直接返回该对象。
2. 判断对象是否为数组，如果为真，则遍历数组，对其成员进行递归调用克隆函数。
3. 如果为非数组对象，遍历对象成员并递归调用成员对象，最后返回克隆对象。



### 拥抱未来的深度克隆方法
在MDN上面看到一个对象复制函数，用到了*Object.create()、Object.isPrototypeOf()*等比较新的方法，基本只能在 IE9+ 使用，称之为拥抱未来的深度克隆方法。

``` js
//对象复制函数
function copy(obj){
    var objproto = Object.getPrototypeOf(obj); //返回该对象的原型。
    
    var copy = Object.create(objproto); //创建一个拥有指定原型和若干个指定属性的对象。
    
    //返回一个由指定对象的所有自身属性的属性名（包括不可枚举属性）组成的数组。
    var propNames = Object.getOwnPropertyNames(obj);

    
    propNames.forEach(function(name) {
    	//返回指定对象上一个自有属性对应的属性描述符
        var desc = Object.getOwnPropertyDescriptor(obj, name);

        //直接在一个对象上定义一个新属性，或者修改一个已经存在的属性， 并返回这个对象。
        Object.defineProperty(copy, name, desc); 
        
    });
    return copy;
}
```
### 参考资料


> [MDN-Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)


