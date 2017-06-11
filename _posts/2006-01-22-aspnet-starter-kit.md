---
ID: 18
title: ASP.NET Starter Kit
author: 唐睿
date: 2006-01-22 22:40:00 +0800
categories: [技术]
tags: [编程, 框架, .net, ASP.net]
layout: post
published: true
---

微软的东西使得程序员的工作变得越来越轻松，当然同时也使得靠吃程序员这碗饭过活的人日子越来越难过了。最近我在微软 MSDN 中文网站上又发现了新东西（当然对于经常关注微软动态，且经常浏览微软几大英文门户的人们来说，这也许算是老掉牙的东西了），一套开放源代码的、完全免费的且说不好具有什么种类版权的门户网站入门套件： ASP.NET Starter Kit 。据说如果你有足够的实力，经过重写的源代码甚至是可以直接拿出来卖钱的。（怎么越看越像 BSD 版权呢？）有这么好的东西，怎么能不尝试一下呢？以下就是我对 ASP.NET Starter Kit 体验的随笔。

# 一、什么是 ASP.NET Starter Kit

关于这个问题我是无法给出什么权威性的解释的，但是现如今像这样的东西满网络里到处都是，大家肯定也不会陌生，只不过她系出名门，由微软的牛人们写出来的，地位自然显赫得多了。不要简单的望文生义，以为这套东西是为初学者准备的，虽然我还没有看到那里，但据说这个套件里的各个部分都使用了看起来令人兴奋的设计模式，绝对是学习和使用的不二之选。

本套件包含五个部分， Portal Starter Kit （门户网站入门套件）, Commerce Starter Kit （电子商务入门套件）, Reports Starter Kit （报表产生入门套件）, TimeTracker Starter Kit （项目追踪入门套件） 和 Community Starter Kit （论坛入门套件）。每一个部分都可以直接拿过来部署，也可以经过自己的定制，重新发布。据说已经有几个网站开始采用该套件中的 Portal 部分部署自己的门户网站，相应的内容大家可以到微软的网站上找到。

# 二、如何获得 ASP.NET Starter Kit

当然是在微软的网站上，英文版的可以到以下网址找到： http://www.asp.net/default.aspx?tabindex=9&tabid=47 , 而中文版的可以到 MSDN 中文网站上去下载。

# 三、ASP.NET Starter Kit 的版权声明

随着 ASP.NET Starter Kit 的分发，微软也同时也分发了该套件的版权声明，在 EULA.rtf 文档中，如果真的有人想要用它做些什么东西的话，最好还是好好看一看。我粗略的看了一下，怎么看怎么像 BSD 版权，包括什么必须保留微软版权、不允许加入其它的有版权产品，不允许使用微软的标志等等。

# 四、ASP.NET Starter Kit 主要部分介绍

由于本人也是刚刚开始接触这个东西，加之目前有一个项目恰好可以用它来改，所以我首先选择了门户部分作为学习的突破口。

## 4.1 ASP.NET Portal

### 4.1.1 安装

据文档中说，该套件应该包含一个 .exe 或 .msi 的安装程序，不幸的是我在相应的 Setup 目录下没有找到这样的程序（很可能是我找到的这个发布包有问题），所以只好用功能强大的，但是相对难使的命令行工具。在 Setup 目录下可以找到 Cmd Installer 目录，进到里面就会看到 PRTL.exe 的命令行安装程序、两个写好的用于本地和远程安装的示例批处理文件 LocalFullInstall.bat 和 RemoteFullInstall.bat 、用于安装数据库的数据库脚本和 PRTL.exe 安装程序的用户文档。根据该用户文档，PRTL命令的参数如下：

#### 4.1.1.1 INSTALL\_OPTION

用于设定安装模式，有四个选项：

**FULL** - 既安装 Web 又安装数据库。启用此选项需要提供所有的必选参数。
DB	仅安装指定的数据库实例。启用此选项需要提供 DB\_FILES\_DIR, DB\_SERVER, DB\_NAME, DB\_OWNER\_LOGIN, DB\_OWNER\_PWD, SQL\_ADMIN\_USER, SQL\_ADMIN\_PWD参数。

**WEB** -	仅安装 Web 组件。启用此选项需要提供 WEB\_FILES\_DIR, TARGET\_DIR, IIS\_SERVER, WEB\_SITE, VDIR, DB\_CONNECTION, DB\_NAME, DB\_SERVER, DB\_OWNER\_LOGIN, DB\_OWNER\_PWD 参数。

**COPY\_ONLY** - 仅将 WEB\_FILES\_DIR 目录下的所有内容拷贝到 IIS\_SERVER 机器上的 TARGET\_DIR 目录下。启用此选项需要提供 IIS\_SERVER, WEB\_FILES\_DIR, TARGET\_DIR 参数。

#### 4.1.1.2 WEB\_FILES\_DIR

待安装的 Web 页面所在的相对或绝对路径。一般在你解开的压缩包里会有一个名为 PortalCSVS （如果是 VB 工程的话似乎就应该叫 PortalVBVS ）的目录，此下的文件就是所谓的待安装的 Web 页面。

#### 4.1.1.3 DB\_FILES\_DIR

数据库脚本所在的相对或绝对路径。在 PRTL.exe 文件所在的目录里面有一个名为 SQL Scripts 的目录，它就是数据库脚本所在的目录。其中有四个 SQL 脚本文件。

#### 4.1.1.4 TARGET\_DIR

WEB\_FILES\_DIR 目录下的内容将要被拷贝到的目标路径（相对或绝对均可）。在该目标路径下 PRTL 会创建一个子目录。

#### 4.1.1.5 IIS\_SERVER

安装有 IIS 的机器的计算机名。同时该计算机也是 TARGET\_DIR 所在的计算机。

#### 4.1.1.6 WEB\_SITE

一个已经存在的 Web 站点， VDIR 参数指定的虚拟目录将会在该站点中创建。此参数应该提供站点属性中描述栏中的内容。例如默认网站。

#### 4.1.1.7 VDIR

将要安装的门户网站所在的虚拟目录的名字。建议该名称最好叫做 PortalCSVS （即与 分发包中 Web 页面内容所在的那个目录名相同，因为当用 VS.net 打开工程的时候，会默认到名称为 PortalCSVS 的虚拟目录下寻找站点）。

#### 4.1.1.8 DB\_CONNECTION

门户应用程序连接数据库的方式。只能使用SQL选项， Windows 集成认证是不支持的。

#### 4.1.1.9 DB\_SERVER

安装有 SQL Server 数据库的机器的计算机名。

#### 4.1.1.10 DB\_NAME

SQL Server 中将要安装所有数据库对象的数据库名称。（不是数据库服务器实例的名称。）

#### 4.1.1.11 DB\_OWNER\_LOGIN

将要具有该数据库 Owner 角色的登陆名。如果该登陆名不存在，则会被创建。

#### 4.1.1.12 DB\_OWNER\_PWD

登陆名的密码。

#### 4.1.1.13 SQL\_ADMIN\_USER

具有可以创建数据库实例和用户权限的 SQL 用户名。

#### 4.1.1.14 SQL\_ADMIN\_PWD

该用户的密码。

#### 4.1.1.15 LOG\_FILE (Optional)

安装日志的文件名。可以提供希望存储该日志的完整路径。如果没有提供该参数，日志文件将会被创建到同PRTL.exe相同的目录下。

例如在我的机器上，安装命令如下：

```bat
PRTL.exe INSTALL_OPTION=FULL WEB_FILES_DIR="..\..\PortalCSVS" DB_FILES_DIR="SQL Scripts" TARGET_DIR="C:\inetput\wwwroot\portal" IIS_SERVER="localhost" WEB_SITE="默认网站" VDIR="PortalCSVS" DB_CONNECTION=SQL DB_SERVER="localhost" DB_NAME="Portal" DB_OWNER_LOGIN="PortalAdmin" DB_OWNER_PWD="123" SQL_ADMIN_USER="SA" SQL_ADMIN_PWD="sa" LOG_FILE="installLog.txt"
```

生成的安装日志内容如下：

```
[2004-07-10 15:03:46] Setup Initialized …
[2004-07-10 15:03:46] Start Installing Web Components …
[2004-07-10 15:03:46] Copying web files from H:\Documents and Settings\MyUserProfile\My Documents\Visual Studio Projects\ASP.NET Portal (CSVS)\PortalCSVS to C:\inetput\wwwroot\portal\PortalCSVS
[2004-07-10 15:03:47] Updating web.config…
[2004-07-10 15:03:47] Updating EditHtml.aspx…
[2004-07-10 15:03:47] Creating Virtual Directory: PortalCSVS, on PASTHERO
[2004-07-10 15:03:48] Start Installing Database: localhost.Portal …
[2004-07-10 15:03:48] Executing SQL Script: 8b21cd7e-006f-4f09-8016-7a26f8dfd412CreateDB.sql
[2004-07-10 15:03:50] Executing SQL Script: f241dac2-1e7e-4e70-80ab-49e5ad575e98CreateDBObjects.sql
[2004-07-10 15:03:52] Executing SQL Script: c51bb364-0d31-4c67-9813-747f3ad813cfGrantPermission.sql
[2004-07-10 15:03:52] Executing SQL Script: 2f166c1a-ef9a-4422-841e-63026f5f13f2LoadData.sql
[2004-07-10 15:03:54] Setup Completed!
```
