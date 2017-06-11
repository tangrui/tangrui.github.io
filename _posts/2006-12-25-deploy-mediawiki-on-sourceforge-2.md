---
ID: 92
title: >
  在 Sourceforge 项目空间上部署 MediaWiki（二）
author: 唐睿
date: 2006-12-25 14:43:30 +0800
categories: [技术]
tags: [运维, Sourceforge, MediaWiki]
layout: post
published: true
---

《在 Sourceforge 项目空间上部署 MediaWiki》的[第一部分](/2006/deploy-mediawiki-on-sourceforge.html)在这里。

造成 MediaWiki 不能向 config 目录写文件的原因，估计是 Sourceforge 服务器没有开放这样的权限。幸亏，MediaWiki 提供了一个变通的办法。找到下面的这些代码，把他们注释掉：

```php
if (!is_writable(";")) {
  dieout("<h2>Can't write config file, aborting</h2>

  <p>In order to configure the wiki you have to make the
  <tt>config</tt> subdirectory writable by the web server. Once configuration is done you'll move the created
  <tt>LocalSettings.php</tt> to the parent directory, and for added safety you can then remove the
  <tt>config</tt> subdirectory entirely.</p>

  <p>To make the directory writable on a Unix/Linux system:</p>

  <pre>
  cd <i>/path/to/wiki</i>;
  chmod a+w config
  </pre>");
}
```

这样一来，当目录没有写权限的时候，操作不会被取消，而是生成的配置文件会以文本的形式被打印在页面上。把这些代码拷贝、粘贴到一个名为 `LocalSettings.php` 的文件，然后上传到 MediaWiki 的根目录，在我们这个例子中也就是 `/home/groups/s/sa/sample/htdocs/wiki/` 就可以了。

最后还需要解决一下 session 的问题。由于 Sourceforge 使用了服务器集群，所以把 session 保存在内存中是有问题的，所以需要用文件来存储 session：

在 `/tmp/persistent/` 下面创建一个用来保存 session 的目录，例如 `/tmp/persistent/sample/sessions`。这里的 sample 就是你的项目的 unix name。因为这个名称是唯一的，所以不用担心会跟别人的重复。但注意千万不要把它建在你的项目目录下，因为他们是不能被 web 服务器写入的。

确保这个目录是可写的，也就是执行一下 `chmod a+w /tmp/persistent/sample/sessions`。sample 仍然是项目的 unix name。
告诉 MediaWiki 使用这个目录来存放 session，方法是在 `LocalSettings.php` 文件的开始处添加 `session_save_path("/tmp/persistent/sample/sessions/");` 注意不要丢掉 sessions 后面的那个 "/"。如果你这样做之后，在编辑或查看页面的时候退出，碰到空白页面，请把 `ini_set( "include_path", ".:$IP:$IP/includes:$IP/languages" );` 添加在上一步的 `session_save_path` 之前。

这样，你的 Wiki 就基本上可以在 Sourceforge 的项目空间上运行了，不过还有很多问题这里没有涉及到，比如：如何配置邮件系统、如何修改网站的 logo、或者是一些碰到的其他的问题。更多内容请详见 Running MediaWiki on Sourceforge.net。（中国大陆地区访问可能会有障碍。）
