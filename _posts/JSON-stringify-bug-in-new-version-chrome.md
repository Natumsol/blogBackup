title: 新版本Chrome序列化performanceTiming的bug
date: 2015-06-15 15:06:32
categories: 技术
tags: 
     - Chrome
     - JSON
---
昨天在Chrome 里序列化`window.performance.timing`对象时发现了一个Bug。此Bug只出现在新版的Chrome（Chrome 41+）中，问题重现如下：

## bug说明
``` javascript
//	Google Chrome	45.0.2427.7 (正式版本) dev-m （64 位）
//	修订版本	3dbfc97efe069741d1507c995573283ce6912406-refs/branch-heads/2427@{#10}
//	操作系统	Windows 
//	Blink	537.36 (@196784)
//	JavaScript	V8 4.5.42
// 	Flash	18.0.0.160

JSON.stringify({ name: "Yoghurt" })//正常输出"{"name":"Yoghurt"}"
JSON.stringify(window.performance.timing)//bug出现，输出为"{}"
```
<!-- more -->
经过搜索得知此为新版Chrome的一个[bug](http://src.chromium.org/viewvc/blink?view=revision&revision=196854)，产生的原因我找到了一个比较清晰的[解释](https://code.google.com/p/chromium/issues/detail?id=467366#c5)如下：
>It's by design.  JSON.stringify does not encode DOM attributes on prototype chains.  Since almost all DOM attributes are (or must be) on prototype chains, so you should not use JSON.stringify directly for DOM attributes.

中文意思大概是`JSON.stringify`设计成不会序列化原型链上的DOM属性，而Chrome 41+后，`window.performance.timing`对象变为DOM原型链上的属性了，故不会被序列化。

## 解决方案
这个bug解决起来也不难，重新写一个stringify函数代替原生的JSON.stringify就可以了，具体代码为：

```javascript
// Stringifies a DOM object.  This function stringifies not only own properties
// but also DOM attributes which are on a prototype chain.  Note that
// JSON.stringify only stringifies own properties.
function stringifyDOMObject(object)
{
    function deepCopy(src) {
        if (typeof src != "object")

            return src;
        var dst = Array.isArray(src) ? [] : {};
        for (var property in src) {
            dst[property] = deepCopy(src[property]);
        }
        return dst;
    }
    return JSON.stringify(deepCopy(object));
}
```

## 测试结果
```javascript 
stringifyDOMObject(window.performance.timing)
//正常序列化为"{"navigationStart":1434361277562,"unloadEventStart":1434361277688,"unloadEventEnd":1434361277688,"redirectStart":0...}"
```

----
## 更新
在实际中测试发现，如果被序列化的对象有`toJSON`函数属性，就会造成无法正常序列化的问题。
Bug fixed后的代码如下：
```javascript
// Stringifies a DOM object.  This function stringifies not only own properties
// but also DOM attributes which are on a prototype chain.  Note that
// JSON.stringify only stringifies own properties.
function stringifyDOMObject(object)
{
    function deepCopy(src) {
        if (typeof src != "object")
            return src;
        var dst = Array.isArray(src) ? [] : {};
        for (var property in src) {
            if(property !== "toJSON")
                dst[property] = deepCopy(src[property]);
        }
        return dst;
    }
    return JSON.stringify(deepCopy(object));
}
```
完。
