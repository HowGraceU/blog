---
title: js和后台语言的差异
date: 2019-03-08
tags:
    - js
---

## 一、强类型

**强类型优势：**
**1.  数据类型固定不变，减少了程序由于数据类型问题报错的概率，保证程序健壮性。**
**2.  所有变量类型确定，增加了一定可读性。**

## 二、变量

### var 声明变量

**使用 var 关键字声明变量，会导致变量的提升。在 let 出现之后，这个问题很好的解决了。**

``` js
console.log(a);
var a = 1;
// 打印 undefined

console.log(a);
let a = 1;
// 报错 b is not defined
```

### function 声明

**使用 function 关键字声明函数时，会导致函数提升，提升的方式与浏览器的版本密切相关。**

``` js
function test() {
    console.log('out');
}

(function () {
    if (false) {
        function test() {
            console.log('in');
        }
    }

    test()
})()

// 老版本浏览器 打印 in
// 新版本浏览器 报错 test is not a function
```

**在ES6的浏览器中，它们的行为实际上是这样的：**

**1.   允许块级作用域中定义函数。**

**2.   函数声明实际上将会类似于使用 var 声明的函数表达式，函数名将会提升至当前函数作用域顶。**

**3.   同时函数声明也会保持在块级作用域中的提升行为。**

**上述代码在现代浏览器中类似：**

``` js
var test;
test = function () {
    console.log('out');
}

(function () {
    var test;
    if (false) {
        test = function () {
            console.log('in');
        }
    }

    test();
})()
```

## 三、this

**一般后台语言中的 this 就指向当前class类，但js 中 this 是多变的，这已经是老生常谈了。js 中的 this 分了三种情况。**

**1.   普通函数调用，this 指向window。**

**2.   对象调用，this 指向调用方法的对象。**

**3.   构造函数，this 指向返回的实例。**

## 四、函数参数

**js 中的函数在运行时，传入的参数与定义时的参数数量可以不同，在函数内可以通过 arguments 对象取得未定义的参数，这使 js 有时候变得方便，但是缺失了函数的重载（可以通过一些[技巧](../../06/js-skill/)实现函数重载）**

## 五、数组

**js 中的数组不是真的数组，他属于对象。当数组下标都是连续时，数组为快数组，而下标不连续时，数组就变成了 map。当操作只有出栈入栈时，js 数组的性能会不如 List。**

**[从Chrome源码看JS Array的实现](https://zhuanlan.zhihu.com/p/26388217)**

## 六、面向对象

**在 es6 到来之前，网上流传了多个版本的使用 es3 实现类和继承，其中最为出名的就是基于原型的继承方式。**

**在 es6 中引入 Class 关键字，这简直太棒了，避免了各个公司或插件实现的继承不同而造成阅读代码困难的问题，并且 Class 使用起来相对方便。**

**但是 Class 暂时还没有私有变量的概念，不像其他的后端语言。**