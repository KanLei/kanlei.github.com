---
layout: post
title: "Provide ToString() for Type"
description: "条目5"
category: "Book"
tags: [c#]
---
{% include JB/setup %}

**1.方便调用者**

>可以在ToString()方法中提供该类型的具体信息
System.Console.WriteLine()方法、System.String.Format()方法在内部都调用了 ToString() 方法。
注：自定义值类型无相应的重载方法，因此不会调用 ToString()，导致装箱。

**2.方便调试**

>在 Debug 模式下，将鼠标放置到对象上会现实 ToString() 中所描述的内容。
也可以用特性(Attribute) DebugDisplay("Name={PropertyName},...") 来替代。

source: Effective C#: 50 Specific Ways to Improve Your C#
