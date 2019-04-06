---
title: js 小细节
date: 2019-03-12
tags:
    - js
---

**此贴用来记录一些 js 的小细节（常见坑）**

### 一、js 不能没有分号！！

**先标题档一下，js 可以没有分号，但是某些时候又不得不加分号。在使用立即执行函数，容易在开头漏掉分号，以至于 js 把上一行的返回当成了一个函数来运行。**

``` js
(() => {alert(1)})() // alert 1
(() => {alert(2)})() // 报错
```

### 二、同时声明同名函数与定义变量

**在同时声明函数和定义变量时，如果没有给变量赋值，则函数生效。**

``` js
function foo() {}; // 定义函数在先
var foo;

console.log(foo);
// 打印 函数foo

var foo; // 定义变量在先
function foo() {};

console.log(foo);
// 打印 函数foo
```

### 三、给变量赋值函数，且给函数名字。

**给变量赋值函数，且给函数一个名字，在函数内部打印这个名字即指向该函数。函数内部无法给这个变量赋值，且外部无法获取这个变量。**

``` js
let a = function foo() {
    console.log(foo); // 打印 foo 函数
    foo = 1;
    console.log(foo); // 打印 foo 函数

    // 若重新定义了 foo 变量，贼内部的 foo 不再指向函数。
    // var foo = 1;
    // console.log(foo); // 打印1
}

console.log(foo); // 报错 foo is not defined
```

---

**2019-04-03 补充**

**没有定义给属性的函数可以在函数内部重新定义函数名。**

```js
function test() {
	console.log(test);
	test = 1;
	console.log(test);
}

test(); // 打印 函数 和 1
test(); // 报错


let test = function test() {
	console.log(test);
	test = 1;
	console.log(test);
}

test();  
test(); // 无法修改 test 值，打印 4 次函数
```

### 四、不能 new es6 的函数

**所谓 es6 的函数是指对象的简写函数和箭头函数。两者在 new 的时候都会报错，**

``` js
let a = {
    foo() {}
}

new a.foo(); // a.foo is not a constructor

let foo = () => {};

new foo(); // foo is not a constructor
```

### 五、new 的时候，函数内的 this 指向创建的实例

**在调用 new 的时候，函数内的 this 指向创建的实例，且不能通过 bind 来改变。**

``` js
var obj = {
    a: 10,
    foo: function () {
        console.log(this.a);
    }
}

var obj2 = {
    a: 20
}

obj.foo() // 打印 10
var myFoo = obj.foo.bind(obj2);
myFoo(); // 打印 20
new myFoo(); // 打印 undefined
```

### 六、原型链属性失效

**当实例中定义过属性，即使是 undefined，也不会取原型链上的属性。**

``` js
function foo(a) {
    this.a = a;
}
foo.prototype.a = 10;

console.log((new foo()).a); // 打印 undefined
```

### 七、function 的变量提升

**在老版本的浏览器中，声明函数会提升到当前函数作用域的顶端，在最后声明的函数在作用域内都可以使用。**

``` js
foo(); // 打印 1

if (false) {
    function foo() {
        console.log(1);
    }
}
```

**在支持 es5 的现代浏览器中，却是不同的结果。声明函数时，会先将定义变量（var foo）提升到<span style='color:Crimson;'>函数作用域</span>的顶端，然后在<span style='color:Crimson;'>块级作用域</span>顶端对函数赋值。**

``` js
console.log(foo); // foo 已经定义，打印 undefined
{
    console.log(foo); // foo 函数提升，打印 foo 函数
    function foo() {};
}


foo2(); // 报错 foo2 is not a function

if (true) { // 不管是 true 还是 false，运行 foo2 时 foo2 还未赋值为函数
    function foo2() {
        console.log(1);
    }
}
```

### 八、严格模式

**es5 中规定了严格模式后，给 js 增加了很多合理的规范，但是严格模式对函数中的函数不起作用。**

``` js
function useStrict() {
    'use strict';
    console.log(this);
}
useStrict(); // 打印 undefined

function noStrict() {
    console.log(this);
}
noStrict(); // 打印 window

function useStrict2() {
    'use strict';
    noStrict();
}
useStrict2(); // 打印 window
```