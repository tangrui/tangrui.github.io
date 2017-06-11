---
ID: 64
title: window.eval() 和 eval() 的区别？
author: 唐睿
date: 2006-05-26 10:29:47 +0800
categories: [技术]
tags: [编程, JavaScript, 浏览器]
layout: post
published: true
---

前两天作了一个试验，把 JavaScript 文件当作一个字符串读到浏览器的内存中，然后使用 eval() 方法来解析，对某些内容的 JavaScript 文件（我用的是 [prototype.js](http://prototype.conio.net)）会有异常抛出。好像浏览器的 JavaScript 引擎总是试图去执行一些本来是声名性的语句，例如定义 function。在我的试验中，当解析到 prototype 给 Object 添加的 extend 方法时，说第一个参数 destination 没有定义。

当研究了另一个非常有名的 Ajax 框架 [Dojo](http://dojotoolkit.org) 之后发现，如果使用 window.eval() 方法，上面的问题就没有了，解析可以非常顺利地进行，就像把 JavaScript 文件通过 `<script>` 标签引用近来一样。这就产生了一个问题难道 window.eval() 和 eval() 还不一样？

众所周知，根据 JavaScript 规范，其实那些我们经常用到的理解意义上的全局方法，比如 alert()，其实也是有它的依存对象的，这个对象就是 [window](http://www.w3schools.com/htmldom/dom_obj_window.asp)，也就是说 alert() 就等于 window.alert()，当然 eval() 应该也不例外，但是为什么在这里表现却是如此的不同？

上网 Google 了一下，在这篇[文章](http://blog.csdn.net/aimingoo/archive/2006/02/13/597661.aspx)中找到一点线索，但是仍然不能解决我的疑问。

文章中说，本来 JavaScript（其实他说的是 JScript）应该有两个方法，一个是 eval() 另一个是 execScript()（但是后来在 [w3shools](http://www.w3schools.com/htmldom/dom_obj_window.asp) 上面却发现 window 只有 execScript() 方法，却没有 eval()！），前者是在函数调用的上下文中执行，就是说如果你在 `function myFun() {}` 中使用 eval() 那么它就只能访问 myFun 中的成员；而后者是在全局的上下文中执行。文章后面还说到了一些 Firefox 相关的问题，但是我发现 IE 的行为在这一点上跟 Firefox 是一样的。如果这两个方法仅仅有这点不同的话，那么真的可以像文章中说得那样理解：

* 如果在函数中使用 window.eval() 来执行，则使用全局上下文环境。
* 如果使用 eval() 来执行，则使用当前函数的上下文环境。

但是为什么使用函数上下文环境解析就会试图去执行声名性语句呢？不得其解。
