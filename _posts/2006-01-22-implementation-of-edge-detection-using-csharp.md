---
ID: 22
title: 使用 C# 实现图像的边缘检测
author: 唐睿
date: 2006-01-22 22:47:13 +0800
categories: [技术]
tags: [编程, 图形图像处理, 边缘检测]
layout: post
published: true
---

我本人对图像处理没什么兴趣，要不是这门课要交作业，我才懒得做这些东西。唉……不过程序写了，自然会有一点想法，发到 Blog 上，以备后用吧。

但是，即便是写了程序，也仍然不知道作边缘检测的原理何在，只是模糊的知道大概是对图像灰度求梯度，梯度大的就是边缘了。但毕竟图像是离散化的，可以使用另外的方法求梯度，而不用像高数中那样拼命地算偏导数了。有很多学者提出了很多种不同性能的模板，只要按照模板作简单的四则运算就行了，当然这也是能用程序实现的关键。

由于作为教学课程，所有的内容都是以简单的灰度图像来说明举例的，当然边缘检测也不例外，留作业写程序也是一样，所以马上就遇到的一个问题是如何将彩色图像转为灰度图像。在课程中，都是简单的认为灰度图像只有一个亮度，这自然是没错的，但是放到计算机里，灰度也是一种颜色，是颜色就要使用色彩模式（最常用的自然是 RGB 了），那么这种灰度到底应该是怎样的颜色编码呢？索性取向 Photoshop ，看一看各种灰度色调，终于有所发祥。其实也可以这么想，全黑是 #000000 ，全白是 #FFFFFF ，那么是不是只要 RGB 值都相等，这个颜色就是灰度色呢？试了一下，果然如此。这样就好办了，至少第一步知道了转换的目标是什么了。但马上就又有了一个问题，彩色图片的颜色这么多，那么如何知道哪种彩色颜色对应哪种灰度颜色呢？这一点我从 .net Framework 中找到了答案。其实我从一开始就想在 .net Framework 中寻找有没有直接将 RGB 转成灰度或者是 HIS 模式（因为 HIS 模式中的 I 就使亮度，自然就容易转成灰度了）的，是不是太奢望了，所以我也没抱太大的希望，但是在这过程中却发现 Color 中有关一个实例方法 GetBrightness() ，就是用来获得颜色亮度的，真是踏破铁鞋无觅处，得来全不费功夫。该方法返回一个 0~1 之间的浮点数，那么如果 RGB 每个各占一个字节的话，那么刚好可以用这个值去乘以 255 ，然后拼成一个 RGB ，这个颜色就是原始色彩所对应的灰色。程序代码如下：

```csharp
// 定义两个颜色变量，oColor 为原始色彩，gColor 为对应的灰度色彩
Color oColor, gColor;
// 原始色彩的亮度
float brightness;
// 灰度色彩用 RGB 来表示，由于 R=G=B 所以只用一个变量就可以了
int gRGB;
// 遍历图像中的每个像素
for (int i = 0; i < oBmp.Width; i++) {
  for (int j = 0; j < oBmp.Height; j++) {
    // 得到像素的原始色彩
    oColor = oBmp.GetPixel(i, j);
    // 得到该色彩的亮度
    brightness = oColor.GetBrightness();
    // 用该亮度计算灰度
    gRGB = (int)(brightness * 255);
    // 组成灰度色彩
    gColor = Color.FromArgb(gRGB, gRGB, gRGB);
    // 最后将该灰度色彩赋予该像素
    gBmp.SetPixel(i, j, gColor);
  }
}
```

其实还是很简单的。这之后就可以按照书中所说的模板游历的方法来进行边缘检测了。程序如下：

```csharp
// template为模板，nThreshold 是一个阈值，
// 用来将模板游历的结果（也就是梯度）进行划分。
// 大于阈值的和小于阈值的分别赋予两种颜色，白或黑来标志边界和背景
private void EdgeDectect(int[,] template, int nThreshold) {
  // 取出和模板等大的原图中的区域
  int[,] gRGB = new int[3, 3];
  // 模板值结果，梯度
  int templateValue = 0;
  // 遍历灰度图中每个像素
  for (int i = 1; i < gBmp.Width - 1; i++) {
    for (int j = 1; j < gBmp.Height - 1; j++) {
      // 取得模板下区域的颜色，即灰度
      gRGB[0,0] = gBmp.GetPixel(i - 1, j - 1).R;
      gRGB[0,1] = gBmp.GetPixel(i - 1, j).R;
      gRGB[0,2] = gBmp.GetPixel(i - 1, j + 1).R;
      gRGB[1,0] = gBmp.GetPixel(i, j - 1).R;
      gRGB[1,1] = gBmp.GetPixel(i, j).R;
      gRGB[1,2] = gBmp.GetPixel(i, j + 1).R;
      gRGB[2,0] = gBmp.GetPixel(i+1, j - 1).R;
      gRGB[2,1] = gBmp.GetPixel(i + 1, j).R;
      gRGB[2,2] = gBmp.GetPixel(i + 1, j + 1).R;
      // 按模板计算
      for (int m = 0; m < 3; m++) {
        for (int n = 0; n < 3; n++) {
          templateValue += template[m, n] * gRGB[m, n];
        }
      }
      // 将梯度之按阈值分类，并赋予不同的颜色
      if (templateValue > nThreshold) {
        eBmp.SetPixel(i, j, Color.FromArgb(255, 255, 255)); // 白
      } else {
        eBmp.SetPixel(i, j, Color.FromArgb(0, 0, 0)); // 黑
      }
      templateValue = 0;
    }
  }
}
```
