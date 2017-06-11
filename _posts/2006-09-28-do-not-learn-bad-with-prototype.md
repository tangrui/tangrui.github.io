---
ID: 33
title: 不要跟 prototype 一起学坏
author: 唐睿
date: 2006-09-28 18:51:37 +0800
categories: [技术]
tags: [编程, JavaScript, prototypejs]
layout: post
published: true
---

新版本的 JSON 修改了 API，将 JSON.stringify() 和 JSON.parse() 两个方法都注入到了 JavaScript 的内建对象里面，前者变成了 Object.toJSONString()，而后者变成了 String.parseJSON()。

不可否认，经过这样的修改，JSON 的接口确实比以前漂亮且好用多了，对于这样的对象：

```javascript
var obj = new Object;
var arr = new Array();
arr[0] = 'a';
arr[1] = 'b';
obj['key1'] = arr;
obj['key2'] = 'c';
```

以前的代码得这样写：

```javascript
var jsonString = JSON.stringify(obj);
var myObj = JSON.parse(jsonString);
```

而现在的代码是：

```javascript
var jsonString = obj.toJSONString();
var myObj = jsonString.parseJSON();
```

前者 JSON 作为一个工具类，提供一组方法实现转换功能，而后者让你觉得对象天生就具有这样的方法。如果对于编译语言来说，这可能没什么问题，但是对于 JavaScript 问题就来了。考虑下面的代码片段：

```javascript
var obj = new Object();
for (var k in obj) {
  document.write(k + &quot;&lt;br /&gt;&quot;);
}
```

如果是一个干净（没有包含任何其他 JavaScript 类库）的 JavaScript 执行环境，这段代码应该不会输出任何内容，因为 obj 是空的；但是当代码中包含了新版本的 JSON 后，你会发现结果是 toJSONString。很多情况下我们会用 JavaScript 的 Object 来模拟 Map，那么上面的这段代码的功能就是遍历此 Map，一个空的 Map 怎么会凭空出来了一个成员？这正是由于新版本的 JSON 的实现所决定的：

```javascript
Object.prototype.toJSONString = function () {}
```

JSON 并不是第一个采用这种写法的 JavaScript 类库，始作俑者就是我文章标题中提到的 prototype。prototype 为 Object 注入了 extend 和 instanceof 等方法来支持 JavaScript 的面向对象扩展，这样写的后果就是造成了非常严重的兼容性问题。你有没有注意到为什么有些 JavaScript 的类库特别声明“使用了 prototype”或者说是“与 prototype 兼容”？因为如果他使用了 prototype 那么你需要在用此类库的时候格外小心，像上面那样的代码不能正常工作；而如果说其与 prototype 兼容，那么可能该类库并没有使用 prototype 但是你使用 prototype 并不会影响他的正常运行。

prototype 固然是个成功且应用广泛的 JavaScript 框架，但是这并不能说明他解决问题的思路就是正确的。开发编译语言程序的人们都知道要以组织或公司的名称作为命名空间前缀，来避免冲突，奈何如此著名的两大 JavaScript 框架提供方反而积极去打破这个规则呢？难道实现了功能就可以不考虑兼容性的问题吗？
