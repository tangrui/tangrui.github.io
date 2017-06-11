---
ID: 38
title: 如何重写 Equals 方法
author: 唐睿
date: 2006-08-28 17:32:14 +0800
categories: [技术]
tags: [C#, .net]
excerpt: 如何在 C# 中重写 Equals 方法。
layout: post
published: true
---

不要以为这是件容易的事情，先来看一个正确的 Equals 方法应该具备的条件：

1. obj.Equals(obj) 应该永远返回真
2. obj1.Equals(obj2) 和 obj2.Equals(obj1) 应该返回相同的结果
3. 如果 obj1.Equals(obj2) 而 obj2.Equals(obj3) 那么 obj1.Equals(obj3) 应该也为真

别以为我的智商就小学水平，回去看看你的代码，这些都能做到吗？写一个具备如此条件的 Equals 方法不是件容易的事情！从实现角度说，一般来说分为两种情况：

一、给直接继承自 Object 基类的类型重写 Equals 方法

```csharp
public class MyType {
  private int x;
  private string s;

  public override bool Equals(object obj) {
    // 如果 obj 是空的话，就直接返回假
    // 因为当前对象不会是空，否则在还没有调用 Equals 方法之前就会先抛出空指针异常
    if (obj == null) return false;

    // 如果 obj 与当前对象不属于同一个类型，也直接返回假
    // 显然两个不同类型的对象不可能相等
    if (this.getType() != obj.getType()) return false;

    // 这个强制类型转换是不会抛出异常的
    // 因为你已经知道 obj 属于该类型
    MyType o = (MyType)obj; 

    // 最后逐个比较他们的成员变量是否相等就可以了
    if (this.x != o.x || this.s != o.s) return false;

    return true;
  }
}
```

二、给间接继承自 Object 基类的类型重写 Equals 方法

```csharp
// MyType2 继承自 MyType，而不是直接继承自 Object
public class MyType2 : MyType {
  private DateTime d;
    
  public override bool Equals(object obj) {
    // 如果基类认为这两个对象不相等
    // 直接返回假
    if (!base.Equals(obj)) return false;

    // 两种实现方法的区别仅在于此，下面都是一样的
    // 直接继承自 Object 的类型之所以没有上面这一段
    // 是因为 Object 判断两个对象是否相等的逻辑非常简单
    // 通常就是判断他们的引用是否相等。因为当前对象和 obj 很有可能是不同的引用
    // 所以这种情况下，你的 Equals 方法绝大多数情况下都会返回假
    // 如果 obj 是空的话，就直接返回假
    // 因为当前对象不会是空，否则在还没有调用 Equals 方法之前就会先抛出空指针异常
    if (obj == null) return false;

    // 如果 obj 与当前对象不属于同一个类型，也直接返回假
    // 显然两个不同类型的对象不可能相等
    if (this.getType() != obj.getType()) return false;

    // 这个强制类型转换是不会抛出异常的
    // 因为你已经知道 obj 属于该类型
    MyType o = (MyClass)obj;

    // 最后逐个比较他们的成员变量是否相等就可以
    if (this.d != o.d) return false;

    return true;
  }
}
```

参考了 Jeffry Richter 的 Applied Microsoft.NET Framework Programming 一书。
