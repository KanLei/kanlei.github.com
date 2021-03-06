---
layout: post
title: "Garbage Collector in C#"
description: ""
category: "Article"
tags: [c#]
---
{% include JB/setup %}

## 简述

.Net 应用程序是在托管环境中运行的，C# programmer 从来不直接从内存中删除一个托管对象（如：C 中的 free() 或 C++ 中的 delete，以及 Objective-C 中的 [release](https://www.tomdalling.com/blog/cocoa/an-in-depth-look-at-manual-memory-management-in-objective-c/)），.Net 对象会被分配到一块叫做托管堆（managed heap）的内存区域上。.Net GC(Garbage Collector) 垃圾收集器会替你控制这块托管的内存区域的回收。因此在 .Net 中你一般不用担心内存泄露、悬挂指针等问题。

## `new` 指令

使用 `new` 关键字将一个对象分配在托管堆上，然后就不用再管。

`newobj` 指令通知 CLR 执行下面的核心任务：

1. 计算分配对象所需的总内存数（包含类型的成员变量和类型的基类的必需内存）
2. 检查托管堆，确保有足够的空间来放置要分配的对象。如果空间足够，调用类型的构造函数最终将内存中新对象的引用返回给调用者，
它的地址恰好是下一个对象的指针的上一个位置
3. 在将引用返回给调用者之前，移动下一个对象的指针，指针指向托管堆上的下一个可用位置

    如果托管堆没有足够的内存来分配所请求的对象，就会进行垃圾回收

当进行垃圾回收时，垃圾回收器会暂时**挂起当前的进程中所有活动的线程** { [Concurrent GC](http://www.mono-project.com/news/2017/05/17/concurrent-gc-news/?utm_campaign=Weekly%2BXamarin&utm_medium=email&utm_source=Weekly_Xamarin_127)可以减少线程被挂起的时间 }，以保证应用程序在回收过程中不会访问堆。

## 检查回收对象

GC 能够了解某个实体目前是否依旧被应用程序的某些活动对象所引用；对于那些没有被任何活动对象直接或间接引用的对象，GC 会将其判断为垃圾。GC 在其专门的线程中运行，默默地为程序清除不再使用的内存。压缩托管堆（将当前仍旧使用的对象放在连续的内存中，可以利用**局部性原理**提高性能），因此空余的空间也会是一块连续的内存。

垃圾回收器采用的是 mark-and-compact 算法（标记和更改对象的同步块索引中的一个位(bit)），在执行垃圾回收的时候，GC 不是枚举所有访问不到的对象；相反，它是通过压缩所有相邻的可达对象来执行垃圾回收。这样，由不可访问的对象占用的内存就会被覆盖。

为了优化检查的过程，堆上的每个对象被指定属于某一代(generation)。

> 代的设计思路：对象在堆上存在的时间越长，它就更可能应该保留。

### 代
自上一次垃圾收集以来，新创建的对象属于第0代对象，而若是某个对象在经过过一次垃圾收集之后仍旧存活，那么它将成为第1代对象。两次及两次以上垃圾收集后仍旧没有被销毁的对象就变成了第2代对象。（第0代对象大多属于局部变量，而成员变量和全局变量（CLR允许全局变量的定义，即便C#中不支持）则会很快成为第1代对象，直至第2代）。

每一代都会有一个预算容量(以KB为单位)，如果分配一个新对象造成超出预算，就会启动一次垃圾回收。CLR垃圾回收器是**自调节**的，所以会根据回收垃圾对象的数量动态设置预算容量的大小。
一般来说，大概10个周期的GC中，会有一次同时检查第0代和第1代对象，大概100个周期的GC中，会有一次同时检查所有对象。

``` c#
public static void Main (string[] args)
{
	// 输出堆上的估计的字节数量
	Console.WriteLine("Estimated bytes on heap: {0}", GC.GetTotalMemory(false));

	// MaxGeneration 是从 0 开始的，为显示目的加 1
	Console.WriteLine("This OS has {0} object generations.\n", (GC.MaxGeneration + 1));

	// 强制垃圾回收，并等待每一个对象都被终结
	GC.Collect();
	GC.WaitForPendingFinalizers();

	// 只回收第 0 代对象
	GC.Collect(0);
	GC.WaitForPendingFinalizers();
}
```

## 非托管资源

.Net 提供了两种控制非托管资源生命周期的机制：终结器（finalizer）和 IDisposable 接口

终结器：

1. 终结器将由 GC 调用，调用将发生在对象成为垃圾之后的某个时间（无法确定其发生的具体时间），因此 .Net 并不能保证析构操作的确切时间。
2. 依赖终结器还会带来性能上的问题。当 GC 发现某个对象属于垃圾，但该对象需要执行终结操作时(定义了析构函数)，就不能将其直接从内存中移除。首先，GC 将调用其终结器，而终结器并不在执行垃圾收集的线程上执行。GC 将把所有需要执行终结的对象放在专门的队列中，然后**让另一个线程来执行这些对象的终结器**。这样，GC 可以继续执行其当前的工作，在内存中移除垃圾对象。而在下一次的 GC 调用时，才会从内存中移除这些已被终结的对象。因此需要调用终结器的对象将在内存中多停留一次 GC 周期的时间，如果终结对象进入第1代或第2代，那么将停留更长的 GC 周期。

因此，尽量不要使用终结器来释放非托管资源。

### IDisposable 接口：

`IDisposable.Dispose()` 方法的实现中需要完成如下4个任务*：

1. 释放所有非托管资源
2. ~~释放所有托管资源~~，包括释放事件监听程序
3. 设置一个状态标志，表示该对象已经被销毁，若是在销毁之后再次调用对象的公有方法，那么应该抛出 ObjectDisposed 异常。
4. 跳过终结操作，调用 GC.SuppressFinalize(this) 即可

由于很多非托管资源都非常宝贵（如数据库和文件句柄），所以它们应尽可能快地被清除，而不能依靠垃圾回收的发生。.Net 中使用了一种标准的模式能够在使用者正常调用是通过 IDisposable 接口释放掉非托管资源，也会在使用者忘记的情况下使用终结器释放。这个模式和 GC 配合，可以保证仅在最糟糕的情况下才调用终结器，尽可能降低其带来的性能影响。

``` c#
public class MyResource : IDisposable
{
	// 用来判断 Dispose() 是否已经被调用
	private bool disposed = false;

	public void Dispose ()
	{
		// true 表示对象用户触发了清理过程
		CleanUp(true);

		// 现在跳过终结
		GC.SuppressFinalize(this);
	}

	private void CleanUp (bool disposing)
	{
		if (!this.disposed) {
			if(disposing){
				// 释放托管资源
			}
			// 释放非托管资源
		}
		disposed = true;
	}

	~MyResource ()
	{
		// 指定 false 表示 GC 触发了清理过程
		CleanUp(false);
	}
}
```

[*Destructor*](https://msdn.microsoft.com/en-us/library/66x5fx1b.aspx)
