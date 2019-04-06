---
title: 使用 async 同步处理 websocket 返回
date: 2019-03-17
tags:
    - js
    - websocket
    - async
---

**在使用 websocket（下文叫ws） 与交互时，使用 send 方法发送数据，监听 message 事件接受数据，以前的做法是在 message 触发时触发回调函数，而在 es6 中也可以使用 async 同步处理数据。**

## 一、确定 ws 数据

**ws 不像 http，当你用 ws 发送了2条请求，当然也会收到2条响应，但是你不知道的是，哪一条响应对应哪一条请求。**

**在开发时，我一般会在协议里带上 method,以此来标记响应对应的请求**

``` js
// 请求数据
{
    method: 'getEmail',
    data: {
        id: 1
    }
}

// 响应的数据
{
    method: 'getEmail',
    data: {
        email: 'xxx@xx.com'
    }
}
```

**不过当发送了多个相同 method 的请求还是无法区分响应对应哪一条请求，可以在协议里加上id，使其在每次发送 ws 请求时自增长，并且响应时带上这个 id。若 id 相同则为对应的响应**

## 二、处理 message

### 回调函数

**以前会定义一个 key 为 method，value 为对应回调函数的 json 来处理响应。这种方法就让业务逻辑中多了一层回调函数。**

``` js
// 发送
ws.send(JSON.stringify(
    { method: 'getEmail' }
));

// 回调
let wsCb = {
    getEmail(data) {
        // 回调逻辑
    }
}

ws.onmessage = function (e) {
    let {data: {method}} = e;

    let fn = wsCb?.[method];
    fn && fn();
}
```

### async 处理回调

**使用 async 可以将异步改同步，并且减少回调函数的使用。**

``` js
// 回调
let wsCb = {}

ws.onmessage = function (e) {
    let data = e.data;
    let method = data.method;

    let fn = wsCb?.[method];
    delete wsCb.[method];

    fn && fn(data);
}

let wsSend = (data) => {
    ws.send(JSON.stringify(data);

    let methos = data.methos;
    return new Promise (res, rej) {
        wsCb[methos] = function (data) {
            res(data);
        }
    }
}

// 主业务
(async () => {
    let req = { method: 'getEmail' };

    let res = await wsSend(req);

    console.log(res)
})()
```

### 使用事件处理函数

**js 中有很多事件处理的库，可以用来管理自定义的事件，这里就拿 jQuery 的事件绑定来做例子（jQuery 的实现方式见之前的[博客](../../../../2019/03/04/jquery-technology-insider/)）。**

``` js
class ws {
    constructor () {
        let ws = new WebSocket();
        
        this.ws = ws;
        this.initEvent();
    }

    initEvent() {
        let ws = this.ws;

        ws.onopen = function () {};
        ws.onmessage = function (e) {
            let data = e.data;
            let method = data.method;

            $(this).trigger(method, [data]);
        };
    }

    send(data) {
        let ws = this.ws;
        let method = data.method;

        ws.send(data);

        return new Promise (res, rej) {
            $(this).one(method, (data) => {
                res(data);
            })
        }
    } 
}

// 主业务
(async () => {
    let ws = new ws();
    let req = { method: 'getEmail' };

    let res = await ws.send(req);

    console.log(res)
})()
```