---
ID: 626
title: Java 栈上的 JavaScript
author: 唐睿
date: 2015-07-31 19:36:55 +0800
categories: [技术]
tags: [演讲, Lightening Talk, Java, JavaScript, Rhino, Nashorn, invokedynamic, CDE.IO, ShenJS 2015]
excerpt: >
  Enterprise JavaScript 是一个非常庞大的话题，会涉及到 JavaScript 跟各种 Java 中间件的互操作问题，这里仅就如何在一个完整的 Java 栈上以 JavaScript 作为首要编程语言来提高 Java 系统的开发效率展开讨论。
layout: post
published: true
---

{% youtube "https://youtu.be/S0imw6Ycc64" %}

*<small>注：该视频托管在 Youtube 上面，你懂得该怎么做。</small>*

这是笔者在 [ShenJS 2015](/2015/shenjs-2015) 上面所做的一个 Lighting Talk，当时的题目叫做 Enterprise JavaScript。其实那天准备了很多功能演示和代码片段，无奈时间太短，只把幻灯片过了一遍就没有时间了，于是在这里做一个完整的介绍。

Enterprise JavaScript 是一个非常庞大的话题，会涉及到 JavaScript 跟各种 Java 中间件的互操作问题，这里仅就如何在一个完整的 Java 栈上以 JavaScript 作为首要编程语言来提高 Java 系统的开发效率展开讨论。

作为一门常年把持 [TIOBE](https://www.tiobe.com/tiobe-index) 排行榜冠军的静态编译类语言，Java 固然有数不胜数的优点，以至于其在企业级应用开发领域拥有不可替代的地位。但是，相信每一个 Java 开发人员都会在日常工作中忍受其种种弊端带来的烦恼，其中最令人无法忍受的正是其静态编译类的特性。静态类型在减少开发人员出错可能性的同时降低了语言的动态扩展能力；而编译执行除了能保证代码的编译期校验以外，更多的就只剩下无尽的等待了。尽管现在大多数的工具链和集成开发环境都可以做到增量编译，但是增量编译后可执行代码的热部署能力却是差强人意。尽管有一些[商业的](https://zeroturnaround.com/software/jrebel)或[开源的](http://www.hotswapagent.org)解决方案，但总归不是天生特性，使用起来很不方便。

其实针对 Java 语言缺陷的改良，社区始终也没有停止尝试。著名的成果包括 [Scala](http://www.scala-lang.org)、[Groovy](http://www.groovy-lang.org)、[Jython](http://www.jython.org)、[JRuby](http://jruby.org) 以及我们今天要讨论的 JavaScript。社区通常都有这么一个共识：Java 虚拟机是个好东西，但 Java 语言本身不一定。因此改良的重点全部集中在语言层面，通过引入具有不同特性的新语言，经过编译器或解析器翻译成 Java 字节码，再交给虚拟机来运行。

另一方面，JavaScript 近两年人气爆棚，有逐渐发展成为全栈开发语言的趋势。尽管在 Node 平台上使用 JavaScript 开发后端应用早已是习以为常的事情，但是在 Java 平台上，这件事情就不是那么显而易见了。说到在 Java 开发栈上运行 JavaScript 就不能不提大名鼎鼎的 [Rhino](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Rhino)。这是早在 [1997](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Rhino/History) 年就已经出现的 JavaScript 引擎，并完全由 Java 代码编写而成。后来被 JDK 1.6 收纳为官方默认的 JavaScript 引擎。这也就意味着现如今的 Java 虚拟机，都是一直可以支持 JavaScript 的。不过由于大环境（在 Java 平台上面跑 JavaScript 毕竟接受度不高）以及 Java 虚拟机对动态语言支持能力欠缺等问题，使得这件事情并没有流行起来。

另外一个导致 Rhino 不够流行的原因，就是对其性能的诟病。在早期的时候 Rhino 是将 JavaScript 代码直接编译为 Java Class，运行期性能非常出色，往往可以击败很多用 C 实现的基于 JIT 技术的引擎。不过生成 Java 二进制代码和装载这些生成的类是个非常重量级的操作，需要耗费大量时间，而且编译过程产生的许多无用中间类和字符串无法被虚拟机回收掉。不过在引入了解析模式以后这些问题就得到了一定的解决。一些 [性能](http://hns.github.io/2010/09/21/benchmark.html) [测试](http://hns.github.io/2010/09/29/benchmark2.html) [报告](https://www.techempower.com/benchmarks) 显示出 Rhino 的性能依旧非常良好。

后来 Java 官方也意识到虚拟机对动态语言欠缺支持以后，就从 JDK 1.7 开始引入了 [invokedynamic](http://www.javaworld.com/article/2860079/scripting-jvm-languages/invokedynamic-101.html) 的技术，可以有效提升动态语言的运行性能，而且 Oracle 也在 JDK 1.8 中自带了一个基于此技术实现的新一代的 JavaScript 引擎 [Nashorn](http://www.oracle.com/technetwork/articles/java/jf14-nashorn-2126515.html)，终于让性能不再是什么大问题了。

![JavaScript in JVM](/static/uploads/2015/javascript-in-jvm.png)

在不同版本的 JDK 上运行 JavaScript，可以有不同的解决方案（如上图）。如果是在老版本的 1.6 上面，只能使用 Rhino，以及在 Rhino 之上封装的开发框架 [Ringo](http://ringojs.org)，这个组合里面没有对 invokedynamic 的支持。当然由于向后兼容性的存在，其实这个解决方案是可以适用于后续 1.7 和 1.8 版本的。如果是在 1.7 版本上面尝试使用 JavaScript，那么可以考虑 [DynJS](http://dynjs.org) 引擎，这是另外一个独立的 JavaScript 引擎实现，支持了 invokedynamic 技术，与此配套的还有一个 [Nodyn](http://nodyn.io) 项目，是对 Node 的一个移植。不过很可惜，这套开发栈没等流行起来，就被后来的技术给超越了，因此除非你必须要坚守在 1.7 上面，否则并不建议采用。最后，也是这个话题的终极解决方案，就是升级到 JDK 1.8 并使用官方的 Nashorn 引擎，与其对应的框架可以选用 [Vert.x](http://vertx.io)。Vert.x 是由 Eclipse 贡献的，实现了良好的事件驱动和异步非阻塞机制，以及 Reactive 特性，这些都是 Ringo 所没有的。不过 Oracle 官方也有一个基于 Nashorn 引擎的 Node 实现，叫做 [avatar.js](https://avatar-js.java.net)，不过截止到发稿时，该项目已经有一段时间没有更新过了。

下面展示一些使用 JavaScript 开发 Java 应用的示例。

*<small>注：在尝试运行这些样例代码之前，请确认已经正确安装了 JDK 1.8u51 或以上版本、Vert.x 3.x 和 Maven 3.x 版本。</small>*

### 一、操作 ArrayList

```javascript
var list = new java.util.ArrayList();
list.add(1);
list.add(2);
list.add(3);
for (var i = 0; i &lt; list.size(); i++) {
  print(list.get(i));
}
```

![ArrayList](/static/uploads/2015/arraylist.gif)

### 二、Filter and Reduce

```javascript
var data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

var filtered = data.filter(function(i) {
  return i % 2 == 0;
});
print(filtered);

var sumOfFiltered = filtered.reduce(function(acc, next) {
  return acc + next;
}, 0);
print(sumOfFiltered);
```

![Filter and Reduce](/static/uploads/2015/filter-and-reduce.gif)

### 三、Web 服务

```javascript
vertx.createHttpServer()
  .requestHandler(function (req) {
    req.response()
    .putHeader("content-type", "text/plain")
    .end("Hello World!\n");
}).listen(8080, function() {
  print('listening on port 8080...');
});
```

![Web server in Vert.x](/static/uploads/2015/web-server-in-vertx.gif)

### 四、Spring、Stick 和 Hibernate (SSH)

这是一个比较复杂的例子，首先项目使用 Maven 来管理，并通过 jetty-maven-plugin 来运行。项目内部使用 Spring 作为整个集成的核心，用 Hibernate 来实现数据访问。Stick 是基于 Ringo 的一个组件，实现了类似 express 一样的功能。下面的这段代码，演示了如何通过 JavaScript 实现一个 router，并在 router 内部调用 Spring 和 Hibernate 的 API 实现数据查询，并返回 JSON 结果。此项目的完整源代码请在[这里](https://github.com/tangrui/enterprise-javascript-shenjs2015)获取。

```javascript
var {Application} = require('stick');
var {html, json} = require('ringo/jsgi/response');

var {Context} = net.tangrui.shenjs.ringossh.web.SpringAwareJsgiServlet;

exports.router = (function() {
  var app = new Application();
  app.configure('params', 'route');

  app.get('/hello', function() {
    return html("Hello World!");
  });

  app.get('/tasks', function() {
    var context = Context.getInstance();
    var emf = context.getBean('entityManagerFactory');
    var em = emf.createEntityManager();
    var query = em.createQuery('from Task');
    var tasks = query.getResultList();
    var result = []; 
    for (var i = 0; i &lt; tasks.size(); i++) {
      var task = tasks.get(i);
      var taskObj = {};
      taskObj.name = task.name;
      taskObj.description = task.description;
      result.push(taskObj);
    }
    em.close();
    return json(result);
  });

  return app;
})();
```

运行该项目前，需要首先安装并配置一个 MySQL 数据库，并运行在默认的 3306 端口，root 用户的口令为 mysecretpassword。创建一个名为 ringo-ssh 的数据库，字符集为 UTF-8。然后使用如下的 SQL 语句创建一个数据库表 T\_TASK，并插入一些数据：

```sql
CREATE TABLE `T_TASK` (
  `F_ID` varchar(255) NOT NULL,
  `F_DESC` varchar(2000) DEFAULT NULL,
  `F_NAME` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`F_ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO `T_TASK` (`F_ID`, `F_NAME`, `F_DESC`) VALUES ('1', 'ShenJS 2015', '别忘记在 7 月 11 日和 12 日。');
INSERT INTO `T_TASK` (`F_ID`, `F_NAME`, `F_DESC`) VALUES ('2', '给妈妈打电话', '周末记得给妈妈打电话。');
```

另外，该项目依赖了我们[公司](http://www.zyeeda.com)的两个开源项目，分别是 [origin](https://github.com/zyeeda/origin) 和 [cdeio-runtime](https://github.com/zyeeda/cdeio-runtime)。请从 Github 上 clone 下来以后，分别进入项目目录，运行 `mvn clean install` 来编译和安装。注意执行的顺序为先 origin 再 cdeio-runtime。最后，可以通过运行 `mvn jetty:run` 命令来启动这个示例。

*<small>注：如果是第一次运行该示例，Maven 会下载很多依赖包，这个过程根据网速快慢会长短不一，请耐心等待。</small>*

以下是系统运行后的控制台截图和 curl 的返回结果。

![Ringo SSH Maven](/static/uploads/2015/ringo-ssh-maven.png)

![Ringo SSH Curl](/static/uploads/2015/ringo-ssh-curl.png)

总之，在 Java 平台上使用 JavaScript 作为首要开发语言是完全可行的，在上面这个示例中，除了 Hibernate 必须的领域实体等是使用 Java 来编写的之外，其他的大部分逻辑都是可以用 JavaScript 完成的，而且可以跟各种 Java 中间件完美集成。有关这方面的深入介绍远不止一篇博客所能言尽，有兴趣了解更多的可以参考我公司的 ~~[CDEIO](http://www.zyeeda.com/platform.html)~~ 平台。
