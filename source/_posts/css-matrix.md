---
title: css 矩阵和 tramsform 之间换算
date: 2019-03-03
tags:
    - css
    - matrix
---

**在学数学时，同学们总在一起打趣：学这么多的函数运算有什么用，以后买菜总不能先列一个函数，算出该付多少钱吧。可谁知，在科技高速发展的时代，打趣的话也快要“实现”了。**
**在 css3 中，退出了新属性——— matrix(矩阵)，这不正是大一时候学的线性代数吗（好在大一的时候还不是很皮，都认真听讲了）**

### 矩阵计算

**css3 中的 matrix 用来代替多个 transfrom 属性或者一些 transfrom 无法实现的效果，就来看一看 matrix 是怎么实现变换的吧。**

**首先来看一下矩阵的参数：**
``` js
transform: matrix(a,b,c,d,e,f);
```

**这6个参数提供了一个 3 * 3 的矩阵，而经过这个矩阵能够计算出变换前的 x，y 值与变换后的 x，y 值的关系。反应式如下：**

![matrix](../../../../img/css-matrix/matrix.gif)

**其中 x，y 为变化前的坐标，ax+cy+e 为变换后的 x'，bx+dy+f 表示变换后的 y'。**
**举个例子，如果我们有一个矩形有 A(0, 0) B(200, 0) C(0, 100) D(200, 100) 四个点组成，经过矩阵转换后，我们希望得到如下变换：**

![transfrom](../../../../img/css-matrix/transfrom.png)

**在 css 中，坐标轴的零点都是左上角，x 轴和 y 轴的方向也与数学中的坐标轴有所不同。也就是我们需要得到的矩阵坐标为 A'(300, 100) B'(300, 300) C'(400, 100) D'(400, 300)，那么就可以根据公式 ax+cy+e=x'，bx+dy+f=y' 算出 a,b,c,d,e,f 各为多少。**

``` js
// 将 A(0, 0) 和 A'(300, 100) 两点带入 ax+cy+e=x' bx+dy+f=y'
a * 0 + c * 0 + e = 300;
b * 0 + d * 0 + f = 100;

e = 300;
f = 100;

// 以此类推，将四个点全部带入计算得出 a,b,c,d,e,f 四个值
// 此例中为 
transform: matrix(0, 1, 1, 0, 300, 100);
```

**最终矩阵变换的效果图如下**

<iframe src="https://howgraceu.github.io/demo/matrix/" style="margin-left: -2em; width: 100%; height: 225px;"><iframe>

### matrix 与 transfrom 各个属性的转换。

**transfrom 的各个属性能够与 matrix 相互转换。**

``` css
/* matrix 默认值 */
transform: matrix(1, 0, 0, 1, 0, 0);

/* translate */
transform: translate(x, y);
transform: matrix(0, 0, 0, 0, x, y);

/* scale */
transform: scale(x, y);
transform: matrix(x, 0 * x, 0 * y, y, 0, 0);

/* rotate */
rotate: rotate(θ);
transform: matrix(cosθ, sinθ, -sinθ, cosθ, 0, 0);
```

**张鑫旭大神的博客[理解CSS3 transform中的Matrix](https://www.zhangxinxu.com/wordpress/2012/06/css3-transform-matrix-%E7%9F%A9%E9%98%B5/) 一文对此有详细的解释。**

**但是需要注意的是，当 scale 和 rotate 一起使用的时候，两者的值需要相乘。**

``` css
/* scale 2 */
transform: scale(x, y);
rotate: rotate(θ);

transform: matrix(cosθ * x, sinθ * x, -sinθ * y, cosθ * y, 0, 0);
```

### matrix 与 transfrom 转换函数。

**根据 matrix 与 transfrom 之间的关系，可以提出一个函数来进行两者之间的转换（转换顺序为先 transfrom 再 rotate）。**

``` js
const PI = Math.PI;

function encodeMatrix({translateX = 0, translateY = 0, rotate=0, scaleX=1, scaleY=1} = {}) {
    let rad = rotate * PI / 180;
    let a = scaleX * Math.cos(rad);
    let b = scaleX * Math.sin(rad);
    let c = scaleY * -Math.sin(rad);
    let d = scaleY * Math.cos(rad);
    let e = translateX;
    let f = translateY;

    return [a,b,c,d,e,f]
}

function decodeMatrix([a = 1, b = 0, c = 0, d = 1, e = 0, f = 0] = []) {
    let translateX = e;
    let translateY = f;
    let scaleX = Math.round(Math.sqrt(Math.pow(a, 2) + Math.pow(b, 2)));
    let scaleY = Math.round(Math.sqrt(Math.pow(c, 2) + Math.pow(d, 2)));
    let radC = Math.acos(a / scaleX);
    let radS = Math.asin(b / scaleX);

    let rad
    if (b >= 0) {
        rad = radC;
    } else if (b < 0) {
        rad = 2 * PI - radC;
    }

    let rotate = Math.round(rad * 180 / Math.PI);

    return {translateX, translateY, rotate, scaleX, scaleY}
}

let matrix = encodeMatrix({translateX: 30, translateY: 40, rotate: 45, scaleX: 2, scaleY: 3});
//  [1.4142135623730951, 1.414213562373095, -2.1213203435596424, 2.121320343559643, 30, 40]

let transfrom = decodeMatrix([1.4142135623730951, 1.414213562373095, -2.1213203435596424, 2.121320343559643, 30, 40]);
// {translateX: 30, translateY: 40, rotate: 45, scaleX: 2, scaleY: 3}
```

**上述函数在自己写的一个 [svg-editor](https://howgraceu.github.io/demo/svg/) 的例子中使用，但是在使用过程中发现如果缩放倍数不为整数时，可能会得到不精确的结果，导致最后结果偏差较大。**

**最后附上一个用于转化 transfrom 和 matrix 的网页，[meyerweb-matrix](https://meyerweb.com/eric/tools/matrix/)**