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

****