---
title: jquery 技术内幕
date: 2019-03-04
tags:
    - js
    - jquery
---

<script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.js"></script>

**记录前端鼻祖 ———— jquery 的 js 技巧**

### jquery 对象

**在没有 es6 的 Class 之前，js 中都使用 function 以及他的 propotype 来实现面向对象。但是尽管如此，总有人在初始化实例时，忘记加上 new 关键字，这就会造成意外的全局污染。**

``` js
function Person(name) {
    this.name = name;
}

// 正常使用
var man = new Person('老王');
console.log(man.name); // 打印 老王

// 忘记带 new
var man = Person('老王');
console.log(window.name); // window 添加了一个变量 name
```

**这就会在你不知情的情况下，定义了一个全局变量。而 jquery 在生成新对象的时候，从来都没有带上 new 关键字，jquery 在创建对象时使用了一个技巧。**

``` js
function Person(name) {
    this.name = name;
}

function createPerson(name) {
    return new Person(name);
}

var man = new createPerson('老王');
var woman = createPerson('小丽');

console.log(man.name); // 打印 老王
console.log(woman.name); // 打印 小丽
console.log(man.prototype === woman.prototype); // 打印 true
```

**使用 new 关键字后，如果函数 return 了一个对象，返回对象，若返回的不是对象则创建一个新对象返回。**

**在 createPerson 函数返回了一个 new Preson，使得即使不使用 new 关键字也可以创建对象实例，既保证创建对象时的安全，也方便调用。**

**源码中的代码：**

![jquery_init](../../../../img/jquery_technology_insider/jquery_init.png)

### jquery 原型

**在使用 jquery 的插件时，在 $.fn 上扩展方法，对象 $() 上就能访问刚刚挂载的方法。**

``` js
$.fn.log123 = function () {
    console.log(123)
}

$().log123() // 打印 123
```

**再结合上面返回的 $.fn.init 实例，可见 $.fn 就是 $.fn.init.prototype。的确在源码中也做了这样的定义，使得返回的对象可以拿到原型上的方法。**

![jquery_init_propotype](../../../../img/jquery_technology_insider/jquery_init_propotype.png)

**再附上一张 jquery 非常绕的原型图：**

![jquery_propotype](../../../../img/jquery_technology_insider/jquery_propotype.png)

### 
