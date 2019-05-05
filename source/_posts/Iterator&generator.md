---
title: 迭代器与生成器
date: 2019-05-03
tags:
    - js
    - es6
---

## 一、迭代器规范

**一个标准的遍历器需要拥有以下属性，并且通过查看 for-of 的转码能够知道这些属性都是怎么样被调用的。**

``` js
Iterator [必须]
	next() {method}: 取得下一个IteratorResult
Iterator [可选]
	return() {method}: 停止迭代并返回IteratorResult
	throw() {method}: 通知错误并返回IteratorResult

IteratorResult
	value {property}: 当前的迭代值或最终的返回值
		（如果它的值为`undefined`，是可选的）
	done {property}: 布尔值，指示完成的状态

// for-of 转码
for (var _iterator = it[Symbol.iterator](), _step; (_step = _iterator.next()).done;) {
    var a = _step.value;
    console.log(a);
}
```

**根据上面的规范，我们可以自己定义一个可迭代的对象，并用 for-of 来遍历它。**

``` js
let obj = {
    0: 0,
    1: 1,
    length: 2,
    [Symbol.iterator] () {
        let _obj = this, index = 0, length = _obj.length;

        return {
            [Symbol.iterator] () {return this},
            next() {
				return index < length ?
                { value: `No.${index} is ${_obj[index++]}`, done: false }:
                { value: 'undefined', done: true};
			},

			return() {
				console.log(
					"iterator return"
				);
				return { value: _obj[index], done: true };
			}
        }
    }
}

for(let item of obj) {
	console.log(item);
    break;
}

// No.0 is 0
// iterator return

console.log([...obj]);
//  ["No.0 is 0", "No.1 is 1"]
```

## 二、Generator
### Generator

**Generator 函数可以在运行期间暂停它自己，还可以立即或者稍后继续运行，而且 Generator 函数生成的是一个迭代器，所以可以放入 for-of 中循环。**

``` js
function* gen() {
    yield 1;
    yield 2;
    yield 3;
}

for(let item of gen()) {
    console.log(item);
}
// 1， 2， 3

console.log([...gen()]);
//  [1, 2, 3]
```

**这让在对象上添加迭代器变得简单。**

``` js
let obj = {
    0: 0,
    1: 1,
    length: 2,
    *[Symbol.iterator] () {
        let index = 0, length = this.length;

        while(index < length) {
            yield `No.${index} is ${this[index++]}`;
        }
    }
}

console.log([...obj]);
//  ["No.0 is 0", "No.1 is 1"]
```

### yield *

**一个 * 使 yield 成为一个机制非常不同的 yield *，称为 yield 委托。yield * ..需要一个可迭代对象；然后它调用这个可迭代对象的迭代器，并将它自己的宿主 generator 的控制权委托给那个迭代器，直到它被耗尽。**

``` js
function *foo() {
	yield *[1,2,3];
}

// 等同于
function *foo() {
	yield 1;
	yield 2;
	yield 3;
}
```

**如果 yield 委托的是一个 Generator 函数，且委托的 Generator 函数中有返回值，返回值会被前一个函数的 yield * 捕获。**
``` js
function *foo() {
	yield *[1,2,3];
	return 4;
}

function *bar() {
	var x = yield *foo();
	console.log( "x:", x );
}

for (var v of bar()) {
	console.log( v );
}

// 1
// 2
// 3
// x: 4
```

### 转译 Generator

**以下为 babel 中比较系统得显示了 Generator 的转译，一个比较系统的遍历器。**

``` js
// 源代码
function* bar() {
    yield 1;
    yield 2
    yield 3
    yield 4;
}

// babel 转义代码
var IteratorPrototype = {};
IteratorPrototype[iteratorSymbol] = function() {
    return this;
}

function Generator() {}
function GeneratorFunction() {}
function GeneratorFunctionPrototype() {}

var Gp = GeneratorFunctionPrototype.prototype = Generator.prototype = Object.create(IteratorPrototype);
GeneratorFunction.prototype = Gp.constructor = GeneratorFunctionPrototype;
GeneratorFunctionPrototype.constructor = GeneratorFunction;
GeneratorFunctionPrototype[toStringTagSymbol] = GeneratorFunction.displayName = "GeneratorFunction";

function mark(genFun) {
    Object.setPrototypeOf(genFun, GeneratorFunctionPrototype);
    genFun.prototype = Object.create(Gp);
    return genFun;
}
// 这里创建了一个奇怪的原型链
GeneratorFunction.prototype = GeneratorFunctionPrototype;
GeneratorFunctionPrototype.prototype = Object.create(IteratorPrototype);
genFun.__proto__ = GeneratorFunctionPrototype = GeneratorFunction.prototype;
genFun.prototype = Object.create(GeneratorFunctionPrototype.prototype);

// 所以，genFun 是一个 GeneratorFunction 的实例
// genFun 继承自 Generator(GeneratorFunctionPrototype), Generator(GeneratorFunctionPrototype) 又继承自 Iterator

// 在 Iterator 实例上个挂载迭代器的方法
["next", "throw", "return"].forEach(function(method) {
    Gp[method] = function(arg) {
        return this._invoke(method, arg);
    };
});

function wrap(innerFn, outerFn, self, tryLocsList) {
    // If outerFn provided and outerFn.prototype is a Generator, then outerFn.prototype instanceof Generator.
    var protoGenerator = outerFn && outerFn.prototype instanceof Generator ? outerFn : Generator;
    // 创建一个 genFun 实例
    var generator = Object.create(protoGenerator.prototype);
    var context = new Context(tryLocsList || []); // 一个全局变量

    // The ._invoke method unifies the implementations of the .next,
    // .throw, and .return methods.
    generator._invoke = makeInvokeMethod(innerFn, self, context);

    return generator;
}

var GenStateSuspendedStart = "suspendedStart";
var GenStateSuspendedYield = "suspendedYield";
var GenStateExecuting = "executing";
var GenStateCompleted = "completed";

function makeInvokeMethod(innerFn, self, context) {
    var state = GenStateSuspendedStart;

    return function invoke(method, arg) {
        context.method = method;
        context.arg = arg;

        while (true) {
            if (context.method === "next") {
                // Setting context._sent for legacy support of Babel's
                // function.sent implementation.
                context.sent = context._sent = context.arg;

            } else if (context.method === "throw") {
                if (state === GenStateSuspendedStart) {
                    state = GenStateCompleted;
                    throw context.arg;
                }
                context.dispatchException(context.arg);
            } else if (context.method === "return") {
                context.abrupt("return", context.arg);
            }

            state = GenStateExecuting;

            // 调用 innerFn, 拿到返回值
            var record = tryCatch(innerFn, self, context);
            if (record.type === "normal") {
                // If an exception is thrown from innerFn, we leave state ===
                // GenStateExecuting and loop back for another invocation.
                state = context.done ? GenStateCompleted : GenStateSuspendedYield;

                if (record.arg === ContinueSentinel) {
                    continue;
                }

                return {
                    value: record.arg,
                    done: context.done
                };

            } else if (record.type === "throw") {
                state = GenStateCompleted;
                // Dispatch the exception by looping back around to the
                // context.dispatchException(context.arg) call above.
                context.method = "throw";
                context.arg = record.arg;
            }
        }
    }
}

function tryCatch(fn, obj, arg) {
    try {
        return {
            type: "normal",
            arg: fn.call(obj, arg)
        };
    } catch (err) {
        return {
            type: "throw",
            arg: err
        };
    }
}

// 主函数
"use strict";

// outerFn
var _marked =
/*#__PURE__*/
mark(bar);

function bar() {
    return wrap(function bar$(_context) {
        // innerFn
        while (1) {
            switch (_context.prev = _context.next) {
                case 0:
                _context.next = 2;
                return 1;

                case 2:
                _context.next = 4;
                return 2;

                case 4:
                _context.next = 6;
                return 3;

                case 6:
                _context.next = 8;
                return 4;

                case 8:
                return _context.abrupt("return", 5);

                case 9:
                case "end":
                return _context.stop();
            }
        }
    }, _marked);
}
```