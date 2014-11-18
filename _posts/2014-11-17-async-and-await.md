---
layout: post
title: "Async and Await"
description: ""
category: "article"
tags: [c#, async await]
---
{% include JB/setup %}


## 异步的开始点


``` c#
public async Task<string> DownloadAsync(string url)
{
	...
	Task<string> content = client.GetStringAsync(url);
	...
	string result = awiat conent;
	return result;
}
```

上面的代码中，当方法执行到 `client.GetStringAsync(url)` 时会异步执行 `GetStringAsync`，并立即返回一个 `Task<stirng>` 对象给 `content`，方法继续执行，直到遇到 `await` 后， 如过 `content` 还未完成，则立即返回给方法的调用者，并开启一个新的线程执行后续代码（即方法体中的剩余代码，如果 `await` 处于是循环体内，那下一次循环就是使用线程池线程来执行）；如果可以立即获得结果，会继续在主线程执行方法体中剩余的代码并返回，方法调用者获取 `Task<string>` 对象后，可以继续往下执行，从而避免了主线程被阻塞，从而实现我们所需要的异步效果；也可以调用 `Wait()` 方法阻塞主线程，直到获取结果。

## 自定义 async 方法

`await` 一个自己定义的普通方法并不会使方法的执行开启一个新的线程，而是当真正调用 `FCL` 中的可等待方法时才有可能会开启一个新的线程。

``` c#
public async Task MyMethodAysnc()
{
	Task<string> result = await DownloadAsync("http://...");
	...
}
```

这里的 `await DownloadAsync("http://...")` 并不会开始异步执行，而是直到执行到方法体中的 `client.GetStringAsync(url)` 时，才会开始真正的异步。

## 捕获同步上下文

`await` 默认会捕获同步上下文，如果是 `GUI` 程序，捕获的是 `UI` 线程，所以 `await` 后的更新 `UI` 界面的操作是合法的；如果是 `Console` 程序，默认捕获的是线程池线程。然而，捕获 `UI` 线程会导致占用主线程资源，因此最好的方式是不捕获 `UI` 线程，使用线程池线程来执行。

``` c#
// 使用线程池线程
await DownloadAsync("http://...").ConfigAwait(false);
```

线程执行时默认会拥有自己的上下文，上下文中保存的线程执行时寄存器中的值和一些环境信息，当启动一个新的线程时，默认执行上下文会流向新线程，以便于新线程能够访问原线程的一些信息，通常我们不会去考虑执行上下文。关于同步上下文与执行上下文 [ExecutionContext vs SynchronizationContext](http://blogs.msdn.com/b/pfxteam/archive/2012/06/15/executioncontext-vs-synchronizationcontext.aspx) 。

## 异步执行 `I/O` 密集型操作

异步执行计算密集型操作就如同我们之前用多线程的效果是相同的。`C#` 5 中新的异步模式，带来了真正的异步 `I/O` 密集型操作。

``` c#
using (FileStream fs = new FileStream(filePath, FileMode.Open, FileAccess.Read, FileShare.Read, 8, true)) 
{ 
	await fs.ReadAsync(...);
} 

```

普通的 `I/O` 操作，当线程从 user-mode 到 kernel-mode 读取磁盘文件时，由于读取磁盘文件的操作不涉及到线程，因此线程会被阻塞，直到文件读取完成，线程获取读取结果并恢复执行。而异步的 `I/O` 操作，上面 `true` 参数和 `ReadAsync` 方法结合使用，当读取磁盘文件时，线程不会被阻塞，而是立即返回，如过是线程池线程，则返回线程池，等下执行下一个任务单元。这种方式减少了不必要线程的创建，从而解决了系统资源，提升了性能，特别是在执行数据库访问操作时，大大增加了服务器的可响应性，可扩展性。 [Synchronous and Asynchronous I/O](http://msdn.microsoft.com/en-us/library/windows/desktop/aa365683(v=vs.85).aspx)


[*Asynchronous Programming*](http://msdn.microsoft.com/en-us/library/hh191443(v=vs.110).aspx) / 
[*Scott Hanselman*](http://www.hanselman.com/blog/TheMagicOfUsingAsynchronousMethodsInASPNET45PlusAnImportantGotcha.aspx) / 
[*asp.net*](http://www.asp.net/web-forms/overview/performance-and-caching/using-asynchronous-methods-in-aspnet-45)
