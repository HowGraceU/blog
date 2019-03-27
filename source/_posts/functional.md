---
title: 函数式编程
date: 2019-03-19
tags:
    - js
    - functional
---

<script src="https://cdn.bootcss.com/lodash.js/4.17.12-pre/lodash.js"></script>

**js中函数是一等公民，这句话意味着函数和其他数据类型一样，可以赋值给其他变量，也可以作为参数传入另一个函数，或者作为别的函数的返回值。**

# 纯函数

**对于相同的输入，永远会得到相同的输出，而且没有任何可观察的副作用，也不依赖外部环境的函数称之为纯函数**

``` js
let xs = [1, 2, 3, 4, 5];

// slice 是纯函数，固定输入得到固定输出，没有副作用
xs.slice(0, 3) // [1, 2, 3] xs不变
xs.slice(0, 3) // [1, 2, 3] xs不变
xs.splice(0, 3) === xs.splice(0, 3) // [1, 2, 3] xs变成[4, 5]
xs.splice(0, 3) === xs.splice(0, 3) // [4, 5] xs变成[]
```

## 纯函数的优点

### 可缓存性
**由于固定输入得到固定输出，所以可以将函数结果缓存起来。**

``` js
import _ from 'loadsh';
const sin = _.memorize(x => Math.sin(x));

sin(1);

// 第二次计算有了换成。速度会变快
sin(2);
```

### 可移植性

**命令式编程中“典型”的方法和过程都深深地根植于它们所在的环境中，通过状态、依赖和有效作用（available effects）达成；纯函数与此相反，它与环境无关，只要我们愿意，可以在任何地方运行它。**

``` js
// 不纯的
var signUp = function(attrs) {
  var user = saveUser(attrs);
  welcomeUser(user);
};

var saveUser = function(attrs) {
    var user = Db.save(attrs);
    ...
};

var welcomeUser = function(user) {
    Email(user, ...);
    ...
};

// 纯的
var signUp = function(Db, Email, attrs) {
  return function() {
    var user = saveUser(Db, attrs);
    welcomeUser(Email, user);
  };
};

var saveUser = function(Db, attrs) {
    ...
};

var welcomeUser = function(Email, user) {
    ...
};
```

### 可测试性

**纯函数让测试更加容易。只需简单地给函数一个输入，然后断言输出就好了。**

# 函数柯里化

**传给函数部分参数，返回一个函数处理剩下的参数。**

``` js
let comparedAge = (min, age) => min < age;

let initMinAge = (min) => {
    return (age) => comparedAge(min, age)
}

let checkAge19 = initMinAge(19);

checkAge19(15); // false
checkAge19(23); // true
```

**以下为个人查阅 lodash 源码，制作的简化版的 curry**

``` js
const curry = (func, partials, arity=func.length)=>{
    const wrapper = function() {
        let length = arguments.length;
        let args = Array(length);
        let index = length;

        while (index--) {
            args[index] = arguments[index];
        }

        if (partials) {
            args = args.concat(partials);
        }

        if (length < arity) {
            return curry(func, args, arity - length);
        }

        return func.apply(window, args.reverse());
    }
    return wrapper;
}

let abc = function(a, b, c) {
    return [a, b, c];
};

let a = curry(abc);

console.log(a(1)(2)(3)); // [1, 2, 3]

let b = a(4);
console.log(b(5)(6)); // [4, 5, 6]

let c = a(7)(8);
console.log(c(9)); // [7, 8, 9]
```

# 代码组合

**f 和 g 都是函数，x 是在它们之间通过“管道”传输的值。特性有结合律，符合结合律意味着不管你是把 g 和 h 分到一组，还是把 f 和 g 分到一组都不重要。**

``` js
var compose = function(f,g) {
    return function(x) {
        return f(g(x));
    };
};

// 结合律（associativity）
var associative = compose(f, compose(g, h)) == compose(compose(f, g), h);
// true
```

## pointfree

**函数无须提及将要操作的数据是什么样的。一等公民的函数、柯里化（curry）以及组合协作起来非常有助于实现这种模式。**

``` js
// 非 pointfree，因为提到了数据：word
var snakeCase = function (word) {
    return word.toLowerCase().replace(/\s+/ig, '_');
};

// pointfree
var replace = function (reg, replaceStr) {
    return function (str) {
		return str.replace(reg, replaceStr);
	}
}

var toLowerCase = function (str) {
	return str.toLowerCase();
}

var snakeCase = compose(replace(/\s+/ig, '_'), toLowerCase);

snakeCase('A B'); // a_b
```

## debug

**如果在 debug 组合的时候遇到了困难，那么可以使用下面这个实用的，但是不纯的 trace 函数来追踪代码的执行情况。**

``` js
var trace = curry(function(tag, x){
  	console.log(tag, x);
  	return x;
});

var snakeCase = compose(replace(/\s+/ig, '_'), compose(trace('[snakeCase]'), toLowerCase));

snakeCase('A B');
// log  [snakeCase] a b
// 返回 a_b
```

# 尾递归

**尾递归的判断标准是函数的最后一步是否调用自身。尾递归调用栈永远都是更新当前的栈帧，这样就避免了爆栈的危险，而浏览器并未支持，原因有 1.在引擎曾消除递归是隐式行为，程序员意识不到。2. 堆栈信息丢失导致开发者难以调试。我们可以将递归写成while。**

``` js
// 传统递归
function sum(n) {
    if (n === 1) return 1;
    return n + sum(n - 1);
}

sum(5);
// 调用栈
sum(5)
(5 + sum(4))
(5 + (4 + sum(3)))
(5 + (4 + (3 + sum(2))))
(5 + (4 + (3 + (2 + sum(1)))))
(5 + (4 + (3 + (2 + 1))))
(5 + (4 + (3 + 3)))
(5 + (4 + 6))
(5 + 10)
15

// 尾递归调用
function sum(n, total = 0) {
    if (n === 1) return n + total;
    return sum(n - 1, n + total);
}

sum(5);
// 调用栈，每次调用栈都指向新返回的函数
sum(5)
sum(4, 5)
sum(3, 9)
sum(2, 12)
sum(1, 14)
15
```