---
title: js 中的“魔鬼”
date: 2019-04-05
tags:
    - js
    - leak
---

## eval 导致闭包引用不释放

### 利用 weakSet 查看内存引用。

**首先来讲一下如何看内存泄漏。第一我们可以利用 devTool 中的 Memory 来查看内存，其次 es6 中出现了 WeakSet 数据结构，它的好处就是 GC 收回内存时，不会计算 WeakSet 中的引用，可以利用他来看一些内存是否已被回收。**

``` js
let obj = {a: {}};

let weakset = new WeakSet([obj.a]);
console.log(weakset.has(obj.a)); // true，此时若打印 weakset 可以看到里面包含了一个对象

// console.log(obj.a)
// 若此处打印了 obj.a 对象，则会在控制台中保存 obj.a 的引用，会导致下面触发 GC 的时候不回收对象
// 当然也可以清空控制台之后再触发 GC

delete obj.a;
// 因为 GC 不是随时触发的，此时一定要点一下 devTool 中的 Memory 里的
console.log(weakset); // 打印出一个空的 weakset
```

### 内存泄漏

**然后我们利用 WeakMap 来查看一个 eval 对闭包的影响。**

``` js
class Jqx {};
let weakset =  new WeakSet();
// 正常的闭包
function test() {
    let jqx = new Jqx();
    let jqx2 = new Jqx();

    weakset.add(jqx);
    weakset.add(jqx2);

    return function () {
        return jqx;
    }
}

let getJqx = test();
// 手动触发 GC
console.log(weakset); // WeakSet {Jqx}，只有一个对象
```

**上面闭包没有对 jqx2 的引用，所以 GC 会回收，通过内存快照得到下面结果。**

![no_eval](../../../../img/es3_leak/no_eval.png)

``` js
class Jqx {};
let weakset =  new WeakSet();
// 带 eval 的闭包
function test() {
    let jqx = new Jqx();
    let jqx2 = new Jqx();

    weakset.add(jqx);
    weakset.add(jqx2);

    return function () {
        eval('');
    }
}

let getJqx = test();
// 手动触发 GC
console.log(weakset); // WeakSet {Jqx, Jqx}，两个对象都在
```

**上面闭包中有 eval，因为 eval 是在运行时才会将其中的字符串转为 js，所以 js 引擎保留了整个闭包中的所有的内容。通过内存快照得到下面结果。**

![eval](../../../../img/es3_leak/eval.png)

**此外，eval也容易引起网站被攻击，不到万不得已就不要使用了。**

## new Function

**讲完 eval，再来说说 new Function。Function 的调用与 eval 相似，不过 Function 中若有变量，则都指向全局变量，而不会对闭包中的变量进行引用，所以也不会像 eval 一样造成内存泄漏。**

``` js
var name = 'jqx';
var test = function () {
    var name = 'jqx2';
    return new Function('console.log(name)');
}

test()(); // 打印 jqx
```

## with

**说到 eval 肯定也要提一嘴 with，俩兄弟双双欺骗词法作用域（在运行时改变作用域）。**

``` js
var obj = {
    name: 'jqx'
}

with (obj) {
    name = 'jqx2';
    job = 'programmer'; // 若对象 obj 上没有 job 属性，则赋值到 window 上
}

console.log(obj.name);
console.log(obj.job);
console.log(window.job);
```

## 严格模式下的 eval 和 with

**严格模式下 with 就直接被禁用了，而 eval 无法修改作用域。不过 eval 还是可以引用到闭包中的变量，所以闭包还是不会被释放。**

``` js
class Jqx {};
let weakset =  new WeakSet();
// 带 eval 的闭包
function test() {
    
    let jqx = new Jqx();
    let jqx2 = new Jqx();

    weakset.add(jqx);
    weakset.add(jqx2);

    return function () {
		'use strict';
        eval('console.log(jqx);var a = 1;');
		console.log(a);
    }
}

let getJqx = test();
// 手动触发 GC
console.log(weakset); // WeakSet {Jqx, Jqx}，两个对象都在

getJqx(); 
// 打印对象 jqx
// 报错 a 不存在
```