---
layout: post
title: "Memory Alignment"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### 什么是内存对齐？

现在的计算机普遍采用的是 64bit 和 32bit 的 CPU，而为了让 CPU 更高效地读写内存，采用了按 [**word size**](https://stackoverflow.com/questions/19821103/what-does-it-mean-by-word-size-in-computer) 来读取的方式（64bit word size 是 8 字节，32bit word size 是 4 字节），之所以这么做，是为了避免 CPU 为了读取一个类型的值而多次访问内存，造成额外的访问开销。实现这种高效访问内存的手段就是要让类型存储地址保证对齐，保证 CPU 一次就可以完成读取和写入操作。

### 内存如何对齐？

```swift
struct Data {
    var data1: Int8
    var data2: Int32
    var data3: Int16
}
```

以 `Swift` 中的结构体定义为例，猜测一下 `Data` 结构体的占多少个 `size` 字节？是 7 个字节吗？

```swift
MemoryLayout<Data>.size    // 10
```

通过 `MemoryLayout` 的 `size` 属性我们得到 `Data` 占用 10 个字节。

> 1(Int8) + 4(Int32) + 2(Int16) 结果不是 7 吗？
 
之所以结果不是 7 而是 10 就是因为内存对齐的缘故。

```swift
MemoryLayout<Int8>.alignment    // 1
MemoryLayout<Int32>.alignment   // 4
MemoryLayout<Int16>.alignment   // 2
```

`Swift` 中我们可以通过 `alignment` 属性得到对齐大小。`Int8` 的对齐大小为 1，所以 `Int8` 所在的起始地址必须是 1 的倍数；`Int32` 的对齐大小为 4，起始地址必须是 4 的倍数；`Int16` 对齐大小为 2，起始地址必须是 2 的倍数。从上可以得出结论，**类型存储的起始地址必须是类型对齐大小的倍数**。

通过上面的描述，真实的内存布局应该如下所示：

```swift
struct Data {
    var data1: Int8     // address 01
    // padding 3
    var data2: Int32    // address 04
    var data3: Int16    // address 05
}
```

由于 `Int32` 的起始地址必须从 4 的倍数开始，所以，必须在 `Int8` 之后空出 3 个字节的大小，以保证 `Int32` 起始地址为 04。这就解释了为什么 `Data` 的 `size` 大小为 10。

> 1(Int8) + 3(padding) + 4(Int32) + 2(Int16) = 10
 
这里的 `size` 只是表示 `Data` 结构体自身的字节大小，但并不表示 `Data` 在内存中占用的字节大小，什么意思呢？`Swift` 为我们提供了另外一个属性 `stride`，表示占用真实的内存空间的大小。

```swift
MemoryLayout<Data>.stride    // 12
```

`Data` 占用内存为 12 个字节，比其自身(10)多占用了 2 个字节。这是因为**类型所占用的总的空间大小必须是最大对齐大小的倍数**，这里最大对齐大小是 4，所以真实内存大小是 4 的倍数，且 > 10，即最小占用 12 个字节。

填充后真实的内存布局应该如下所示：

```swift
struct Data {
    var data1: Int8     // address 01
    // padding 3
    var data2: Int32    // address 04
    var data3: Int16    // address 05
    // padding 2
}
```

### 内存布局的应用

在 `Swift` 语言中，我们可以通过 `Mirror` 结构体来访问类型的成员，但却无法对其值进行更改，只能绕过 `Mirror` 直接对对象所在的内存地址进行存取，这就需要我们了解类型在内存的中布局规则。


[Code](https://github.com/KanLei/ExtensionMirror)


[*The Swift Reflection API and what you can do with it*](https://appventure.me/2015/10/24/swift-reflection-api-what-you-can-do/) / 
[*Purpose of memory alignment*](https://stackoverflow.com/questions/381244/purpose-of-memory-alignment) / 
[*Swift 中的指针使用*](https://onevcat.com/2015/01/swift-pointer/) / 
[*Swift进阶之内存模型和方法调度*](http://blog.csdn.net/hello_hwc/article/details/53147910) / 
[*Swift 对象内存模型探究1*](https://mp.weixin.qq.com/s/zIkB9KnAt1YPWGOOwyqY3Q) /
[*Using Pointers And Interacting With C*](https://www.raywenderlich.com/148569/unsafe-swift)