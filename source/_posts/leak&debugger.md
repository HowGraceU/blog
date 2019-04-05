---
date: 2018-08-23
title: 常见的内存泄露和排查
description: 常见的内存泄露和排查
tags:
    - js
    - devtools
author: jinqixiao
---
# 内存泄露

## 什么是内存泄露

本质上来讲，内存泄露是当一块内存不再被应用程序使用的时候，由于某种原因没有返还给操作系统或者空闲内存池的现象。

## js的内存管理

js中的垃圾回收机制，会周期性地检查之前被分配出去的内存是否可以从应用的其他部分访问。换句话说，什么样的内存仍然可以从应用程序的其他部分访问，不可访问的内存可以通过算法确定并标记以便返还给操作系统。

# 常见4种的内存泄露

1.  意外的全局变量

``` js
/* 例子1 */ 
function foo () {
    bar = 'a hidden global variable';
}

/* 例子2 */ 
function foo () {
    this.bar = 'a global variable';
}

foo()
```

##### 全局变量注意事项

如果你必须使用全局变量来存储很多的数据，请确保在使用过后将它设置为null。

2.  监听

``` js
var el = ducumemt.getElementById('a');

var onClick = () => {
    el.innerText = 'text';
}

el.addEventListener('click', onClick);

el.removeEventListener('click', onClick);
el.parentNode.removeChild(el);
```

dom节点和js代码之间的循环引用，在ie老版本上会有内存泄露bug。大部分类库在废弃一个节点之前，会移除listener。目前，现代浏览器（包括IE）都能发现这些循环引用并正确回收，在废弃一个节点之前调用removeEventListener不再是必要的操作。

3.  DOM引用

``` js
<div id='a'>
    <div id='b'></div>
<div>

var divB = document.getElementById('b');
document.body.removeChild(document.getElementById('a'));
```

js代码中保留了子元素的特定引用，然后移除父辈元素div#a，表面上看整个元素都被移除了，但是在内存中还保留着对div#a的引用。因为div#b保留着对父辈的引用，导致整个div#a都保留在内存中。

4.  闭包

``` js
/* 代码一 */ 
var obj = null;
var replaceObj = () => {
    var oldObj = obj;
    obj = {
        memory: new Array(1000000).join('*'),
        closeFn () {
            console.log(123);
        }
    };
    oldObj && oldObj.closeFn();
}

setInterval(replaceObj, 1000);
```
上述代码调用结果如下图

![无闭包时函数调用](../../../../img/leak&debugger/1.jpg)

断点进函数查看，并没有闭包引用，并且打印稳定1秒钟1个

正常时候的内存情况为下图

![正常内存](../../../../img/leak&debugger/2.jpg)

---
分割线
___

``` js
/* 代码二 */
var obj = null;
var replaceObj = () => {
    var oldObj = obj;
    obj = {
        memory: new Array(1000000).join('*'),
        closeFn () {
            /* 将函数调用移至闭包内 */
            if (oldObj) {
                console.log('hi')
            }
            console.log(123);
        }
    };
    oldObj && oldObj.closeFn();
    console.log('---------------------');
}

setInterval(replaceObj, 1000)
```

上述代码调用结果如下图

![有闭包时控函数调用](../../../../img/leak&debugger/4.jpg)

断点进函数查看，函数内部存在闭包引用，每次创建的新对象都会引用上一个对象，导致每次创建的对象都不被GC回收。因为闭包引用而导致对象没有被回收，熟练使用谷歌调试也可以查出来,重点看代码三。

---
分割线
___

``` js
/* 代码三 */
var obj = null;
var replaceObj = () => {
    var oldObj = obj;

    var unused = () => {
        if (oldObj) {
            console.log('hi')
        }
    }

    obj = {
        memory: new Array(1000000).join('*'),
        closeFn () {
            console.log(123);
        }
    };
}

setInterval(replaceObj, 1000)
```

上述代码因为不能在闭包里面打印，所有用谷歌调试工具抓内存查看

文中代码在浏览器中运行时的内存情况

![文中代码内存情况](../../../../img/leak&debugger/5.jpg)

导致内存泄露的原因是在同一个父作用域下创建闭包时，这个作用域是共享的。代码中closeFn的闭包作用域和unused的闭包作用域是共享的。即便unused函数从未被使用且不可能被使用，它对oldObj的引用造成了oldObj的活跃（阻止它被回收）。代码三中closeFn拥有的闭包与代码二中closeFn拥有的闭包相同。

---
**2019-04-04 记录，v8引擎修复了没有调用的函数对闭包的引用，但是在 closeFn 里面加上 debugger，可以看到还是保存了对闭包的引用。**
___

#   devTools调试

##  Profiles视图

在老版本谷歌上显示为Profiles，新版本谷歌改名为Menory，文中使用谷歌版本69。该功能可以对js内存进行快照，也可以记录一段时间内的内存分配情况，如下图。

![快照、录像](../../../../img/leak&debugger/6.jpg)

以上文代码三内存泄露为例，

1.  快照

快照中summary视图提供了不同类型的分配对象以及他们的合计大小。shallow size表示一个特定类型所有对象总和，retained size表示此对象的shallow size和保留此对象的其他对象的shallow size总和，distance表示对象到GC根（最初引用的那块内存）的最短距离。

![快照](../../../../img/leak&debugger/7.jpg)

图中的字符串的shallow size等于retained size，大概因为字符串不是引用类型。
字符串上面的closeFn函数的shallow size只有56，二retained size的大小是3000640，是因为这个对象被其他对象引用，其他对象的总大小加上closeFn函数的shallow size得到3000640。
图中可以看出，window的distance是1，即最接近GC根，往下的Object直接定义在window上所有distance是2，后面一层一层引用，引用得越深，distance越大。

2.  对比快照

快照中comparison视图提供了这一次快照与上一次快照直接的对比

![快照对比](../../../../img/leak&debugger/8.jpg)

这个视图里能清晰看出这一次快照与上一次快照新增，删除了多少对象。

3.  录像

devTools还提供了录像功能，点击Profiles，选择第二个切换到录像。

![切换录像](../../../../img/leak&debugger/9.jpg)

开始录像后能清晰看到网页根据时间轴所产生的内存，可以选择某一时间段去查看其中的内存分配

![录像](../../../../img/leak&debugger/10.jpg)

## 参考文献
javaScript中4种常见的内存泄露陷阱(http://web.jobbole/com/88463)
本文全是作者对这篇文献的理解，或与原文有所不同，若有错误欢迎指出。