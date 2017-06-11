---
ID: 190
title: >
  如何去掉 OSX
  系统中打开方式里的重复项
author: 唐睿
date: 2013-05-04 23:43:13 +0800
categories: [技术]
tags: [操作系统, Machintosh, Mac, OSX]
excerpt: >
  通过一个命令去除 OSX 系统打开方式里面的重复项。
layout: post
published: true
---

在 Terminal 里面执行如下命令：

```bash
/System/Library/Frameworks/CoreServices.framework/Frameworks/LaunchServices.framework/Support/lsregister -kill -r -domain local -domain system -domain user
```

之后重启 Finder 就可以了。
