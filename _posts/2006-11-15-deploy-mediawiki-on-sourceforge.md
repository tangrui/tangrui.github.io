---
ID: 31
title: >
  在 Sourceforge 项目空间上部署 MediaWiki
author: 唐睿
date: 2006-11-15 14:07:51 +0800
categories: [技术]
tags: [运维, Sourceforge, MediaWiki]
layout: post
published: true
---

在开始部署之前，有一些准备工作要做：

你需要在 Sourceforge 上面拥有自己的帐号，并加入到一个项目，而且你的帐号在该项目中需要有访问 shell 的权限。同时确保该项目已经开通 MySQL 数据库。

到 MediaWiki 的官方网站去下载 MediaWiki 1.6.8 版本。注意千万不要下载最新版本，因为 MediaWiki 从 1.7 版以后需要 php 5 的支持，而 Sourceforge 服务器上部署的仍然是 php 4，因此无法运行最新版本。

从 PuTTY 网站上下载 PuTTY.exe 和 PSFTP.exe。

为了方便，假设我在 Sourceforge 上拥有用户 usr，密码为 pwd，而且参与的项目为 sample。该项目的 MySQL 数据库具有读写权限，可以添加、删除表和索引，并可以锁定表的用户为 mysqlusr，密码 mysqlpwd。

把下载来的 mediawiki-1.6.8.tar.gz 文件上传到项目空间，具体做法如下：

1. 运行 psftp.exe
2. 执行 open shell.sourceforge.net
3. 按照提示输入你在 sourceforge 上注册的用户名，例如 usr
4. 输入用户密码，例如 pwd
5. 进入项目的 htdocs 目录，执行（以 sample 项目为例）`cd /home/groups/s/sa/sample/htdocs/`
6. 把 mediawiki-1.6.8.tar.gz 复制到 psftp.exe 所在目录
7. 运行 put mediawiki-1.6.8.tar.gz
8. 待上传完成，关闭 psftp

解压已经上传的 mediawiki-1.6.8.tar.gz 文件：

1. 运行 putty.exe
2. 在 PuTTY Configuration 界面的 Host Name 中输入 shell.sourceforge.net，确认 Protocal 选中 SSH，然后点 Open
3. 按照提示输入用户名，例如 usr
4. 然后输入用户密码，例如 pwd
5. 进入项目的 htdocs 目录，执行（以 sample 项目为例）`cd /home/groups/s/sa/sample/htdocs/`
6. 执行 tar -xvzf mediawiki-1.6.8.tar.gz
7. 将解压后的目录重命名为 wiki，执行 mv mediawiki-1.6.8 wiki

登录 phpMyAdmin，创建一个 Database，例如 wikidb。当然用 sourceforge 提供的 MySQL 服务创建 Database 时，命名都需要有一个前缀，在此忽略。

打开浏览器，登录项目 Wiki，例如 `http://sample.sourceforge.net/wiki`。此时由于 MediaWiki 尚未配置，信息提示让你进入配置页面，点击进入配置页面，发现出错了。大意是说你的 config 目录没有写权限，让你在 MediaWiki 的安装目录执行 `chmod a+w config` 命令更改权限。但是，尽管你按照提示完成了设置，错误依旧。

本文的[第二部分](/2006/deploy-mediawiki-on-sourceforge-2.html)在这里。
