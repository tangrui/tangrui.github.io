---
ID: 13
title: >
  Quake III Arena 源代码在 Win32 下的编译
author: 唐睿
date: 2006-01-22 22:26:00 +0800
categories: [技术]
tags: [开源, Quake, Id Software, John Carmack]
layout: post
published: true
---

按照 John Carmack 的承诺 Quake III Arena 也终于开源了！在 Id Software 的网站上下就能购下载到。

不过似乎网上更流行的版本是从这里能够得到的一个名为 quake3-1.32b-.source.zip 的 5.45M 的压缩包，里面括了相对较全的内容，有 Q3A 的全部源码，lcc, q3asm, q3map 和 q3radiant 等工具的源代码。最主要的是它还包含了 vs.net 2003 的工程文件，使得编译变简单了许多。

我最早就是用这个版本编译通过的，不过后来发现在官方网站上放出的版本和这个不太一样，本想想继续尝试编译后者，并且根据其结果来写这篇网志的，不过遗憾的是，按照说明上的指导，编译不能成功。会出现找不到文件的错误，估计是 Id 官方的编译器本身配置有些问题，我没看源码也没有深入研究，希望有兴趣的朋友能告诉我原因和解决办法。

言归正传，其实这个版本里面的信息还是足够丰富的，很容易编译，只在个别地方有些小问题。不过我还要唠叨一下，我不是一个 Quake 玩家，所以对其中的很多术语不是很清楚，也不了解整个游戏的结构，只是出于好奇尝试了一下，有问题的话还请大家多多拍砖！

### 代码结构

从说明文件来看，这份代码主要包含了这些部分：

* `code/` - Quake III Arena source code (renderer, game code, OS layer etc.)
* `code/bspc` - bot routes compiler source code
* `code/game` - governs the game, runs on the server side
* `code/ui` - handles the ui on the client side
* `lcc/` - the retargetable C compiler (produces assembly to be turned into qvm bytecode by q3asm)
* `q3asm/` - assembly to qvm bytecode compiler
* `q3map/` - map compiler ( .map -> .bsp ) – this is the version that comes with Q3Radiant 200f
* `q3radiant/` - Q3Radiant map editor build 200f (common/ and libs/ are support dirs for radiant)

### 编译 Quake III Arena

我指的是编译 Quake III 唯一的那个可执行文件。

在 code 目录下面能够找到 quake3.sln 这个 vs.net 2003 的解决方案文件，你尽可以打开它看看到底都有些什么东西，不过我在这里只谈编译，所以就拿它当个黑盒，直接编译了。

编译 Quake 需要 DirectX SDK 的支持，因为看游戏目录中包含的是 DirectX 7.0 所以我估计 7.0 的 SDK 就可以了，不过我仍然用了最新的 DirectX 9.0c 2005 年六月的那个版本，可以到微软的网站上去下载。不过我一直不理解的是，据说 Quake 是一个纯 OpenGL 的游戏，怎么它的编译要用到 DirectX 呢？无论如何安装好 DirectX SDK 之后，就可以编译了，不过在此之前要确保你的机器里已经安装了 vs.net 2003，并且 devenv.exe 这个文件在你的 path 环境变量中。（devenv.exe 就是 vs.net 集成开发环境的可执行文件，一般的安装目录在 %ProgramFiles%\Microsoft Visual Studio .NET 2003\Common7\IDE 里面。）打开命令窗口，进入 code 目录，运行命令：

```bat
devenv quake3.sln /build release
```

不出意外的话，编译会顺利完成，其中会有几个警告，不过不影响结果。之后你会在 release 目录中找到一大堆编译好的二进制文件，不过有用的似乎只有那个 quake3.exe。

### 安装 Quake III Arena

只是因为在 code 目录下面有一个 installrelease.bat 文件，所以这一步就姑且叫做安装吧。在进行这一步之前，还是要准备一下环境变量，将 code\win32\mod-sdk-setup\bin 这个路径加入到 path 中，因为需要用到 lcc 和 q3asm 两个编译工具。然后打开 intallrelease.bat 这个文件，注释掉最后一行和倒数第三行，并且将倒数第二行中的 "_ta" 和 "g:" 去掉，使得最后三行变为这样：

```bat
rem call closefiles
copy release\quake3.exe \quake3\quake3.exe
rem call installvms
```

为什么要这样做，仔细看看也就明白了。首先是根本就没有 closefiles.bat 这个文件，然后将 release 目录下的 quake3.exe 拷贝到根目录下的 quake3 子目录中。最后的 installvms.bat 只不过是做了一个移动操作，没有什么用处，因此只样改过的 intallrelease.bat 文件就可以很好的工作了。最后还要确保在根目录中没不存在 quake3 这样的目录。

好了，这个时候，只要简单的执行一下这个文件，待结束后看看根目录下是否多了一个 quake3 的目录，里面包含了 baseq3 和 missionpack 两个子目录，具体的目录结果如下：

```
Quake3
| \ baseq3 \ vm \ cgame.map
| \ cgame.qvm
| \ qagame.map
| \ qagame.qvm
| \ ui.map
| \ ui.qvm
|
| \ missionpack \ vm \ qagame.map
| \ qagame.qvm
| \ cgame.map
| \ cgame.qvm
| \ ui.map
| \ ui.qvm
```

仔细的看一下输出的日志，就可以明白为什么会产生这些文件了。

### 运行 Quake III Arena

想必你已经能够猜到，只要运行 code 目录下的那个 runrelease.bat 就可以了。不过有两点要注意，这样运行 quake3.exe 需要两个参数，一个是 fs_basepath 是要指向 quake3 这个目录，而另一个 fs\_cdpath 则需要指向 Quake III Arena 的原始光盘，因为需要从里面读取资源文件。保证这两个参数都没有问题了，你应该就能够看到熟悉的 Quake III Arena 的画面了，否则会出现 Couldn't load default.cfg 的错误。

### 其他问题

如果你在运行编译好的结果时出现了如下的问题 User interfaces is version 3, expected version 6，请到 Id 的官方网站去下载最新的 Point Release，安装之后就可以正常运行了。

怎么样，还是足够简单的，至少比我想象的要简单，还是那句话，这只是一个黑盒编译，希望熟悉 Quake 游戏的人们能够深入的解释一下得到的结果，算是抛砖引玉了……
