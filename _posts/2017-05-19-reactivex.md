---
layout: post
title: "ReactiveX"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### 核心思想

`ReactiveX` 的核心思想是**将拉模型**转化为**推模型**。以迭代器为例：

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

上面是实现 `C#` 中迭代器的接口定义，我们需要不断的调用 `MoveNext()` 并根据其返回值决定是否调用 `Current` 属性。这是**拉模型**的实现方式，需要主动去请求并获取数据。

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

**推模型**完全实现了**拉模型**的所有功能，只不过我们不再需要去主动请求并获取数据，而是订阅推送，当新消息出现时，会自动通知我们。这种模型极大地改善了诸如异步回调，并发，异常处理等编程方式，不再因为冗长的嵌套方式而导致[回调地狱](http://callbackhell.com/)，也无需担心数据状态是否被并发所破坏，以及异常的流式处理。`ReactiveX` 的这种特性，也被简单地概括为**异步的数据流**。

### 关键概念

学习 Rx，我们需要从 `IObservable` 和 `Subscribe` 开始，即如何得到一个可观测对象和如何获取通知。

#### 如何得到 IObservable

#### 如何获取通知

### ReactiveUI 应用场景演示

##### 方便取消注册事件，解决内存泄露

如当我们需要在 Xamarin.iOS 中监测 UITextField 文本变化，通常我们可以注册 `EditingChanged` 事件获取通知，但在 Xamarin.iOS 中存在一个问题 **Native 对象间的相互引用会导致内存泄露**，这时，我们就不得不使用弱引用或在页面退出时手动取消注册事件等方式，书写起来会让代码显得很繁琐。ReactiveUI 为我们提供了注册以及取消注册的便捷方式:

```csharp
this.WhenAnyValue(v => v.TextField.Text).Subscribe(x => { }).DisposeWith(bag);
```

bag 类型是 `CompositeDisposable`，通过将不同的订阅添加到 bag 中，可以很方便让我们对资源释放做统一处理。将观察操作放置到 `ViewWillAppear` 方法中，将释放操作 `bag.Clear()` 放置到 `ViewWillDisappear` 中，避免由于循环引用会导致 `Dispose(bool)` 方法不被调用。

> 通过 WhenAnyValue 检测 TextField 文本变化时需注意：手动通过代码改变文本不会触发该通知；可以使用 FromEventPattern 订阅 EditingChanged 事件 + TextField.SendActionForControlEvents(UIControlEvent.EditingChanged) 来实现最终效果。





[*.NET Reactive Framework*](https://www.youtube.com/watch?v=looJcaeboBY) / [*Getting Started with Rx*](https://msdn.microsoft.com/en-us/library/hh242975(v=vs.103).aspx) / [*Subject/Observer is Dual to Iterator*](http://csl.stanford.edu/%7Echristos/pldi2010.fit/meijer.duality.pdf) / 
[*Async JavaScript with Reactive Extensions*](https://www.youtube.com/watch?v=XRYN2xt11Ek)