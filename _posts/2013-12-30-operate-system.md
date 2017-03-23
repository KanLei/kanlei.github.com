---
layout: post
title: "Operate System"
description: ""
category: "computer"
tags: [computer]
---
{% include JB/setup %}

操作系统的部分知识结构图，采用 Blumind 绘制。欢迎指正和补充

[知识结构图]({{ site.url }}/assets/pictures/OperateSystem.png)

### 概念

##### 地址空间(Address Space)
> 操作系统将单个物理内存抽象成供多个进程使用的私有虚拟内存的过程叫做内存虚拟化(Virtualize Memory)。进程在虚拟内存中所处的空间范围叫做进程的地址空间，所处的位置地址叫做虚拟地址。一切程序中可见的地址都是虚拟地址。