---
ID: 29
title: Hermite 曲线的算法与实现
excerpt: >
  本文主要介绍了一种自由曲线 Hermite 曲线的数学原理，并以此为依据构造了 Hermite 曲线的计算机算法，并在微软的 .net 框架下使用 GDI+ 实现。
author: 唐睿
date: 2006-01-23 22:26:35 +0800
categories: [技术]
tags: [计算机科学, 编程, 算法, 合成曲线, Hermite 曲线, .net, C#, GDI+]
layout: post
published: true
---

### 一、数学原理

曲线是构建几何模型的最基本元素之一，主要分为解析曲线和合成曲线。解析曲线通常是先有曲线方程，然后才能把曲线画出来，这种方式对于曲线的构造者来说是非常复杂且不直观的，而且改变曲线的一些参数，设计者也无法立刻了解曲线形状会做怎样的变化。在作曲线设计时，设计者通常希望能先将大致形状用很直观的方式描绘出来，并能够很容易的依照所需要的形状作修改，因此合成曲线是比较合适的方式。

合成曲线通常以参数的形式来表现，是由设计者根据其设计的需求和几何信息，去“合成”出这条曲线。这样由设计者输入的几何数据，就是曲线的控制点。

一般的合成曲线，至少需要一个三次的参数式：

![公式 1](/static/uploads/2006/hermite-equation-1.gif)

<small>公式(1)</small>

用向量表示为：

![公式 2](/static/uploads/2006/hermite-equation-2.gif)

<small>公式(2)</small>

如何确定式中的参数，就形成了不同的合成曲线的构造方法。Hermite 曲线就是通过曲线的起点（P0）、终点（P1）、起点切向量（V0）和终点切向量（V1）来确定曲线的。改变这四个参数，就可以控制 Hermite 曲线的形状。*图(1)*就是构建 Hermite 曲线的示意图。

![图 1](/static/uploads/2006/hermite-curve.png)

<small>图(1)</small>

当给定以上四个参数之后，如何来确定这条 Hermite 曲线呢？

首先将*公式(2)*作一次微分得到切线方程：

![公式 3](/static/uploads/2006/hermite-equation-3.gif)

<small>公式(3)</small>

然后将 `u=0` 和 `u=1` 代入*公式(2)*和*公式(3)*中，即得到 P0、P1、V0 和 V1：

![公式 4](/static/uploads/2006/hermite-equation-4.gif)

<small>公式(4)</small>

整理*公式(4)*得到：

![公式 5](/static/uploads/2006/hermite-equation-5.gif)

<small>公式(5)</small>

将*公式(5)*代入*公式(1)*得到：

![公式 6](/static/uploads/2006/hermite-equation-6.gif)

<small>公式(6)</small>

这就是 Hermite 曲线的参数方程。根据此方程，对于 `P0=(-2,2)`，`P1=(3,-1)`，`V0=(8,10)` 和 `V1=(15,10)` 这四个参数，可以构造如下的表格：

| **u**   | **0.0**        | **0.1**      | **0.2**      | **0.3**     | **0.4**      | **0.5**      |
| **P**   | (-2, 2)        | (-1.3, 2.6)  | (-0.94, 2.6) | (0.69, 2.2) | (-0.53, 1.4) | (-0.38, 0.5) |
| **u**   | **0.6**        | **0.7**      | **0.8**      | **0.9**     | **1.0**      |              |
| **P**   | (-0.15, -0.42) | (0.22, -1.2) | (0.82, -1.6) | (1.7, -1.6) | (3, -1)      |              |

使用 Matlab 绘制此曲线，得到*图(2)*：

![图 2](/static/uploads/2006/hermite-curve-2.png)

图(2)

但是在做曲线设计时，Hermite 曲线仍然存在不少问题，例如在建立 Hermite 曲线时设计者必须输入曲线两端切向量的大小和方向，这对设计者来讲仍然是不直观的。另外，Hermite 曲线不具有区域控制的能力，在建立 Hermite 曲线时提供的四个参数，改变任何一项输入，整条曲线的形状都会发生变化，设计者很难对其进行局部的、小范围的修改。

### 二、算法

有了以上的数学基础，构造算法就不是件困难的事情。

```csharp
/// <summary>
/// 绘制Hermite曲线的核心方法
/// </summary>
/// <param name="pen"/>绘制曲线用的Pen对象
/// <param name="p0"/>起点坐标
/// <param name="p1"/>终点坐标
/// <param name="v0"/>起点切向量
/// <param name="v1"/>终点切向量
private void DrawHermite(Pen pen, Point p0, Point p1, Point v0, Point v1) {
  // 计算出来的当前坐标
  int x = p0.X;
  int y = p0.Y;

  // 计算出来的前一个坐标，使用该两个做标连成一条线，来绘制曲线
  int preX, preY;

  // 根据参数计算每个点的坐标，参数的增量为 0.01
  for (double i = 0.0; i < = 1.0; i = i + 0.01) {
    preX = x;
    preY = y;

    // 保存计算中间结果，避免重复计算，提高算法效率
    double i2 = i * i;
    double i3 = i2 * i;
    double express = 3 * i2 - 2 * i3;

    // 计算横坐标和纵坐标
    x = (int)((1 - express) * p0.X + express * p1.X + (i - 2 * i2 + i3) * v0.X + (i3 - i2) * v1.X);
    y = (int)((1 - express) * p0.Y + express * p1.Y + (i - 2 * i2 + i3) * v0.Y + (i3 - i2) * v1.Y);

    // 画线
    this.drawingSurface.DrawLine(pen, preX, preY, x, y);
  }

  this.drawingSurface.DrawLine(pen, x, y, p1.X, p1.Y);
}
```

### 三、使用 GDI+ 实现

在 .net 框架下，使用 GDI+ 实现这个算法是件轻松的事情。但是在编程过程中仍然出现了几个问题。
      首先，如何确定两个起点和两个切向量。本程序采用了如下的方法：先选定一个点，然后拉出一条直线，以该点为起点（或终点）并以该直线的方向和长度作为起点（或终点）切向量的方向和大小。

其次，本程序可以实现类似 PhotoShop 中的钢笔功能。所以有一个如何产生拉动的效果的问题。这对这个问题使用了两种不同的解决方法：1、对于画曲线，使用了两个画笔，一个用于绘制，一个用于擦除。当鼠标移动的时候，就会使用绘制的画笔绘制新曲线，并用擦除画笔擦除刚才的曲线。但该方法会导致另一个问题，就是当该曲线覆盖到其他线条上之后，当曲线离开后，该线条就会有部分被擦掉。但是想解决这个问题是很困难的。（不知道 PhotoShop 是如何实现的。）2、画直线的时候，使用了 GDI+ 中自带的一个 ControlPaint.DrawReversibleLine() 方法，该方法可以自己解决以上的问题。

第三、像这样一遍一遍的重画和擦除，会很占用系统资源，但是没有什么更好的解决方法，从网上找到的文章来看，如果复写（Override）OnMouseDown, OnMouseMove 和 OnMouseUp 事件，会比处理此三个事件的方法要来得效率高一些，因此本程序的所有事件全部采用了这种方法。下面的这段代码，是复写了 OnMouseMove 事件，用于处理当鼠标按下左键移动的时候，产生的拉动效果。

```csharp
protected override void OnMouseMove(MouseEventArgs e) {
  // 判断当鼠标移动的时候是否有鼠标左键按下
  if (e.Button == MouseButtons.Left) {
    // isContinuedDrawing 是个标志变量，标志所产时的动作是否为了画第二个参量
    if (!isContinedDrawing) {
      endPoint[0].X = e.X;
      endPoint[0].Y = e.Y;

      // 使用 ControlPaint 画直线
      ControlPaint.DrawReversibleLine(PointToScreen(startPoint[0])PointToScreen(previousPoint), Color.Black);
      ControlPaint.DrawReversibleLine(PointToScreen(startPoint[0]), PointToScreen(endPoint[0]), Color.Black);

      previousPoint = endPoint[0];
    } else {
      endPoint[1].X = e.X;
      endPoint[1].Y = e.Y; ControlPaint.DrawReversibleLine(PointToScreen(startPoint[1]), PointToScreen(previousPoint), Color.Black);

      Point _v0 = new Point();
      _v0.X = endPoint[0].X - startPoint[0].X;
      _v0.Y = endPoint[0].Y - startPoint[0].Y;

      Point _v1 = new Point();
      _v1.X = previousPoint.X - startPoint[1].X;
      _v1.Y = previousPoint.Y - startPoint[1].Y;

      // 擦除以前的曲线
      this.DrawHermite(erasePen,startPoint[0], startPoint[1], _v0, _v1);
      // 画新的曲线
      ControlPaint.DrawReversibleLine(PointToScreen(startPoint[1]), PointToScreen(endPoint[1]), Color.Black);
      _v1.X = endPoint[1].X - startPoint[1].X;
      _v1.Y = endPoint[1].Y - startPoint[1].Y;

      this.DrawHermite(drawPen,startPoint[0], startPoint[1], _v0, _v1);
      previousPoint = endPoint[1];
    }
  }

  // 调用父类中的 OnMouseOver 事件三
  base.OnMouseMove(e);
}
```

### 四、参考文献

* [设计几何模型的建构](http://designer.mech.yzu.edu.tw/class/mechancialDesign/abst/96_ppt_chi/ch12.pdf)
