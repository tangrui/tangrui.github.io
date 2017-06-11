---
ID: 19
title: 设计模式 — Builder
author: 唐睿
date: 2006-01-22 22:41:53 +0800
categories: [技术]
tags: [编程, 设计模式, GoF, C#]
layout: post
published: true
---

上下文：一个产品有 n 个零件，通过构建产品的不同部分来生产不同的产品。

解决方法：将一个产品的不同部件交由某个构建者的不同方法来实现，然后导演通过调用不同的方法，构建出不同的产品。

```csharp
using System;

namespace Builder {
  /// <summary>
  /// 产品，其中包括两个部件
  /// </summary>
  public class Product {
    string part1 = string.Empty;
    string part2 = string.Empty;

    public string Part1 {
      get { return part1; }
      set {  part1 = value; }
    }

    public string Part2 {
      get { return part2; }
      set { part2 = value; }
    }
  }

  /// <summary>
  /// 构建者的接口标准
  /// </summary>
  public interface IBuilder {
    void BuildPart1();
    void BuildPart2();
    Product RetrieveProduct { get; }
  }

  /// <summary>
  /// 从接口派生出具体的构建者
  /// </summary>
  public class ConcreteBuilder:IBuilder {
    Product myProduct = new Product();

    public void BuildPart1() {
      // TODO:  添加 ConcreteBuilder.BuildPart1 实现
      myProduct.Part1 = "Part1 has been built!";
    }

    public void BuildPart2() {
      // TODO:  添加 ConcreteBuilder.BuildPart2 实现
      myProduct.Part2 = "Part2 has been built!";
    }

    public Product RetrieveProduct {
      get {
        // TODO:  添加 ConcreteBuilder.RetrieveProduct getter 实现
        return myProduct;
      }
    }
  }

  /// <summary>
  /// 导演通过调用具体构建者的不同构建方法，构建不同的部件，进而行成不同的产品
  /// 该导演生产了这样的三种产品
  /// 1、具有零件 1 和 2 的产品
  /// 2、只具有零件 1 的产品
  /// 3、只具有零件 2 的产品
  /// </summary>
  public class Director {
    Product product1, product2, product3;

    public Product Product1 {
      get { return product1; }
    }
    public Product Product2 {
      get { return product2; }
    }
    public Product Product3 {
      get { return product3; }
    }

    public Director() { }

    public void Construct() {
      ConcreteBuilder myCB1 = new ConcreteBuilder();
      myCB1.BuildPart1();
      myCB1.BuildPart2();
      product1 = myCB1.RetrieveProduct;

      ConcreteBuilder myCB2 = new ConcreteBuilder();
      myCB2.BuildPart1();
      product2 = myCB2.RetrieveProduct;

      ConcreteBuilder myCB3 = new ConcreteBuilder();
      myCB3.BuildPart2();
      product3 = myCB3.RetrieveProduct;
    }
  }

  /// <summary>
  /// 测试 Builder 模式
  /// </summary>
  class TestBuilder {
    /// <summary>
    /// 应用程序的主入口点。
    /// </summary>
    [STAThread]
    static void Main(string[] args) {
      Director myDirector = new Director();
      myDirector.Construct();
      Console.WriteLine(myDirector.Product1.Part1);
      Console.WriteLine(myDirector.Product1.Part2);
      Console.WriteLine(myDirector.Product2.Part1);
      Console.WriteLine(myDirector.Product2.Part2);
      Console.WriteLine(myDirector.Product3.Part1);
      Console.WriteLine(myDirector.Product3.Part2);

      Console.Read();
    }
  }
}
```
