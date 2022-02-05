---
title: 理解ES6之箭头函数
date: 2017-02-02 19:49:49
categories: 前端
banner: http://img.yanyuanfe.cn/es6.png
tags:
	- ES6
---



> 当你看到箭头函数（Arrow Function）和（=>）的时候，我想你也能很容易将他们联系到一块儿。

![image](http://img.yanyuanfe.cn/es6.png)

<!--more-->

<div class="tip">
    实习的时候，项目采用了React架构，代码中应用了ES6/7的规范，并使用了babel转译，也加入了相应的Polyfill库。可以预见ES6的发展已经越来越深入人心，在此处总结了ES6中箭头函数的用法。
</div>

### 箭袋中的新羽

ES6中引入了一种编写函数的新语法。  
  
  箭头函数就是个简写形式的函数表达式，并且它拥有词法作用域的this值（即不会新产生自己作用域下的this, arguments, super 和 new.target 等对象）。此外，箭头函数总是匿名的。  
  箭头函数的定义由一个参数列表（零个或多个参数，如果参数不是只有一个，需要有一个( .. )包围这些参数）组成，紧跟着是一个=>符号，然后是一个函数体。  
  当你的函数中只有一个参数时，箭头函数的语法非常简单：标识符=>表达式。你无需输入function和return，一些小括号、大括号以及分号也可以省略。 
``` js
// ES5
function foo(x) {
    return ++x;
}

// ES6
var foo = x => ++x;
```
当你的函数中含有多重参数（也可能没有参数，或者是不定参数、默认参数、参数解构）时，你需要用小括号包裹参数list。

``` js
// ES5
function foo(x,y) {
    return x + y;
}

function bar() {
    return 100
}

// ES6
var foo = (x,y) => x+y;
var bar = () => 100;
```
除表达式外，箭头函数还可以包含一个块语句。

``` js
// ES5
function handleClick(e) {
    e.preventDefault();
    console.log('The link was clicked.');
}

// ES6
function handleClick = e => {
    e.preventDefault();
    console.log('The link was clicked.');
}
```
<div class="tip">
    注意:使用了块语句的箭头函数不会自动返回值，你需要使用return语句将所需值返回。
</div>

<div class="tip">
提示：由于花括号被解释为代码块，所以如果箭头函数直接返回一个对象，必须在对象外面加上小括号。
</div>


```js
let name_maker = (first,last) => {firstname: first,lastname:last};  // 这样写会报SyntaxError！

let name_maker = (first,last) => ({firstname: first,lastname:last});
```

用小括号包裹空对象就可以了。

不幸的是，一个空对象{}和一个空的块{}看起来完全一样。ES6中的规则是，紧随箭头的{被解析为块的开始，而不是对象的开始。因此，=>{}这段代码就被解析为没有任何行为并返回undefined的箭头函数。

更令人困惑的是，你的JavaScript引擎会将类似{key: value}的对象字面量解析为一个包含标记语句的块。幸运的是，{是唯一一个有歧义的字符，所以用小括号包裹对象字面量是唯一一个你需要牢记的小窍门。

### 箭头函数的this？

在箭头函数出现之前，每个新定义的函数都有其自己的  this 值（例如，构造函数的 this 指向了一个新的对象；严格模式下的函数的 this 值为 undefined；如果函数是作为对象的方法被调用的，则其 this 指向了那个调用它的对象）。在面向对象风格的编程中，这被证明是非常恼人的事情。
在ECMAScript 3/5中的普通function函数中，为了得到期望的this值，你可能会新增一个变量来指向期望的 this 对象，然后将该变量放到闭包中来解决。除此之外，你也可以使用 bind 函数，把期望的 this 值传递给当前 函数。  
箭头函数的出现解决了这个问题，箭头函数没有它自己的this值，它会捕获其所在上下文的this 值，作为自己的 this 值。
来看看[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
的例子。
```js
function Person() {
  // 构造函数 Person() 定义的 `this` 就是新实例对象自己
  this.age = 0;
  setInterval(function growUp() {
    // 在非严格模式下，growUp() 函数定义了其内部的 `this`
    // 为全局对象, 不同于构造函数Person()的定义的 `this`
    this.age++; 
  }, 1000);
}

var p = new Person();

```

```js
//ES5
function Person() {
  var self = this; // 也有人选择使用 `that` 而非 `self`. 
                   // 只要保证一致就好.
  self.age = 0;

  setInterval(function growUp() {
    // 回调里面的 `self` 变量就指向了期望的那个对象了
    self.age++;
  }, 1000);
}
```
在这里，你希望在内层函数里写的是this.agg++，不幸的是，内层函数并未从外层函数继承this的值。在内层函数里，this会是window或undefined，临时变量self用来将外部的this值导入内部函数。（另一种方式是在内部函数上执行.bind(this)，两种方法都不甚美观。）  
在ES6中，不需要再hackthis了。
```js
//ES6
function Person(){
  this.age = 0;

  setInterval(() => {
    this.age++; // |this| 正确地指向了 person 对象
  }, 1000);
}

var p = new Person();
```
```js
var adder = {
  base : 1,
    
  add : function(a) {
    var f = v => v + this.base;
    return f(a);
  },

  addThruCall: function(a) {
    var f = v => v + this.base;
    var b = {
      base : 2
    };
            
    return f.call(b, a);
  }
};

console.log(adder.add(1));         // 输出 2
console.log(adder.addThruCall(1)); // 仍然输出 2（而不是3）
```

<div class="tip">
由于 this 已经在词法层面完成了绑定，所以当然也就不能用call()、apply()、bind()这些方法去改变this的指向。
</div>


当我们使用箭头函数时，函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。并不是因为箭头函数内部有绑定this的机制，实际原因是箭头函数根本没有自己的this，它的this是继承外面的，因此内部的this就是外层代码块的this。  

箭头函数与非箭头函数间还有一个细微的区别，箭头函数不会获取它们自己的arguments对象。诚然，在ES6中，你可能更多地会使用不定参数和默认参数值这些新特性。

### 箭头函数绑定

箭头函数可以绑定this对象，大大减少了显式绑定this对象的写法（call、apply、bind）。但是，箭头函数并不适用于所有场合，所以ES7提出了“函数绑定”（function bind）运算符，用来取代call、apply、bind调用。虽然该语法还是ES7的一个提案，但是Babel转码器已经支持。

函数绑定运算符是并排的两个双冒号（::），双冒号左边是一个对象，右边是一个函数。该运算符会自动将左边的对象，作为上下文环境（即this对象），绑定到右边的函数上面。  

``` js
// ES5
bar.bind(foo);
// ES7
foo::bar;

// ES5
bar.apply(foo, arguments);
// ES7
foo::bar(...arguments);


```

如果双冒号左边为空，右边是一个对象的方法，则等于将该方法绑定在该对象上面。

```js
// ES5
var method = ::obj.foo;
// ES7
var method = obj::obj.foo;
```

### 总结

箭头函数有几个使用注意点。

- 函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。

- 不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。

- 不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用Rest参数代替。

- 不可以使用yield命令，因此箭头函数不能用作Generator函数。

- 除了this，以下三个变量在箭头函数之中也是不存在的，指向外层函数的对应变量：arguments、super、new.target。  

### 参考资料

- [ES6标准入门](http://es6.ruanyifeng.com/#docs/function)

- [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

