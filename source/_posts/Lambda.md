---
title: 函数式编程 - 函子
date: 2019-03-24
tags:
    - js
    - lambda
---

**函数式编程通过管道把数据在一系列纯函数间传递。但是，控制流（control flow）、异常处理（error handling）、异步操作（asynchronous actions）和状态（state）呢？还有更棘手的作用（effects）呢？我们都将借助函子帮我们完成。**

**函子是函数式编程中最重要的数据类型，也是基本的运算单位和功能单位。**

## 容器

**我们创建一个容器，这个容器就职于装载值，并且不会向容器添加新属性或者方法。**

``` js
class Container {
    constructor (value) {
        this.__value = value;
    }

    static of(value) {
        return new this(value);
    }

    map(f) {
        return Container.of(f(this.__value));
    }
}

let concat = x => y => y.concat(x);
let prop = x => y => y[x];

Container.of("bombs") // Container('bombs')
    .map(concat(' away')) // Container('bombs away')
    .map(prop('length')) // Container(10)
```

**这样我们就能够在不离开 Container 的情况下操作容器里面的值，并且这很函数式。**

## Maybe

**错误处理时，可以使用 Maybe 函子**

``` js
class Maybe extends Container {
    constructor (value) {
        super(value);
    }

    isNothing() {
        return (this.__value === null || this.__value === undefined);
    }

    map(f) {
        return this.isNothing() ? Maybe.of(null) : Maybe.of(f);
    }
}

let retNull = () => null;

Maybe.of("bombs") // Maybe('bombs')
    .map(concat(' away')) // Maybe('bombs away')
    .map(retNull) // Maybe(null)
    .map(prop('length')) // 当检测到 value 为null，不运行 prop 函数，避免函数运行时报错，返回 Maybe(null)
```

## Either

**数学中没有 if...else，我们可以用 either 纯函数来处理 if...else**

``` js
class Either extends Container {
    constructor(left, right) {
        super();
        this.left = left;
        this.right = right;
    }

    static of(left, right) {
        return  new this(left, right);
    }

    map(f) {
        return this.right ? 
            Either.of(this.left, f(this.right)) :
            Either.of(f(this.left), this.right);
    }
}

let lenLtNum = num => str => str.length < num;
let lenLt4 = lenLtNum(4);

Either.of("jqx", "bombs") // Either("jqx", "bombs")
    .map(lenLt4) // Either("jqx", false)
    .map(concat(' away')) // Either("jqx away", false)
    .map(prop('length')) // Either(8, false)
```

## IO

**我们有很多的不纯的操作，如取 storage 缓存，向后台请求接口，我们把这些函数包装在IO函子中，让调用者替我们承当不纯的部分。**

``` js
class IO extends Container {
    constructor(f) {
        super(f);
    }

    map(f) {
        return new IO(_.compose(f, this.__value));
    }
}

let retWin = () => window;
let getInnerWidth = win => win.innerWidth;
let unsafeWidth = IO.of(retWin).map(getInnerWidth);

unsafeWidth.__value(); // 调用不纯函数得到窗口宽度
```

## Monad

**下面有两个不纯的函数，但是把他们包装一下，每次都返回 IO 函子，包装他们的函数是一个纯函数。**

``` js
const fs = require('fs');

let readFile = function(filename) {
    return new IO(function() {
        return fs.readFileSync(filename, 'utf-8');
    });
};

let print = function(x) {
    return new IO(function() {
        console.log(x);
        return x;
    });
}

let getUser = readFile('./user.txt') // IO{ val: f => user }
                .map(print) // IO{ val: f => IO { f => user }}

getUser.__value().__value(); // 拿到 user
```

**若 IO 函子 map 了一个不纯的函数，则会得到一个 IO 里面包装 IO 的函子，提供一个 Monad 函数处理这种情况。**

``` js
class Monad extends Container {
    join() {
        return this.__value;
    }

    flatMap(f) {
        return this.map(f).join();
    }
}

class IO extends Container {
    map(f) {
        return new IO(_.compose(f, this.__value));
    }

    flatMap(f) {
        // 这里要扩展父的 flatMap 是因为 IO 函子里面放的是函数，
        // 得将之运行一下，返回输出的 IO 扁平化，若函子里面放的是纯数据则不需要扩展
        return super.flatMap(f)();
    }
}

let getUser = readFile('./user.txt') // IO{ val: f => user }
                .flatMap(print) // IO{ val: f => IO { f => user }}

getUser.__value(); // 拿到 user

let getName = prop('name');

let getUserName = readFile('./user.txt') // IO{ val: f => user }
                .flatMap(print) // IO{ val: f => IO { f => user }}
                .map(getName)

getUserName.__value(); // 拿到 userName
```
