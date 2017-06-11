---
ID: 345
title: 提升 Jetty 服务器的启动速度
author: 唐睿
date: 2013-05-03 18:46:12 +0800
categories: [技术]
tags: [编程, Web, Jetty, Servlet, Java EE, 性能]
excerpt: >
  通过 metadata-complete 配置提升 Jetty 服务器的启动速度。
layout: post
published: true
---

最近在使用 Jetty 调试一个系统的时候发现，控制台会长时间停在某个地方，致使系统无法正常启动。经过调整日志的输出级别后发现，Jetty 有一个 AnnotationParser 的类，会去扫描 classpath 中所有的类文件，而碰巧这个项目有一个 20M 大小的 jar 包，就导致这个扫描过程异常缓慢，使得 Jetty 无法在短时间内里面完成启动。

要说明 AnnotationParser 类的作用，需要先从 Servlet 2.5 的规范说起。2.5 版的 Servlet 规范最重要的是引入了对 Java EE Annoation 的支持，也就是可以用注解来声明 Servlet、Filter 等对象。这件事前以前是在 web.xml 文件中完成的，现在可以把 web.xml 中的定义全部移植到 Java 类里面，而不引入任何配置文件。这就导致一个问题：作为 Servlet 容器的 Jetty 怎么能够知道哪些类使用了这些注解？很显然，并没有非常有效的办法，因此 Jetty 才实现了一个 AnnotationParser 的类型，通过遍历的方法来寻找所有使用了注解的类型。

如果我们开发的程序没有使用注解，这个过程就是纯粹在浪费时间，Servlet 规范也提供了一种办法禁用此种类型的扫描，从而提升启动速度。方法就是在 web.xml 文件中增加 metadata-complete 配置。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         metadata-complete="true"
         version="3.0">
```

**<small>注：以上配置中使用的 version 属性一定要大于 2.5，才能支持这个配置。</small>**

参考资料：

* [Slow WAR startups due to annotation scaning (affects Jetty 8)](https://issues.apache.org/jira/browse/SOLR-3309)
* [Speeding up Slow Jetty 8 Startups](http://mostlywheat.wordpress.com/2012/03/10/speeding-up-slow-jetty-8-startups)
* [New Features in Servlets 2.5](http://www.javabeat.net/articles/print.php?article_id=100)
