---
title: JavaScript排序算法之选择排序和插入排序
date: 2018-12-2 20:01:37
banner: http://img.yanyuanfe.cn/algorithm.jpg
tags:
 - 算法
---

> 学习数据结构和算法有助于写出性能更优的代码。

![image](http://img.yanyuanfe.cn/algorithm.jpg)

<!--more-->

> 基础决定你可能达到的高度，而业务决定了你的最低瓶颈

了解和学习JavaScript常用算法可以帮助前端开发者写出性能更好的代码，本文介绍排序算法中的两种时间复杂度为O(n^2)的算法，选择排序和插入排序。

### 选择排序

选择排序的原理是：对排序数组进行遍历，在遍历的过程中，对当前索引之后的数组进行遍历，找到最小元素的索引，将当前索引和最小元素的索引对应元素进行交换。

``` js
function swap(arr, a, b) {
  [arr[a], arr[b]] = [arr[b], arr[a]];
}

function selectionSort(arr) {
  for (let i = 0, len = arr.length; i < len; i ++) {
    let minIndex = i;
    for(let j = i + 1; j < len; j++) {
      if(arr[j] < arr[minIndex]) {
        minIndex = j;
      }
    }
    swap(arr, i, minIndex);
  }
}

```
选择排序因为嵌套了两层循环，所以其时间复杂度为O(n^2)



### 插入排序

插入排序的原理与打扑克牌时按牌的大小整理顺序类似，从数组的第一项开始遍历，遍历过程中对于当前索引的左边数组从右向左进行大小比较，如果左边项大于右边项，则进行交换。否则退出当前循环。代码实现如下：

``` js
function insertionSort(arr) {
  // 从1开始
  for (let i = 1, len = arr.length; i < len; i ++) {
    // 寻找元素合适的插入位置
    for (let j = i; j > 0; j --) {
      if (arr[j] < arr[j - 1]) {
        swap(arr, j, j - 1);
      } else {
        break; // 退出当前循环
      }
    }
  }
}
```
在内层循环中，可以将比较两项大小的操作放到for循环中，以此来简化代码。

``` js
function insertionSort(arr) {
  // 从1开始
  for (let i = 1, len = arr.length; i < len; i ++) {
    // 寻找元素合适的插入位置
    for (let j = i; j > 0 && arr[j] < arr[j - 1]; j --) {
      swap(arr, j, j - 1);
    }
  }
}
```
对比选择排序和插入排序，两者都是进行了两层循环，时间复杂度为O(n^2)，但是从代码中看，插入排序在内层循环中会提前终止循环，是否表示插入排序的速度要优于选择排序呢？

### 测试排序时间

现在我们对两种排序算法进行测试，验证是否插入排序的速度优于选择排序？

此处使用console.time和console.timeEnd来测试一段代码运行消耗的时间。console.time方法是开始计算时间，console.timeEnd是停止计时，输出代码执行的时间。
使用方法如下：


``` js
// 计时开始
console.time('test');

// 测试时间的代码

// 计时结束，输出时间
console.timeEnd('test');

// test:4522.303ms
```
这两个方法接收一个参数，作为计时器的名称。

编写测试代码如下：


``` js
function testFuncTime(tag, fn, args) {
  console.time(tag);
  fn(args);
  console.timeEnd(tag);
}
```
testFuncTime接收三个参数，分别是计时器名称，测试的方法，测试方法的参数。

为了使测试结果更加明显，我们需要一个可以生成任意范围随机数的数组的方法。


``` js
function getRandom(start, end, n) {
  let arr = [];
  for(let i=0; i<n; i++) {
    let choice = end - start + 1;
    let value = Math.floor(Math.random() * choice + start);
    arr.push(value);
  }
  return arr;
}
```
编写getRandom方法用于生成在start和end之前的n个数字的随机数数组。

下面是测试代码：


``` js
let array = getRandom(1, 10000, 1000);
let array2 = [...array];

testFuncTime('selectionSort', selectionSort, array);
testFuncTime('insertionSort', insertionSort, array2);

console.log(array);
console.log(array2);
```
在上述代码中，生成了一个范围在1-10000，长度为1000的随机数数组array，并且拷贝一份赋值给array2，然后运行testFuncTime进行测试，最终输出排序结果。
结果如下：

![排序测试](http://img.yanyuanfe.cn/sortTest1.png)
可以看到插入排序的耗时大约是选择排序的三倍，恰恰和预期相反，那究竟是为什么呢？

原因就是：选择排序内层遍历一次取到最小值的索引，每次只需要一次交换（swap）操作。但是插入排序在内层遍历的时候，会从右向左依次比较大小再交换，而
交换操作又是最耗时的。

下面我们对插入排序的实现进行优化。

### 优化插入排序

``` js
function insertionSort(arr) {
  // 从1开始
  for (let i = 1, len = arr.length; i < len; i ++) {
    // 寻找元素合适的插入位置
    let e = arr[i];
    let j; // j保存元素e应该插入的位置
    for (j = i; j > 0 && arr[j - 1] > e; j --) {
      arr[j] = arr[j - 1];
    }
    arr[j] = e;
  }
}
```
在上述代码中，对插入排序进行优化，在外层循环中，首先，声明一个变量e来保存当前元素，并且声明变量j来保存元素e应该插入的位置，在内层循环中，如果左边项大于当前项，则将左边项赋值到当前项，变量j不断向左移动，直到循环结束，将待排序元素e赋值给arr[j]。

我们声明了新的变量，使用赋值操作来代替交换操作，优化了性能。下面对优化后的代码进行测试。

![排序测试](http://img.yanyuanfe.cn/sortTest2.png)

从上述图片可以看出，优化后的插入排序耗时为选择排序的一半左右，性能有了相当大的提升。


### 总结

本文使用JavaScript实现了时间复杂度为O(n^2)
的两种排序算法，对于选择排序来说，它的缺点是两层循环都必须完整得执行完成所以它的效率在任何条件下都是非常慢的。相比之下，虽然插入排序在最差的情况下复杂度为O(n^2)，但是在实际的应用中，O(n^2)复杂度的排序算法并非是一无是处的，它可以作为更加复杂排序算法的子过程进行优化，并且，对于近乎有序的数组来讲（系统日志按时间生成），性能比O(nlogn)级别的排序算法还要快，在完全有序的数组中，插入排序的复杂度可以达到O(n)。