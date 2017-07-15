---
layout: post
title: "Async and Await"
description: ""
category: "article"
tags: [c#, async await]
---
{% include JB/setup %}


### 异步的开始点


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

上面的代码中，当方法执行到 `client.GetStringAsync(url)` 时会异步执行 `GetStringAsync`，并立即返回一个 `Task<stirng>` 对象给 `content`，方法继续执行，直到遇到 `await` 后， 如果 `content` 还未完成，则立即返回给方法的调用者，并开启一个新的线程执行后续代码（即方法体中的剩余代码，包括在循环体内）；如果可以立即获得结果，会继续在主线程执行方法体中剩余的代码并返回，方法调用者获取 `Task<string>` 对象后，可以继续往下执行，从而避免了主线程被阻塞，从而实现我们所需要的异步效果。

### 自定义 async 方法

`await` 一个自己定义的普通方法并不会使方法的执行开启一个新的线程，而是当真正调用 `FCL` 中的可等待方法时才有可能会开启一个新的线程。

``` c#
public async Task MyMethodAysnc()
{
	Task<string> result = await DownloadAsync("http://...");
	...
}
```

这里的 `await DownloadAsync("http://...")` 并不会开始异步执行，而是直到执行到方法体中的 `client.GetStringAsync(url)` 时，才会开始真正的异步。

### 异常捕获

`async` 方法的返回值类型有 `void`, `Task`, `Task<T>`，其中 `void` 返回类型导致异常直接抛出，而无法被 `try/catch` 捕获到，只用于事件注册。`Task` 和 `Task<T>` 返回类型默认会把异常包装到 `Task` 对象中，只有调用该对象的相关方法如，`await` 该 `Task` 才会触发异常并抛出。

> 注意 `async lambda` 要使用 `Func<Task>` 作为类型，因为 `Action` 类型默认返回值为 `void`，会导致异常无法被正常捕获。

使用 `async` 的标准规范是 **async all the way**，但是当主方法无法使用 `async` 关键字时，如 `static void Main()`，这时我们可以定义一个中间方法，这个中间方法的返回值为 `Task`，因此在主方法中调用该中间方法而不去 `await` 它。但是，这会引发另外一个问题，该中间方法的后续代码会被提前执行，由于我们没有显示地等待中间方法执行结束，我们可以使用 `Wait()` 或者 `Result` 方式，但这种做法会阻塞线程，且容易造成死锁。一种方式是在 `Task` 上调用 `ContinueWith` 方法，并通过指定 `TaskContinuationOptions` 的 `OnlyOnRanToCompletion` 和 `OnlyOnFaulted` 来处理任务完成或失败的情况；另一种方式是显示地使用 `Task` 手动创建一个任务并执行它，但由于我们显示地创建 `Task`，所有发生在 `Task` 中的异常都无法被抛出，因此我们需要在 `Task` 的执行方法中显示地使用 `try/catch` 进行异常捕获。[getting start with async await](https://blog.xamarin.com/getting-started-with-async-await/)

### TaskScheduler

TaskScheduler 提供了获取 Scheduler 的三种方式：

- Current: 获取 `ThreadPoolTaskScheduler`
- Default: 获取 `ThreadPoolTaskScheduler`
- FromCurrentSynchronizationContext (): 只能从 UI 线程中调用并得到包含 `SynchronizationContext.Current` 的 `SynchronizationContextTaskScheduler`，从非 UI 线程获取会触发异常

在使用 `Task.Factory.StartNew` 时，我们可以指定 `TaskScheduler`，指定任务调度所在的线程。通常应该[优先考虑](http://blog.stephencleary.com/2013/08/startnew-is-dangerous.html)使用 `Task.Run`。

我们也可以定义自己的 [TaskScheduler](https://msdn.microsoft.com/en-us/library/ee789351(v=vs.100).aspx)。

### 上下文

#### 同步上下文

> `SynchronizationContext` 是用于解决当不同的应用程序，比如 `ASP.NET`、`WinForm` 或者 `WPF` 共用某一类库时，如果想要执行更新 `UI` 操作，必须区分对待不同的应用以获得 `UI Thread`，`SynchronizationContext` 使得我们可以在类库中忽略具体的应用类型，不同的应用类型提供一个具体的 `Concrete`  `SynchronizationContext`，而我们在类库中只需使用 `SynchronizationContext` 即可。[UIKitSynchronizationContext](https://github.com/xamarin/xamarin-macios/blob/master/src/UIKit/UIKitSynchronizationContext.cs)

同步上下文与线程有关，在 `UI` 线程中请求 `SynchronizationContext.Current` 得到如 `UIKitSynchronizationContext` 的上下文对象；如果在工作线程中请求 `SynchronizationContext.Current` 得到 null，表示默认使用 `SynchronizationContext` 的实现。

`SynchronizationContext` 的默认实现与 `UI` 无关，在其调用线程上执行同步委托，在 ThreadPool 中执行异步委托。

`await` 默认会捕获同步上下文，如果是 `GUI` 程序，捕获的是 `UI` 线程，所以 `await` 后的更新 `UI` 界面的操作是合法的；如果是 `Console` 程序，默认捕获的是线程池线程。然而，捕获 `UI` 线程会导致占用主线程资源，因此最好的方式是不捕获 `UI` 线程，使用线程池线程来执行。

``` c#
// 使用线程池线程
await DownloadAsync("http://...").ConfigAwait(false);
```
执行不进行 UI 操作的**所有**异步方法应该使用 `ConfigAwait(false)` 来提升性能，特别是类库中异步方法。 
之所以是**所有**，是因为：

- 当 `await` 的方法立即执行结束时，此时该方法仍执行在 UI 线程上，后续方法依然需要配置 `ConfigAwait(false)` 才能避免捕获同步上下文。
- 每个方法的上下文捕获是独立的，如 A -> B, 在 B 中使用 `ConfigAwait(false)` 后，B 中剩余的方法在线程池线程上执行，而 A 中的剩余方法仍然在 UI 线程中执行。

#### 执行上下文

线程执行时默认会拥有自己的上下文，上下文中保存的线程执行时寄存器中的值和一些环境信息，当启动一个新的线程时，默认执行上下文会流向新线程，以便于新线程能够访问原线程的一些信息，通常我们不会去考虑执行上下文。关于同步上下文与执行上下文 [ExecutionContext vs SynchronizationContext](http://blogs.msdn.com/b/pfxteam/archive/2012/06/15/executioncontext-vs-synchronizationcontext.aspx) 。

### 异步执行 `I/O` 密集型操作

``` c#
using (FileStream fs = new FileStream(filePath, FileMode.Open, FileAccess.Read, FileShare.Read, 8, true)) 
{ 
	await fs.ReadAsync(...);
} 

```

普通的 `I/O` 操作，当线程从 user-mode 到 kernel-mode 读取磁盘文件时，由于读取磁盘文件的操作不涉及到线程，因此线程会被阻塞，直到文件读取完成，线程获取读取结果并恢复执行。而异步的 `I/O` 操作，上面 `true` 参数和 `ReadAsync` 方法结合使用，当读取磁盘文件时，线程不会被阻塞，而是立即返回，如果是线程池线程，则返回线程池，等下执行下一个任务单元。这种方式减少了不必要线程的创建，从而解决了系统资源，提升了性能，特别是在执行数据库访问操作时，大大增加了服务器的可响应性，可扩展性。[Performing IO Bounds Asynchronous Operations](https://www.wintellectnow.com/Videos/Watch/performing-i-o-bound-asynchronous-operations?videoId=performing-i-o-bound-asynchronous-operations) 

### 状态机

编译器会为我们使用 `await` 的代码自动生成状态机的实现。如

```csharp
private static async Task<int> HttpLengthAsync(string uri)
{
    string text = await new HttpClient().GetStringAsync(uri);
    return text.Length;
}
```

经过编译之后会生成类似下面的代码：

```csharp
private static Task<int> HttpLengthAsync(string uri)
{
    // create state machine and initialize it
    var sm = new HttpLengthSM {
        m_builder = AsyncTaskMethodBuilder<int>.create();
        uri = uri
    };
    
    // start running state machine
    sm.m_builder.start(ref sm);
    // return its task to the caller
    return sm.m_builder.Task;
}

private struct HttpLengthSM: IAsyncStateMachine
{
    public string uri, text;  // params & local variable
    public AsyncTaskMethodBuilder<int> m_builder;
    public int m_state = 0;
    private TaskAwaiter<string> m_awaiter;
    
    private void MoveNext()
    {
        int length;
        try
        {
            if(m_state == 0)
            {
                m_awaiter = new HttpClient.GetStringAsync(this.uri).GetAwaiter();
                if(!m_awaiter.IsCompleted)
                {
                    // if not completed synchronously, prepare to call back
                    m_state = 1;
                    m_awaiter.OnCompleted(this.MoveNext);
                    return;
                }
            }
            
            text = m_awaiter.GetResult();  // completed
            length = text.Length;
        }
        catch(Exception ex)
        {
            m_builder.SetException(ex);
        }
        m_builder.SetResult(length);
    }
}
```

`AsyncTaskMethodBuilder<int>` 用于实现可等待的 `Task`，处理异常或返回最终的结果，`TaskAwaiter<string>` 用于实际执行请求并获取请求结果。

**Note**:

[*The zen of async: Best practices for best performance*](https://channel9.msdn.com/events/Build/BUILD2011/TOOL-829T)

> Cache tasks when applicable.  
> Use ConfigureAwait(false) in libraries.  
> Avoid gratuitous use of ExecutionContext.  
> Remove unnecessary locals.  
> Avoid unnecessary awaits.


[*Parallel Programming in .NET Framework 4*](https://blogs.msdn.microsoft.com/csharpfaq/2010/06/01/parallel-programming-in-net-framework-4-getting-started/) / 
[*Why use async await*](https://msdn.microsoft.com/en-us/magazine/hh456403.aspx) / 
[*Asynchronous Programming*](http://msdn.microsoft.com/en-us/library/hh191443(v=vs.110).aspx) / 
[*Scott Hanselman*](http://www.hanselman.com/blog/TheMagicOfUsingAsynchronousMethodsInASPNET45PlusAnImportantGotcha.aspx) / 
[*asp.net*](http://www.asp.net/web-forms/overview/performance-and-caching/using-asynchronous-methods-in-aspnet-45) /
[*Be Aware Of The Synchronization-Context*](http://www.gamlor.info/wordpress/2010/10/c-5-0-async-feature-be-aware-of-the-synchronization-context/) / [*Understanding-SynchronizationContext*](http://www.codeproject.com/Articles/31971/Understanding-SynchronizationContext-Part-I) / 
[*Synchronous and Asynchronous I/O*](http://msdn.microsoft.com/en-us/library/windows/desktop/aa365683(v=vs.85).aspx)
