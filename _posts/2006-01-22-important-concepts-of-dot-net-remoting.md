---
ID: 21
title: >
  .net Remoting
  中的几个重要概念和实现方法
author: 唐睿
date: 2006-01-22 22:46:32 +0800
categories: [技术]
tags: [.net, .net Remoting]
excerpt: >
  本文主要介绍了 .net Remoting 中一些重要概念，及使用方法。
layout: post
published: true
---

### 应用程序域 (Application domain)

一般的 Windows 应用程序都是以进程的方式在操作系统中运行，操作系统负责分配并管理程序所请求的资源。但是对于使用 .net 编写的托管程序而言，有一个特殊的进程被称为公共语言运行时 (CLR) ，该进程负责加载托管程序并运行。对于在 CLR 中运行的每一个托管程序而言，都有一个被称之为应用程序域的边界，每个托管程序都在自己的应用程序域中安全的运行。不同的应用程序域之间互不干扰。

### 上下文 (Application context)

将应用程序域进一步细分，就形成了上下文，上下文确保一套常用约束和使用语法负责管理其中的所有对象访问。每个应用程序域至少包含一个上下文，称为默认上 下文 (Default context) 。除非某个对象明确的要求一个专门的上下文，否则运行时 (Common language runtime) 将在默认上下文中创建那个对象。

### .net Remoting 边界

对于应用程序而言应用程序域的边界是 .net Remoting 的边界。对应用程序域而言上下文的边界是 .net Remoting 边界，一个普通的对象无法穿越 .net Remoting 边界。

### 不可远程化对象和可远程化对象

#### 一、不可远程化对象 (Non remotable object)

在默认情况下，没有经过任何特殊处理的对象都是不可远程化对象。不可远程化对象无法以任何方式（拷贝或引用）被跨应用程序域的对象所访问，当企图把对象引用传递到其他的应用程序域中时，会有异常产生。

#### 二、可远程化对象 (emotable object)

如果一个类的实例可以穿越 .net Remoting       边界并可以在边界外被访问，则该对象就是可远程化的。在应用程序域外可远程化对象被访问的方式有两种：可以通过对象的完整副本来访问，还可以通过对象的引用（在 .net Remoting 中是以代理的模式来实现的）来访问。

### .net Remoting 的服务器和客户端

.net Remoting 的服务器和客户端与以往的概念没有什么差别，但是在这里要强调的是 .net Remoting 的服器器和客户端显然是处于不同的应用程序域中，但是并不一定处于不同的计算机上。

### 按值列集 (Marshal by value) 和按引用列集 (Marshal by reference)

一、当运行时可以获得一个对象的完整副本的时候，则该对象就可以以所谓按值列集的方式被传递到其他的应用程序域中。实现按值列集的途径就是在声明类的时候添加 Serializable 特性 (Attribute) 或者实现 ISerializable 接口。例如：

```csharp
[Serializable]
class Foo {
    ...
}
```

因此能够按值列集的对象也被称为可序列化的对象。客户端将获得该对象的完整副本。

二、当一个类直接或间接地从 MarshalByRefObject 类继承的时候，运行时就可以在客户端创建一个该对象的代理。例如：

```csharp
class Foo : MarshalByRefObject {
    ...
}
```

### 上下文邦定 (Context bound)

当一个类型的实例只停留在具体的上下文内的时候，该类型就是上下文邦定的类型，在这个域内的其它上下文中的对象不能直接访问该对象。上下文邦定类型通过继承 System.ContextBoundObject 类来实现。

### 代理 (Proxy)

刚才提到，当对象以按引用列集的方式在应用程序域中传递的时候，运行时将在接收方创建一个该对象的代理。该代理可以想象为（通常也是这样实现的）一个封装了该对象所有或部分成员的接口，负责将接收方（客户端）的调用信息发送给发送方（服务器）。

### 通道 (Channel)

通道用于在跨应用程序域的远程对象间传递消息 (Message) 。服务器将选择侦听请求的通道，而客户端则选择希望与服务器进行通信的通道。运行时提供了两种内置的通道 Http 通道和 Tcp 通道。

```csharp
using System.Runtime.Remoting; // .net Remoting 命名空间
using System.Runtime.Remoting.Channels; // .net Remoting 通道命名空间
using System.Runtime.Remoting.Channels.Http; // .net Remoting Http 通道命名空间
using System.Runtime.Remoting.Channels.Tcp; // .net Remoting Tcp 通道命名空间
. . .
// ChannelServices 是一个工具类，里面封装了大量的与通道相关的静态方法
ChannelServices.RegisterChannel ( new HttpChannel() ); // 注册 Http 通道并使用默认端口
ChannelServices.RegisterChannel ( new TcpChannel(4242) ); // 注册 Tcp 通道并使用4242端口
```

### 激活 (Activation)

#### 一、服务器端激活 (Server Activation)

服务器端激活方式是指对象的生存周期（何时生成与何时被垃圾回收）由服务器来决定。服务器端激活有两种方式：单件 (Singleton) 和单调用 (SingleCall) 。

#### 二、客户端激活 (Client Activation)

客户端激活方式是指对象的生存周期由客户端来决定。客户端激活只有一种方式，称为 CAO (Client Activation Object) 。每一个客户端激活创建一个对象，该对象存在于如下两个事件之一到来之前：客户端失掉对对象的引用，对象租借过期。客户端激活模式可以存储每一个客户端的状态，并接受构造函数参数。可以使用如下的代码，在服务器端设置客户端激活的远程对象：

```csharp
RemotingConfiguration.RegisterActivatedServiceType( typeof( SomeMBRType ) );
```

然后在客户端设置如下：

```csharp
RemotingConfiguration.RegisterActivatedClientType( typeof( SomeMBRType ), "http://SomeURL" );
```

此处的 SomeURL 是指服务器端的地址或计算机名。

### 众所周知 (Well-known object) 的对象

服务器端激活的类型我们就称之为众所周知的。在使用众所周知的对象时，服务器要进行如下的设置：对象的类型，何时以及如何实例化对象，和一个客户端用来与该类型联系的名称（或叫终端 end point ）。而同时客户端要设置连接到哪一个服务器，并在终端上获得众所周知的类型。使用 RemotingConfiguration.RegisterWellKnownServiceType 这个方法来注册一个众所周知的类型，该方法需要提供三个参数：待注册的类型，传达给客户端的终端名称和激活模式。例如：

```csharp
using System.Runtime.Remoting; // .net Remoting 名名空间
. . .
WellKnownServiceTypeEntry WKSTE =  new WellKnownServiceTypeEntry(
                     typeof( MyNameSpace.SomeMBRType ),
                     "SomeURI",
                     WellKnownObjectMode.SingleCall );
RemotingConfiguration.RegisterWellKnownServiceType(WKSTE);
```

其中先创建一个 WellKnownServiceTypeEntry 类型的对象，用于在 RegisterWellKnownServiceType 方法中传递参数。
typeof( MyNameSpace.SomeMBRType ) 这个参数待注册的远程对象的类型， SomeMBRType 是 MyNameSpace 命名空间下的一个类。
SomeURI 便是传达给客户端的终端名称。（它的用途在后面会介绍。）
WellKnownObjectMode.SingleCall 就是将该类型注册为单调用模式。

### 单件和单调用

一、单调用：服务器为每个客户端的方法调用生成一个单调用对象，每个对象服务且仅      服务一个请求。只有当方法调用到达的时候才按需求创建对象，并且对象的生存期直至调用结束。单调用模式适用于无需保存状态的应用程序，是解决负载平衡的最好选择。可以通过如下的方法在服务器端将可远程化类型配置为单调用模式：

```csharp
// RemotingConfiguration 是一个工具类
RemotingConfiguration.RegisterWellKnownServiceType(  
   typeof( SomeMBRType ),   // 从 MasharlByRef 继承来的数据的类型信息
   "SomeURI",                     // 发布该服务器端激活对象时使用的 URI
   WellKnownObjectMode.SingleCall ); // 设定为单调用模式

// 然后用如下的方法在客户端再配置一次：
RemotingConfiguration.RegisterWellKnownClientType(
   typeof( SomeMBRType ),               // 要从服务器端得到的远程类型
   "http://SomeWellKnownURL/SomeURI" ); // 众所周知的对象发布的 URL
```

二、单件：服务器在任何情况下都只创建该类型的一个实例，客户端的所有请求都由这      一个实例来处理，且该实例的生存期与服务器的生存期相同。单件模式适用于由状态的应用 程序，但是这种模式的对象只能保存非客户端的状态。同上面的代码，只要将 WellKnownObjectMode 设置为 Singleton 就可以了。

### 众所周知对象的 URL

服务器端激活的对象是在 URL 上发布的，该 URL 是被客户端众所周知的。众所周知对象的 URL 看上去如下：
`ProtocolScheme://ComputerName:Port/ApplicationName/ObjectUri`

其中 ProtocolScheme 代表 .net Remoting 通道所使用的协议，例如 Http 或 Tcp 等。ComputerName 代表 .net Remoting 服务器的名称或地址。Port 是注册通道时使用的端口。ApplicationName 就是服务器端应用程序的名称。当使用 IIS 作为服务器端宿主的时候， ApplicationName 就变成了 IIS 的虚拟目录。
ObjectUri 就是在使用 RegisterWellKnownServiceType 时注册的 SomeURI 。 ObjectUri 必须以 .rem 或 .soap 结束，以区别是使用 Tcp 还是 Http 协议。

### 基于租借的生存期

客户端激活对象的生存期由与对象相关的租借来控制，租借有一个租借期， .net Remoting 基础设施在租借过期的时候放弃对象的引用，每一次方法调用可以更新租借，客户端可以使用代理更新租借，发起者也可以更新租期。

### 客户端激活对象的 URL

客户端激活对象不需要为每一个对象配备一个单独的 URL ，客户端激活对象的 URL 看上去如下：
`ProtocolScheme://ComputerName:Port/ApplicationName`
