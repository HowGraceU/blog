---
title: __proto__ 和 prototype
date: 2019-04-06
tags:
    - js
    - prototype
---

**js 中的面向对象有非常多的版本，知道 es6 之后才统一规范，而且实例中有一些属性能帮我们找到实例的构造函数。**

## 一、__proto__

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

**由此可见，对象实例的 __proto__ 指向创建这个实例的类的原型 prototype，当查找实例上没有的属性时，会去查找 __proto__ 上的属性。而在原型上，都会有一个 constructor 指向构造函数。如下图**

![es3 proto](../../../../img/proto&prototype/es3_proto.png)

**使用 es6 的 class 语法糖得到结果和上述一样。**

![es6 proto](../../../../img/proto&prototype/es6_proto.png)

### Object.getPrototypeOf

**对于这个方法官方的说法是，Object.getPrototypeOf() 方法返回指定对象的原型（内部[[ Prototype ]]属性的值）。而事实上，因为 [__proto__ 属性是被 Web 标准弃用的属性](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/proto)，推荐使用 Object.getPrototypeOf 来代替 __proto__。**

## 二、prototype

**prototype 原型看似和 __proto__ 类似，都是原型的意思，不过 [prototype 是用于类（构造函数）的，而 __proto__ 是用于实例的（instances）](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain#prototype_%E5%92%8C_Object.getPrototypeOf)。而类（构造函数）的 prototype 在 new 这个实例的时候使用。**

**我们来看一下 babel 实现的继承。**

``` js
// 源代码
class A{
	constructor(name) { this.name = name; }
  
    static prop = 1;

	sayName() {
		console.log(this.name);
	}
};

class B extends A {
    constructor(name, age) {
		super(name); this.age = age 
	}
  
    sayDetail() {
		super.sayName();
		console.log(this.age);
	}
};

// babel 转义代码
// 实现 Reflect.get
function _get(target, property, receiver) {
    if (typeof Reflect !== "undefined" && Reflect.get) {
        _get = Reflect.get;
    } else {
        _get = function _get(target, property, receiver) {
            var base = _superPropBase(target, property);
            if (!base)
                return;
            var desc = Object.getOwnPropertyDescriptor(base, property);
            if (desc.get) {
                return desc.get.call(receiver);
            }
            return desc.value;
        }
        ;
    }
    return _get(target, property, receiver || target);
}

function _superPropBase(object, property) {
    while (!Object.prototype.hasOwnProperty.call(object, property)) {
        object = _getPrototypeOf(object); // 取实例的原型
        if (object === null)
            break;
    }
    return object;
}

// 在原型上定义属性和方法，这些方法都是不可枚举的
function _defineProperties(target, props) {
    for (var i = 0; i < props.length; i++) {
        var descriptor = props[i];
        descriptor.enumerable = descriptor.enumerable || false;
        descriptor.configurable = true;
        if ("value"in descriptor)
            descriptor.writable = true;
        Object.defineProperty(target, descriptor.key, descriptor);
    }
}

// 创造类，向原型上定义方法或者在类上定义静态属性
function _createClass(Constructor, protoProps, staticProps) {
    if (protoProps)
        _defineProperties(Constructor.prototype, protoProps);
    if (staticProps)
        _defineProperties(Constructor, staticProps);
    return Constructor;
}

function _inherits(subClass, superClass) {
    // 用父的原型创造一个实例
    subClass.prototype = Object.create(superClass && superClass.prototype, {
        constructor: {
            value: subClass,
            writable: true,
            configurable: true
        }
    });

    // 将子类的 __proto__ 设为 父类，在 es5 中没有这一步操作
    if (superClass) {
        _setPrototypeOf(subClass, superClass);
    }
}

var A = /*#__PURE__*/
function() {
    function A(name) {
        this.name = name;
    }

    _createClass(A, [{
        key: "sayName",
        value: function sayName() {
            console.log(this.name);
        }
    }]);

    return A;
}();

_defineProperty(A, "prop", 1);

var B = /*#__PURE__*/
function(_A) {
    _inherits(B, _A);

    function B(name, age) {
        var _this;

        // 调用父的构造函数，在 es5 中直接调用 A.call(this, name)
        _this = _getPrototypeOf(B).call(this, name); 
        _this.age = age;

        return _this;
    }

    _createClass(B, [{
        key: "sayDetail",
        value: function sayDetail() {
            // 调用父原型上的方法
            _get(_getPrototypeOf(B.prototype), "sayName", this).call(this);

            console.log(this.age);
        }
    }]);

    return B;
}(A);
```

**B.prototype 是一个以 A.prototype 为原型创造的实例，也就是说，B.prototype 是一个 A对象的实例。所以说 prototype 是用于类（构造函数） B，且通过 new 关键字创造的实例，默认没有 prototype 而是 __proto__。**

**以前的 es5 版本的继承，不会将子类的 __proto__人为设为父类（一般只有通过 new 关键字得到的实例才会有__proto__），但在 es6 规范出来之后多出来这一步。es6 中的方法在 prototype 上是无法枚举的属性。这个 __proto__ 不仅仅将两个类关联起来这么简单，它还实现了父类上的静态属性的继承。当取不到子类的静态属性时，会去父类找静态属性（子类的 __proto__），如下。**

![extends static](../../../../img/proto&prototype/extends_static.png)

## 三、网上神图

**copy 一张网上的超级复杂的[原型链神图](https://blog.csdn.net/cc18868876837/article/details/81211729)。**

![copy prototype](../../../../img/proto&prototype/copy_prototype.png)

**针对原博客再总结几点：**

**1.所有的 prototype 都是对象实例，所以 Foo.prototype 和 Function.prototype 的 __proto__ 都指向 Object.prototype，而 Object.prototype 的 __proto__ 属性为 null。**

**2.最左边一条是典型的原型链继承，对于实例 f1，f1 上找不到的属性会在 f1 的 __proto__ 对象里面找，依次往上。而对于构造函数来说，构造函数的 prototype 属性都是父类的实例。**

**3.这张图中的继承关系是 Foo extends Object，而不是 Foo extends Function，Foo 是 Function 的实例。此处调用 Foo.prototype.isPrototypeOf(f1) 和 Object.prototype.isPrototypeOf(f1) 都为true。这里有一个特殊情况，isPrototypeOf 无法判断基本变量，但是本质上，它和 f1 一样，也是从对象继承而来，万物皆对象。如图：**

![everything is object](../../../../img/proto&prototype/everything_is_object.png)

**4.Object 和 Foo 都是构造函数，所以都继承自 Function，不仅如此所有的构造函数都继承自 Function，如 String、Number、Date。它们的 __proto__ 都指向 Function.prototype。**

**5.Function 的构造函数是他自己，所以右上角会有一条奇怪的链，Function 的 __proto__ 和 prototype 都是 Function.prototype。**
