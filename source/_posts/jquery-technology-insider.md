---
title: jquery 技术内幕
date: 2019-03-04
tags:
    - js
    - jquery
---

<script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.js"></script>

**记录前端鼻祖 ———— jquery 的 js 技巧**

### 一、jquery 对象

**在没有 es6 的 Class 之前，js 中都使用 function 以及他的 propotype 来实现面向对象。但是尽管如此，总有人在初始化实例时，忘记加上 new 关键字，这就会造成意外的全局污染。**

``` js
function Person(name) {
    this.name = name;
}

// 正常使用
var man = new Person('老王');
console.log(man.name); // 打印 老王

// 忘记带 new
var man = Person('老王');
console.log(window.name); // window 添加了一个变量 name
```

**这就会在你不知情的情况下，定义了一个全局变量。而 jquery 在生成新对象的时候，从来都没有带上 new 关键字，jquery 在创建对象时使用了一个技巧。**

``` js
function Person(name) {
    this.name = name;
}

function createPerson(name) {
    return new Person(name);
}

var man = new createPerson('老王');
var woman = createPerson('小丽');

console.log(man.name); // 打印 老王
console.log(woman.name); // 打印 小丽
console.log(man.prototype === woman.prototype); // 打印 true
```

**使用 new 关键字后，如果函数 return 了一个对象，返回对象，若返回的不是对象则创建一个新对象返回。**

**在 createPerson 函数返回了一个 new Preson，使得即使不使用 new 关键字也可以创建对象实例，既保证创建对象时的安全，也方便调用。**

**源码中的代码：**

![jquery_init](../../../../img/jquery_technology_insider/jquery_init.png)

### 二、jquery 原型

**在使用 jquery 的插件时，在 $.fn 上扩展方法，对象 $() 上就能访问刚刚挂载的方法。**

``` js
$.fn.log123 = function () {
    console.log(123)
}

$().log123() // 打印 123
```

**再结合上面返回的 $.fn.init 实例，可见 $.fn 就是 $.fn.init.prototype。的确在源码中也做了这样的定义，使得返回的对象可以拿到原型上的方法。**

![jquery_init_propotype](../../../../img/jquery_technology_insider/jquery_init_propotype.png)

**再附上一张 jquery 非常绕的原型图：**

![jquery_propotype](../../../../img/jquery_technology_insider/jquery_propotype.png)

### 三、类型判断

**在判断一个对象是 json 还是 array 的时候，公认的有效的方法是使用 Object.prototype.toString.call( Obj )。对于得到的字符串还要进行 slice 才能拿到对象真正的类型。**

**在 jquery 中，建立了一个 map 来获取对象的类型。**

``` js
const class2type = {};

"Boolean Number String Function Array Date RegExp Object Error Symbol".split( " " ).forEach(name => {
    class2type[ "[object " + name + "]" ] = name.toLowerCase();
})

/*
* 得到
* {
*   "[object Boolean]": "boolean",
*   "[object Number]": "number",
*   "[object String]": "string",
*   "[object Function]": "function",
*   "[object Array]": "array",
*   "[object Date]": "date",
*   "[object RegExp]": "regexp",
*   "[object Object]": "object",
*   "[object Error]": "error",
*   "[object Symbol]": "symbol"
* }
*/  

const toString = class2type.toString;

function toType( obj ) {
	if ( obj == null ) {
		return obj + "";
	}

	return typeof obj === "object" || typeof obj === "function" ?
		class2type[ toString.call( obj ) ] || "object" :
		typeof obj;
}
```

### 四、绑定事件

**jQuery 的绑定事件不仅可以给元素绑，还可以给对象绑，还可以根据参数来判断是否进行事件委托，还可以绑定自定义的事件，多才多艺的一个成员。他会在绑定的时候将需要触发的事件绑定在对象的一个属性上，保存在内存中。**

``` js
jQuery.event.add = function(elem, types, handler, data, selector) {
    var // 省略一堆属性
        elemData = dataPriv.get(elem); // 在获取将要绑定的元素的 jQuery.expando（这个值为一个jQuery开头的唯一字符串） 属性，若不存在这属性则赋值为空对象
    // Make sure that the handler has a unique ID, used to find/remove it later
    if (!handler.guid) {
        handler.guid = jQuery.guid++;
    }

    // 将事件函数推入内存中
    // Init the element's event structure and main handler, if this is the first
    if (!(events = elemData.events)) {
        events = elemData.events = {};
    }
    if (!(eventHandle = elemData.handle)) {
        eventHandle = elemData.handle = function(e) {

            // Discard the second event of a jQuery.event.trigger() and
            // when an event is called after a page has unloaded
            return typeof jQuery !== "undefined" && jQuery.event.triggered !== e.type ? jQuery.event.dispatch.apply(elem, arguments) : undefined;
        };
    }

    handleObj = jQuery.extend({
        type: type,
        origType: origType,
        data: data,
        handler: handler,
        guid: handler.guid,
        selector: selector,
        needsContext: selector && jQuery.expr.match.needsContext.test(selector),
        namespace: namespaces.join(".")
    }, handleObjIn);

    // Init the event handler queue if we're the first
    if (!(handlers = events[type])) {
        handlers = events[type] = [];
        handlers.delegateCount = 0;

        // Only use addEventListener if the special events handler returns false
        if (!special.setup || special.setup.call(elem, data, namespaces, eventHandle) === false) {

            if (elem.addEventListener) {
                elem.addEventListener(type, eventHandle);
            }
        }
    }

    // Add to the element's handler list, delegates in front
    if (selector) {
        handlers.splice(handlers.delegateCount++, 0, handleObj);
    } else {
        handlers.push(handleObj);
    }
}

```

**推入后，可以在绑定事件的对象属性中看到事件队列。**

![jquery_eventHandle](../../../../img/jquery_technology_insider/event_handle.png)

**其中比较有意思的就是 namespace 了，在触发和删除事件的时候会用到，下面是触发事件的函数。**

``` js
jQuery.extend(jQuery.event, {

    // 封装 event 对象，绑上 isTrigger, namespace等属性
    trigger: function(event, data, elem, onlyHandlers) {
        var //定义一堆变量

        cur = lastElement = tmp = elem = elem || document; // 触发事件的对象

        if (type.indexOf(".") > -1) {

            // Namespaced trigger; create a regexp to match event type in handle()
            namespaces = type.split(".");
            type = namespaces.shift();
            namespaces.sort();
        }

        // Caller can pass in a jQuery.Event object, Object, or just an event type string
        event = event[jQuery.expando] ? event : new jQuery.Event(type,typeof event === "object" && event);

        // Trigger bitmask: & 1 for native handlers; & 2 for jQuery (always true)
        event.isTrigger = onlyHandlers ? 2 : 3; // 使用 trigger 函数触发时，onlyHandlers为 undefined
        event.namespace = namespaces.join(".");
        event.rnamespace = event.namespace ? new RegExp("(^|\\.)" + namespaces.join("\\.(?:.*\\.|)") + "(\\.|$)") : null; // 生成 命名空间正则

        handle = (dataPriv.get(cur, "events") || {})[event.type] && dataPriv.get(cur, "handle");
        if (handle) {
            handle.apply(cur, data); // 触发了add时候绑定的函数，走到了 jQuery.event.dispatch
        }
    }
}

jQuery.event.dispatch = function(nativeEvent) {
    var handlers = (dataPriv.get(this, "events") || {})[event.type] || []; // 从缓存中获取绑定的事件

    // Determine handlers
    handlerQueue = jQuery.event.handlers.call(this, event, handlers);

    while ((matched = handlerQueue[i++]) && !event.isPropagationStopped()) { // 遍历事件队列
        event.currentTarget = matched.elem;

        j = 0;
        while ((handleObj = matched.handlers[j++]) && !event.isImmediatePropagationStopped()) {

            // Triggered event must either 1) have no namespace, or 2) have namespace(s)
            // a subset or equal to those in the bound event (both can have no namespace).
            if (!event.rnamespace || event.rnamespace.test(handleObj.namespace)) { // 正则add时存下的handle对象的namespace属性，若匹配上则触发

                event.handleObj = handleObj;
                event.data = handleObj.data;

                ret = ((jQuery.event.special[handleObj.origType] || {}).handle || handleObj.handler).apply(matched.elem, args);

                if (ret !== undefined) {
                    if ((event.result = ret) === false) {
                        event.preventDefault();
                        event.stopPropagation();
                    }
                }
            }
        }
    }
}
```