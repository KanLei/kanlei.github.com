---
layout: post
title: "ReactiveX"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### 核心思想

`ReactiveX` 的核心思想是将拉模型转化为推模型。以迭代器为例：

```csharp
interface IEnumerable
{
	IEnumerator GetEnumerator();
}

interface IEnumerator: IDisposable
{
	bool MoveNext();
	object Current { get; }
}

```

上面是我们实现 `C#` 中迭代器的接口定义，我们需要不断的调用 `MoveNext()` 并根据其返回值决定是否调用 `Current` 属性。这是**拉模型**的实现方式，我们需要主动去请求并获取数据。

其对应的**推模型**接口定义如下：

```csharp
interface IObservable
{
	IDisposable Subscribe(IObserver observer);
}

interface IObserver
{
	void OnNext(object value);
	void OnCompletion();
	void OnError(Exception ex);
}
```

**推模型**完全实现了**拉模型**的所有功能，只不过我们不再需要去主动请求并获取数据，而是我们只需订阅推送，当新消息出现时，会自动通知我们。这种模型使我们极大地改善了诸如异步回调，并发，异常处理等编程方式，我们无需再因为冗长的嵌套方式而导致[回调地狱](http://callbackhell.com/)，也无需再担心数据状态是否被并发所破坏，以及因没有及时处理的异常导致程序崩溃。

// 未完待续


[.NET Reactive Framework](https://www.youtube.com/watch?v=looJcaeboBY)