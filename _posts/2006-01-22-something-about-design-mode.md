---
ID: 26
title: 关于 Design Mode 的一点认识
author: 唐睿
date: 2006-01-22 22:56:22 +0800
categories: [技术]
tags: [编程, GUI, .net, 设计器, Visual Studio]
layout: post
published: true
---

今天绝大部分时间用于解决了一个很小的问题，但是从中却了解到 Visual Studio.net 中 Design Mode 的一些实现方式。

从头开始，先来说说我遇到的这个问题：如何判但一个实例所对应的类是否实现了某一个接口？这其实也算不上什么问题，我起初也是知道的，就是和判断某个对象是否是某个类的实例一样，使用 is 关键字就可以了。但问题偏偏没有那么简单。

我的目的是想要实现这样一种功能，一个用户控件 (User Control) 被放到一个容器（之所以说是容器，因为它不一定只是 Form, 也有可能是 Tab page, Group box 等等）中，那么我需要这个用户控件去检查这个容器是否实现了我规定的一个接口，如果是才允许在这个容器中创建自己。

于是，问题接踵而来。首先，如何得到这个用户控件所在的容器，通过搜索 MSDN 得到两个属性，一个是 UserControl.Container, 另一个则是 UserControl.Parent. 前一个似乎很像，不过它返回的是一个 IContainer 接口的实例，无法直接使用；而后者相对较好，返回一个 Control （该类是所有 Windows 控件，包括 Form 的基类）类的实例。很自然，我就开始在这个属性上下文章。

假如我要指定实现的接口是 IMyInterface. 假如我使用 Form 作为该控件的容器，那么在 IDE 自动给我在默认窗体后面继承一个 System.Windows.Forms.Form 之外，还需要继承这个接口，这部分很简单但是当我使用 is 关键字进行如下判断时就出现了问题 this.Parent is IMyInterface （其中的 this 就是那个用户控件的实例），这个判断永远也通不过。后来我试图将其做一个强制类型转换，却得到了指定转换非法 (Spicified cast is invalid.) 的异常。原因就在于 this.Parent 返回一个 Control 类型，而该类型在 .net 类库中显然没有实现我的那个接口，自然转换非法是正常的。但如何解决呢？

马上就能想到的自然是自己重写一个 Control 类，让该类实现我的接口。其实完全可以重写一个 Form 类，来实现接口。然后再使得你创建的 Form 从你这个扩展了的 Form 去继承，再作如上的转换就不会有问题了。我也就是这样做的，不过稍微画蛇添足了一点：
public abstract class MyCustomForm : System.Windows.Forms.Form,  IMyInterface {}

就是加了一个 abstract 关键字，以阻止用户从该类生成实例。这是就出现了下一个问题，从这个 MyCustomForm 继承的窗体无法在设计器中展现了！这个问题起初没有弄明白是怎么回事，于是我就先去掉 abstract 关键字，使得一切恢复正常。随后我在这个类中添加了一个返回 bool 类型的虚属性，并默认实现返回 false, 就像这样：

```csharp
public virtual bool MyProperty {
  get {
    return false;
  }
}
```

然后我在我真正的 Form 中 （起名为 MainForm）覆盖这个属性，并返回 true. 之后我在用户控件的 Load 事件方法中调用这个属性，然而奇怪的是当向窗体中添加这个用户控件的时候（注意此时仍处于 Design Mode），该属性一直都为 false, 既基类中的实现，也就是说似乎属性没有被覆盖。这时我就想到了一个关于何时使用基类中的方法，何时使用覆盖了的方法的问题。因为可以想得到，当你把 MainForm 转到 Control 之后（通过调用 UserControl.Parent 实现）原有 MainForm 中多余的成员是否会被丢掉，当再转换成 IMyInterface 的实例时是否将无法找到覆盖过的方法，以至于直接去调用积类中的方法。似乎这是理所当然的，但是我随后写了一个类似的小程序试验，结果发现并不是这样，原因也很简单，这些转换只是在引用上发生的，已有的实例中的内容并不会被销毁，那么出现这个问题就太诡异了。

仔细想呀想呀，联想到上面出现的那个 Design Mode 的问题，一切就都能揭示了。 简单的就一句话：Design Mode 在你操作时需要生成一个实例，然而该实例不是运行时的实例。具体来说，你在 Design Mode 中设计一个窗体，那么你会看见 IDE 自动为你创建了一个名叫 Form1 的类，并且继承自 System.Windows.Forms.Form, 同时生成一个可视化的窗体界面，这个界面是一个窗体的实例。但是这个实例不是 Form1 的实例，而是其基类 Form 的实例，原因就是，IDE 是在帮你设计这个 Form1, 显然此时这个 Form1 还不存在，存在的只是一堆代码而已。这样你就可以解释一切了，当我的基类被 abstract 的时候无法被实例化以后，自然设计器就没有办法为我创建这个类的实例使得我能进行可视化操作，但是你仍然可以将程序无误的跑起来。因为所谓的那个 Form1 并不是 abstract 的。也正因为如此，当我的 UserControl 在 Design Mode 时去调用 Interface 中的方法时，自然得到的就是基类中虚属性的实现，而在运行时，得到的就是覆盖了的属性的返回值，经过试验，答案却是是这样的。

另外，你可使用 UserControl.DesignMode 来判断其是处于设计时还是运行时。
