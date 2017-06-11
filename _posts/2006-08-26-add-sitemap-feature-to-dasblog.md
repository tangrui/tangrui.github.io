---
ID: 39
title: >
  给 dasBlog 添加支持 SiteMap 的功能
author: 唐睿
date: 2006-08-26 23:16:21 +0800
categories: [技术]
tags: [dasBlog]
layout: post
published: true
---

今天享受了一下 Google 的 WebMasters 服务，再加上 Google Analytics，真有 WebMater 的感觉。

在这个服务里面，为了网络爬虫能够更好地抓取页面，Google 需要你提供 SiteMap，一个 XML 格式文件。试了 Google 推荐的几个程序和服务，感觉都不是很好，鉴于 dasBlog 是开源的，简单研究了一下，为其添加了生成 SiteMap 的功能。

首先，下载 [net.tangrui.DasBlog.Web.Services.dll](/static/uploads/2006/dasblog-sitemap-dll.zip) 文件，将其放到 dasBlog 的 bin 目录下。

然后，修改 `web.config` 文件，在 `<httphandlers>` 标签下，添加这样的节点：

```xml
<add verb="*"
  path="sitemap.ashx"
  type="net.tangrui.DasBlog.Web.Services.SiteMapHandler, net.tangrui.DasBlog.Web.Services"></add>
```

最后，访问 `http://yourwebsite/sitemap.ashx` 就可以了。

如果需要，可以到[这里](/static/uploads/2006/dasblog-sitemap-src.zip)下载源代码。
