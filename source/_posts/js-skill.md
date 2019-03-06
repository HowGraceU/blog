---
title: js 小技巧
date: 2019-03-6
tags:
    - js
---

### js 重载

**众所周知，js 中没有重载概念，但可以利用代码来实现重载。**

``` js
function addMethod(obj, name, fn) {
    let oldFn = obj[name];

    obj[name] = function (...arg) {
        arg.length === fn.length? fn(...arg): oldFn(...arg);
    }
}

let jqx = {};
addMethod(jqx, 'say', () => {console.log(0)});
addMethod(jqx, 'say', (a) => {console.log(1)});
addMethod(jqx, 'say', (a, b) => {console.log(2)});

jqx.say();
jqx.say('a');
jqx.say('a', 'b');
```

**如果怕调用栈太深，也可以借助 map 对象把 arg.length 和 函数对应起来。**

### 优化 switch

**有些情况下，需同时判断多个条件，写 if-else 和 switch 会使函数看起来相当庞大，不妨试试 map。**

``` js
let classical = new Map();
classical.set(20, '弱冠');
classical.set(30, '而立');
classical.set(40, '不惑');
classical.set(50, '知天命');
classical.set(60, '花甲子');
classical.set(70, '古来稀');
classical.set(80, '耄耋之年');

function age2classical(age) {
    return classical.get(age);
}

age2range(20); // 弱冠
age2range(25); // undefined
age2range(40); // 不惑
```