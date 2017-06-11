---
ID: 17
title: 设计模式 — ProtoType
author: 唐睿
categories: [技术]
tags: [编程, 设计模式, GoF, C#]
date: 2006-01-22 22:37:00 +0800
layout: post
published: true
---

问题：当我们在编写大型应用程序的时候，可能会遇到这样的情况：构造了一个基类，然后从该基类派生出子类。在子类的数目比较少的情况下，没有任何事情会发 生，但是如果当从某一个基类派生出的子类达到一定数目的时候，维护这些子类将是一件非常繁复的工作。

解决方案：既然如此，干脆就不派生子类。无论如何，子类都是具有特性的基类的衍生，我们可以将这种特性直接在基类中反映出来。具体说来，就是实现不同种类的克隆。

方法：传递进相关特性的参数，得到具有该特性的对象。

其它：本示例代码以 C# 为示例语言，通过实现 ICloneable 和 ISerializable 来实现对象的浅表复制 (Shallow copy) 和深层复制 (Deep copy) 。这同时也是学习序列化的很好的演示示例。

附件：一个 XML 文档：

```xml
<?xml version="1.0" encoding="utf-8"?>
<employees>
  <employee>
    <firstname>Rui</firstname>
    <lastname>Tang</lastname>
  </employee>
  <employee>
    <firstname>Lichao</firstname>
   <lastname>Mu</lastname>
  </employee>
</employees>
```

```csharp
using System;
using System.IO;
using System.Xml;
using System.Collections;
using System.Runtime.Serialization;
using System.Runtime.Serialization.Formatters.Binary;

namespace ProtoType {
  /// <summary>
  /// 用于存储每个员工的基本信息
  /// </summary>
  public class Employee {
    private string firstName;
    private string lastName;

    public string FirstName {
      get { return this.firstName; }
      set { this.firstName = value; }
    }

    public string LastName {
      get { return this.lastName; }
      set { this.lastName = value; }
    }

    /// <summary>
    /// 默认构造函数
    /// </summary>
    public Employee() { }

    /// <summary>
    /// 从序列化信息中得到数据
    /// </summary>
    /// <param name="info"/>序列化信息
    /// <param name="context"/>
    /// <param name="index"/>序列化信息中字典的索引
    public Employee(SerializationInfo info, StreamingContext context, int index) {
      this.firstName = info.GetString("emp_fname" + index.ToString());
      this.lastName = info.GetString("emp_lname" + index.ToString());
    }

    /// <summary>
    /// 将数据添加到序列化信息中
    /// </summary>
    /// <param name="info"/>序列化信息
    /// <param name="context"/>
    /// <param name="index"/>序列化信息中字典的索引
    public void GetObjectData(SerializationInfo info, StreamingContext context, int index) {
      o.AddValue("emp_fname" + index.ToString(), firstName);
      info.AddValue("emp_lname" + index.ToString(), lastName);
    }
  }

  /// <summary>
  /// 处理员工信息的类
  /// </summary>
  [Serializable]
  public class EmployeeProcess : ICloneable,ISerializable {
    /// <summary>
    /// 存储所有的员工
    /// </summary>
    private ArrayList employeeArray = new ArrayList();

    /// <summary>
    /// 默认构造函数，读取XML文档，并将员工添加到employeeArray中
    /// </summary>
    public EmployeeProcess() {
      XmlDocument xmlDoc = new XmlDocument();
      Employee objEmp = null;

      xmlDoc.Load("Employees.xml");
      foreach (XmlNode node in xmlDoc.DocumentElement.ChildNodes) {
        objEmp = new Employee();
        objEmp.FirstName = node.SelectSingleNode("FirstName").InnerText;
        objEmp.LastName = node.SelectSingleNode("LastName").InnerText;
        employeeArray.Add(objEmp);
      }
    }

    /// <summary>
    /// 反序列化必须要实现的一个方法，用于在反序列化时构造对象
    /// </summary>
    /// <param name="info"/>序列化信息
    /// <param name="context"/>
    public EmployeeProcess(SerializationInfo info, StreamingContext context) {
      int count;
      string temp = string.Empty;
      Employee objEmp = null;

      count = int.Parse(info.GetValue("emp_count",temp.GetType()).ToString());
      for (int i=0; i<count ; i++) {
        objEmp = new Employee(info, context, i);
        employeeArray.Add(objEmp);
      }
    }

    /// <summary>
    /// 实现浅表复制的克隆方法
    /// 
    /// <returns>返回对象的引用</returns>
    public object Clone() {
      try { return this; }
      catch (Exception exce) {
        Console.WriteLine(exce.Message);
        return null;
      }
    }

    /// <summary>
    /// 实现深层复制的克隆方法
    /// </summary>
    /// <param name="isDeep"/>判断是否请求深层复制的标志量
    /// <returns>返回对象的深层副本</returns>
    public object Clone(bool isDeep) {
      try {
        if (isDeep) { return CreateDeepCopy(); }
        else { return this.Clone(); }
      } catch (Exception exce) {
        Console.WriteLine(exce.Message);
        return null;
      }
    }

    /// <summary>
    /// 序列化必须要实现的一个方法，将数据写到序列化信息中
    /// </summary>
    /// <param name="info"/>序列化信息
    /// <param name="context"/>
    public void GetObjectData(SerializationInfo info, StreamingContext context) {
      Employee objEmp;
      info.AddValue("emp_count",employeeArray.Count);
      for (int i=0; i<employeearray .Count; i++) {
        objEmp = (Employee)employeeArray[i];
        objEmp.GetObjectData(info,context,i);
      }
    }

    /// <summary>
    /// 枚举所有员工的信息
    ///
    /// <returns>员工信息</returns>
    public string GetEmployeeInfo() {
      string strEmpData = string.Empty;
      for (int i=0; i</employeearray><employeearray .Count; i++) {
        strEmpData = strEmpData + ((Employee)employeeArray[i]).FirstName + " " + ((Employee)employeeArray[i]).LastName + "\n";
      }
      return strEmpData;
    }

    /// <summary>
    /// 改变员工信息，用于测试深层复制和浅表复制的异同
    /// 
    public void ChangeEmployeeInfo() {
      foreach (Employee objEmp in employeeArray) {
        objEmp.FirstName = "FirstName";
        objEmp.LastName = "LastName";
      }
    }

    /// <summary>
    /// 通过序列化创建对象的深层副本
    /// </summary>
    /// <returns>返回对象的深层副本</returns>
    private EmployeeProcess CreateDeepCopy() {
      EmployeeProcess objEmpCopy;
      Stream objStream;
      BinaryFormatter objBinFormatter = new BinaryFormatter();

      try {
        objStream = File.Open("Employees.bin",FileMode.Create);
        objBinFormatter.Serialize(objStream, this);
        objStream.Close();

        objStream = File.Open("Employees.bin",FileMode.Open);
        objEmpCopy = (EmployeeProcess)objBinFormatter.Deserialize(objStream);
        objStream.Close();

        return objEmpCopy;
      } catch (Exception exce) {
        Console.WriteLine(exce.Message);
        return null;
      }
    }
  }

  /// <summary>
  /// 测试ProtoType
  /// </summary>
  class TestProtoType {
    [STAThread]
    static void Main(string[] args) {
      // 用正常的方法创建一个对象
      EmployeeProcess mainEmployeeProcess = new EmployeeProcess();
      // 输出该对象的信息
      Console.WriteLine(mainEmployeeProcess.GetEmployeeInfo());

      // 通过浅表复制得到一个刚才创建的对象的副本
      EmployeeProcess childEmployeeProcess1 = (EmployeeProcess)mainEmployeeProcess.Clone();
      // 通过深层复制得到一个刚才创建的对象的副本
      EmployeeProcess childEmployeeProcess2 = (EmployeeProcess)mainEmployeeProcess.Clone(true);
      // 改变用正常方法得到的对象的信息
      mainEmployeeProcess.ChangeEmployeeInfo();
      // 输出浅表副本对象的信息，应该已经改变
      Console.WriteLine(childEmployeeProcess1.GetEmployeeInfo());
      // 输出深成副本对象的信息，和原始的一样
      Console.WriteLine(childEmployeeProcess2.GetEmployeeInfo());

      Console.Read();
    }
  }
}
```
