---
title: js 小技巧
date: 2019-03-06
tags:
    - js
---

### 一、js 重载

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

### 二、优化 switch

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

### 三、一行获取数组的交集、并集、差集

``` js
let a = [1, 2, 3];
let b = [3, 4, 5];

a.filter(item => b.includes(item)); // 交集
[...new Set([...a, ...b])]; // 并集
[...new Set([...a, ...b])].filter(item => !(a.includes(item) && b.includes(item))); // 差集
```

### 四、小数，字符串取整（from vue）

``` js
'1' >>> 0 // 1
1.3 >>> 0 // 1
```

### 五、数组乱序

``` js
const arr = [1,2,3,4,5,6,7,8];

// 第一种
arr.sort(() => 0.5 - Math.random());

// 第二种
function break(arr) {
	for(let i = 0, len = arr.length; i < len; i++) {
		let random = Math.random() * len >>> 0;
		let temp = arr[i];
		arr[i] = arr[random];
		arr[random] = temp;
	}
}
```

### 六、获取 0-9a-z 随机数（from redux）

``` js
function getHash() {
	return Math.random().toString(36).slice(2);
}

getHash(); // "l8jlqzaun7"
```