---
ID: 54
title: IE 和 Firefox 在行为上的差别
author: 唐睿
date: 2006-07-03 15:51:02 +0800
categories: [技术]
tags: [IE, Firefox, JavaScript]
layout: post
published: true
---

1、在 IE 中，不同的 document 创建的 element 不能够相互 append

有两个 frame，name 分别为 frame1 和 frame2，有如下代码：

```javascript
var div1 = frame1.document.createElement("div");
var div2 = frame2.document.createElement("div");
div1.appendChild(div2);
```

这段代码，在 Firefox 中工作正常，而在 IE 中会报告“无效参数（Invalid Arguments）”。当然, IE 的这种行为是正确且安全的。

2、在 IE 中需要使用规范的属性名称

有如下的 HTML 代码:

```html
<table id="table1">
  <tbody id="tbody1">
    <tr>
      <td>123</td>
      <td>456</td>
      <td>789</td>
    </tr>
  </tbody>
</table>
```

现在用 JavaScript 往里面插入一个新行：

```javascript
var domTr = document.createElement("tr");
var domTd = document.createElement("td");
domTd.setAttribute("colSpan", "3");
domTd.innerHTML = "123456789";
domTr.appendChild(domTd);
document.getElementById("tbody1").appendChild(domTr);
```

注意第三行的 colSpan 属性, 在这里如果使用 colspan (也就是说 s 没有大写的话) IE 会无法识别这个属性，但是在 Firefox 里面这两种形式都可以正常工作。

显然 IE 在这个地方的处理就有些太苛刻了，所以用 IE 来设置 DOM 元素的属性的时候最好按照“第一个单词的首字母小写，其余单词的首字母均大写”的规则来进行。
