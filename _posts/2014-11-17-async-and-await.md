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
	string result = await conent;
	return result;
}
```

上面的代码中，当方法执行到 `client.GetStringAsync(url)` 时会异步执行 `GetStringAsync`，并立即返回一个 `Task<stirng>` 对象给 `content`，方法继续执行，直到遇到 `await` 后， 如果 `content` 还未完成，则立即返回给方法的调用者，并开启一个新的线程执行后续代码（即方法体中的剩余代码，如果 `await` 处于是循环体内，那下一次循环就是使用线程池线程来执行）；如果可以立即获得结果，会继续在主线程执行方法体中剩余的代码并返回，方法调用者获取 `Task<string>` 对象后，可以继续往下执行，从而避免了主线程被阻塞，从而实现我们所需要的异步效果；也可以调用 `Wait()` 方法阻塞主线程，直到获取结果。

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

## 异常捕获

`async` 方法的返回值类型有 `void`, `Task`, `Task<T>`，其中 `void` 返回类型导致异常直接抛出，而无法被 `try/catch` 捕获到，只用于事件注册。`Task` 和 `Task<T>` 返回类型默认会把异常包装到 `Task` 对象中，只有调用该对象的相关方法如，`await` 该 `Task` 才会触发异常并抛出。

注意 `async lambda` 要使用 `Func<Task>` 作为类型，以为 `Action` 类型默认返回值为 `void`，会导致异常无法被正常捕获。

## 上下文

#### 同步上下文

同步上下文与线程有关，在 `UI` 线程中请求 `SynchronizationContext.Current` 得到如 `UIKitSynchronizationContext` 的上下文对象；如果在工作线程中请求 `SynchronizationContext.Current` 得到 null，表示默认使用 `SynchronizationContext` 的实现。

`SynchronizationContext` 的默认实现与 `UI` 无关，在其调用线程上执行同步委托，在 ThreadPool 中执行异步委托。

`await` 默认会捕获同步上下文，如果是 `GUI` 程序，捕获的是 `UI` 线程，所以 `await` 后的更新 `UI` 界面的操作是合法的；如果是 `Console` 程序，默认捕获的是线程池线程。然而，捕获 `UI` 线程会导致占用主线程资源，因此最好的方式是不捕获 `UI` 线程，使用线程池线程来执行。

``` c#
// 使用线程池线程
await DownloadAsync("http://...").ConfigAwait(false);
```
执行不进行 UI 操作的异步方法应该使用 `ConfigAwait(false)` 来提升性能，特别是类库中异步方法。 `await` 的方法立即执行结束时，此如果时该方法还未捕获当前同步上下文，因此执行在 UI 线程上，所以后续方法仍需要执行 `ConfigAwait(false)` 避免捕获同步上下文。

每个方法的执行上下文是独立的，因此如果 A -> B, 在 B 中使用 `ConfigAwait(false)` 后，B 中剩余的方法在线程池线程上执行，而 A 中的剩余方法仍然在 UI 线程中执行。

#### 执行上下文

线程执行时默认会拥有自己的上下文，上下文中保存的线程执行时寄存器中的值和一些环境信息，当启动一个新的线程时，默认执行上下文会流向新线程，以便于新线程能够访问原线程的一些信息，通常我们不会去考虑执行上下文。关于同步上下文与执行上下文 [ExecutionContext vs SynchronizationContext](http://blogs.msdn.com/b/pfxteam/archive/2012/06/15/executioncontext-vs-synchronizationcontext.aspx) 。

## 异步执行 `I/O` 密集型操作

``` c#
using (FileStream fs = new FileStream(filePath, FileMode.Open, FileAccess.Read, FileShare.Read, 8, true)) 
{ 
	await fs.ReadAsync(...);
} 

```

普通的 `I/O` 操作，当线程从 user-mode 到 kernel-mode 读取磁盘文件时，由于读取磁盘文件的操作不涉及到线程，因此线程会被阻塞，直到文件读取完成，线程获取读取结果并恢复执行。而异步的 `I/O` 操作，上面 `true` 参数和 `ReadAsync` 方法结合使用，当读取磁盘文件时，线程不会被阻塞，而是立即返回，如果是线程池线程，则返回线程池，等下执行下一个任务单元。这种方式减少了不必要线程的创建，从而解决了系统资源，提升了性能，特别是在执行数据库访问操作时，大大增加了服务器的可响应性，可扩展性。[Performing IO Bounds Asynchronous Operations](https://www.wintellectnow.com/Videos/Watch/performing-i-o-bound-asynchronous-operations?videoId=performing-i-o-bound-asynchronous-operations) 

**Note**:

[*The zen of async: Best practices for best performance*](https://channel9.msdn.com/events/Build/BUILD2011/TOOL-829T)

> Cache tasks when applicable.  
> Use ConfigureAwait(false) in libraries.  
> Avoid gratuitous use of ExecutionContext.  
> Remove unnecessary locals.  
> Avoid unnecessary awaits.


[*Why use async await*](https://msdn.microsoft.com/en-us/magazine/hh456403.aspx) / 
[*Asynchronous Programming*](http://msdn.microsoft.com/en-us/library/hh191443(v=vs.110).aspx) / 
[*Scott Hanselman*](http://www.hanselman.com/blog/TheMagicOfUsingAsynchronousMethodsInASPNET45PlusAnImportantGotcha.aspx) / 
[*asp.net*](http://www.asp.net/web-forms/overview/performance-and-caching/using-asynchronous-methods-in-aspnet-45) /
[*Be Aware Of The Synchronization-Context*](http://www.gamlor.info/wordpress/2010/10/c-5-0-async-feature-be-aware-of-the-synchronization-context/) / [*Understanding-SynchronizationContext*](http://www.codeproject.com/Articles/31971/Understanding-SynchronizationContext-Part-I) / 
[*Synchronous and Asynchronous I/O*](http://msdn.microsoft.com/en-us/library/windows/desktop/aa365683(v=vs.85).aspx)
