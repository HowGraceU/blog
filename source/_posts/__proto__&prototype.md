---
title: __proto__ 和 prototype
date: 2019-04-05
tags:
    - js
    - prototype
---

**js 中的面向对象有非常多的版本，知道 es6 之后才统一规范，而且实例中有一些属性能帮我们找到实例的构造函数。**

## __proto__

**__proto__ 是一个对象实例的属性，而对象实例是通过 new 关键字返回的，我们先来看看使用 js 来模拟一个 new 关键字。**

``` js
function myNew(fun) {
    var obj = {};
    if(fun.prototype !== null){
    // 将实例的proto指向构造函数的原型
        obj.__proto__ = fun.prototype;
    }

    // 如果构造函数有返回结果，用ret接收,通过apply,将构造函数的this指向实例res;
    var ret = fun.call(obj, Array.prototype.slice.call(arguments,1));

    if((typeof ret === 'object' || typeof ret === 'function' ) && ret != null){
        return ret;
    } else {
        return obj;
    }
}
```

**由此可见，对象实例的 __proto__ 指向创建这个实例的类的原型 prototype。而在原型上，都会有一个 constructor 指向构造函数。如下图**

![es3 proto](../../../../img/__proto__&prototype/es3_proto.png)

**使用 es6 的 class 语法糖得到结果和上述一样。**

![es6 proto](../../../../img/__proto__&prototype/es6_proto.png)