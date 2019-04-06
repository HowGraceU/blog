---
title: node 循环引用
date: 2018-09-14 10:54:11
tags:
    - node
    - moudle
---

**近期在node开发的过程中，碰到require文件后，无法调用文件内定义的方法，翻阅资料怀疑是循环引用，于是自己进行了测试。**

## 一、循环引用测试

**a.js**
``` js
console.log('at a.js');
var b = require('./b');

exports.test = "i'm a";

setImmediate(() => {
    console.log(`get b.test ${b.test}`);
})
```

**b.js**
``` js
console.log('at b.js');
var a = require('./a');

module.exports = {
    test: "i'm b"
}

setImmediate(() => {
    console.log(`get a.test ${a.test}`);
})
```

**测试结果如下**
``` js
node .\a.js

at a.js
at b.js
get a.test i'm a
get b.test i'm b

node .\b.js

at b.js
at a.js
get b.test undefined
get a.test i'm a
```

**运行b.js的时候，当运行到 require('./a') 的时候，去加载a.js的代码，而a.js里又 require('./b')，这里导致了循环引用。**

**但是这里并不会进入死循环，原因是node在require的时候发现内存里已经存在这个模块的引用，则返回引用而不会重新去运行这个模块的代码。**

**node .\a.js的时候，require('./b')，而a.js停留在 var b = require('./b');，此时在b.js 得到的a模块的内容为a的module.exports {}。在运行完b.js之后，又在a的module.exports对象上加上属性，所以b中延迟打印出来的a的属性存在。**

**而node .\b.js的时候，require('./a')，b.js停留在var a = require('./a');，此时还没有运行module.exports = { test: "i'm b" }，得到b模块的内容为b的module.exports {}，运行完a之后，执行 module.exports = ... 代码，将module.exports指向了另一个对象，而缓存中b模块的内容则是之前引用的b的module.exports，所以b的test属性并没有加在b模块在内存中的对象上，所以之后再有require('./b')拿到的都是空对象 {}**

## 二、避免循环引用产生的问题

*   尽量不使用module.exports重置模块的引用。
*   非要使用module.exports，module.exports应该放在js的开头，即其他逻辑之前。

### 题外：在查node循环引用问题时，看到一个知名搜索引擎上较为靠前的博客有点问题。

**以下为文中的测试代码**
``` js
a.js

console.log('a.js');
var b = require('./b');
console.log('a+.js');
console.log(b);
exports={name:'a'};
console.log('a++.js');


b.js

console.log('b.js');
var c = require('./a');
console.log('b+.js');
console.log(c);
exports={name:'b'};
console.log('b++.js');
```

**测试结果中，模块a和模块b都是 {}**
**这个代码不管怎么改，打印出来的结果都是 {}，因为在node中 exports={name:'a'}; 无法用来暴露变量。原因如下**

``` js
(function (exports, require, module, __filenam, __dirname) {
    ...文件中代码
})
```
**代码来自《深入浅出nodejs》，上述代码的是在每一次用node运行js脚本的时候，都会运行上述函数来执行脚本中的代码，而脚本中使用的exports、module都只是这个函数中的一个局部变量。当你运行 exports={name:'a'}; 时，不过改变这个函数作用域中，局部变量的指向，而非改变 module.exports 所指向的对象中的属性。**