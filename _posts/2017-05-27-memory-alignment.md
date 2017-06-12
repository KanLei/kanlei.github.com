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

在 `Swift` 语言中，我们可以通过 `Mirror` 结构体来访问类型的成员，但却无法对其值进行更改，唯一更改值的方式是绕过 `Mirror` 直接对实例所在的内存地址进行存取，这就需要我们了解类型在内存的中布局规则。

`Swift` 中我们可以使用 **&** 得到实例所在的内存地址，但只能以参数传递的方式使用 **&**，而不能直接在表达式或语句中使用。`Swift` 为我们提供了获取实例指针的便捷方法。

- withUnsafePointer() - UnsafePointer\<T>
- withUnsafeMutablePointer() - UnsafeMutablePointer\<T>
- withUnsafeBytes() - UnsafeRawBufferPointer
- withUnsafeMutableBytes() - UnsafeMutableRawBufferPointer
- [UnsafeRawPointer](https://developer.apple.com/reference/swift/unsaferawpointer)
- [UnsafeMutableRawPointer](https://developer.apple.com/reference/swift/unsafemutablerawpointer)
- UnsafeBufferPointer\<T\>
- UnsafeMutableBufferPointer\<T>

**Mutable** 是指一个可更改的指针；**\<T>** 是指绑定到指定类型; **Buffer** 是指一块连续的内存地址，如 `Array`；**Raw** 是指一个处理原始字节的指针，与内存是否绑定到指定类型无关，由于是直接移动字节单位的位置，可以使用 `load(as: <T.Type>)` 方法替换 `pointee` 取值。

#### struct

以 `Data` 为例，如果我们想通过内存地址获取和修改 `data2` 的值，我们可以这样做：

```swift
var data = Data(data1: 1, data2: 2, data3: 3)
print(data.data2)  // 2

withUnsafeMutablePointer(to: &data) { pointer in
    var ptr = UnsafeMutableRawPointer(pointer)
                .advanced(by: 4)
                .assumingMemoryBound(to: Int32.self)
    ptr.pointee = 20
    
}

print(data.data2)  // 20
```

首先将 `data` 实例作为参数传递给 `withUnsafeMutablePointer` 方法，得到类型为`UnsafeMutablePointer<Data>` 的 `pointer`，以便于我们可以对指针进行更改，接着将 `pointer` 转化为 `UnsafeMutableRawPointer`，方便将指针按字节进行移动(advanced 移动的单位由当前指针类型所决定，如果要移动单个字节单位，需要先转化为 UnsafeMutableRawPointer 类型)，移动 4 个字节的距离，以保证当前指针所处位置为 `data2` 的起始地址，最后将指针 **Bound** 到指定类型，以便于安全的对数据进行读取。

#### class

下面我们来尝试访问类实例在内存中布局状态，首先将 `Data` 定义由 `Struct` 更改为 `Class`，如下

```swift
class Data {
    var data1: Int8
    var data2: Int32
    var data3: Int16
    
    init(data1: Int8, data2: Int32, data3: Int16) {
        self.data1 = data1
        self.data2 = data2
        self.data3 = data3
    }
}
```

获取 `Data` 类占用 `size` 字节数：

```swift
MemoryLayout<Data>.size    // 8
```

不管我们在 `Data` 中定义多少字段，`size` 都为 8 。这是因为类实例本身存储在堆中，在栈上只存储指向类实例的指针，而指针在 64bit CPU 上占 8 个字节。如果是这样，那我们如何获取堆上实例所在的起始地址呢？`Swift` 为我们提供了 `Unmanaged`，通过该结构体中定义的方法我们可以方便地获取实例真实的存储地址。

```swift
Unmanaged.passUnretained(data).toOpaque()
```

下面我们来验证一下使用 `withUnsafeMutablePointer` 方法获取的指针中的值是不是真的就是实例所在的真实地址。

```swift
var data = Data(data1: 1, data2: 2, data3: 3)

withUnsafeMutablePointer(to: &data, { ptr in
    print(ptr)  // 0x0000000108fffe20
    let address = UnsafeRawPointer(ptr)
        .assumingMemoryBound(to: Int.self).pointee
    print(String(address, radix: 16, uppercase: false))  // 7fdb53c0dc60
})

print(Unmanaged.passUnretained(data).toOpaque())  //  0x00007fdb53c0dc60
```

首先，我们打印出 `data` 指针 `ptr`，然后将指针绑定到 `Int` 类型，获取值后将其转化为 16 进制并打印)，最后将其与我们通过使用 `Unmanaged` 获取的地址进行对比，可见，`ptr` 并不是实例所在的地址，`ptr` 中存储的地址才是实例在堆上的地址。

知道了如何获取类实例的地址，我们来尝试下如何通过地址访问值。

```swift
let ptr = Unmanaged.passUnretained(data).toOpaque()
ptr.advanced(by: 16).assumingMemoryBound(to: Int8.self).pointee
```

上面这段代码会打印 1，即 `data1` 的值。

> 我们将指针移动了 16 个字节的单位，是因为 `class` 在内存中的表示会默认将 *type* 和 *reference count* 两个值存储在实例的起始位置，并各自占用 8 个字节，如果要访问第一个字段的值，首先要移动指针到第一个字段的起始地址。


[Code](https://github.com/KanLei/ExtensionMirror)


[*The Swift Reflection API and what you can do with it*](https://appventure.me/2015/10/24/swift-reflection-api-what-you-can-do/) / 
[*Purpose of memory alignment*](https://stackoverflow.com/questions/381244/purpose-of-memory-alignment) / 
[*UnsafeRawPointer API*](https://github.com/apple/swift-evolution/blob/master/proposals/0107-unsaferawpointer.md) / 
[*Swift 中的指针使用*](https://onevcat.com/2015/01/swift-pointer/) / 
[*Swift进阶之内存模型和方法调度*](http://blog.csdn.net/hello_hwc/article/details/53147910) / 
[*Swift 对象内存模型探究1*](https://mp.weixin.qq.com/s/zIkB9KnAt1YPWGOOwyqY3Q) /
[*Using Pointers And Interacting With C*](https://www.raywenderlich.com/148569/unsafe-swift)