---
layout: post
title: "Custom View in iOS"
description: ""
category: 
tags: [Xamarin, iOS]
---
{% include JB/setup %}

#### 视图复用的自定义方式有两种

* class
* class + xib

#### class 方式

通过代码直接构造实例，或者在布局文件上通过 Custom Class 属性设置

#### class + xib 方式

采用这种方式，需要我们手动关联 class 和 xib 视图，如

```csharp
var lbl = new XLabel ();
UINib nib = UINib.FromName ("XLabel", NSBundle.MainBundle);
NSObject [] views = nib.Instantiate (lbl, null);
var niblbl = views [0] as UILabel;
niblbl.Frame = lbl.Bounds;
lbl.AddSubview (niblbl);
```

1. 首先构造一个 `XLabel` 实例
2. 构造对应的 `nib` 实例
3. 将 `nib` 与 `lbl` 关联，如设置 `outlets`
4. 获取 `views` 的顶层视图 `niblbl` 并设置其 `Frame`
5. 添加 `niblbl` 到 `lbl` 子视图中

通过上面 5 步，我们就可以复用该视图组件了。

为了避免对上面的代码重复的进行复制黏贴，通过我们会将其放置到 `XLabel` 的构造函数中，当实例化一个 `XLabel` 时，关联视图和类的过程也会随之完成。

```csharp
[Register (nameof (XLabel))]
public class XLabel : UILabel
{
    public XLabel (CGRect frame) : base (frame)
	{
	   SetupSubviews ();
    }

    [Export ("initWithCoder:")]
    public XLabel (NSCoder coder) : base (coder)
    {
        SetupSubviews ();
    }

    private void SetupSubviews ()
    {
        var lbl = new XLabel ();
        UINib nib = UINib.FromName ("XLabel", NSBundle.MainBundle);
        NSObject [] views = nib.Instantiate (lbl, null);
        var niblbl = views [0] as UILabel;
        niblbl.Frame = lbl.Bounds;
        lbl.AddSubview (niblbl);
    }
}
```

当我们手动创建 `XLabel` 时会通过 `CGRect` 构造函数来创建，或者通过定义视图组件的 `Custom Class` 方式，最终通过调用 `initWithCoder:` 的方式来创建，两种方式最终都调用了 `SetupSubviews()` 方法完成视图和类的关联。使用上面这种方式，需要注意的一点是，我们在定义 `xib` 时，要通过设置 **File's Owner** 的 Custom Class，而不能直接设置 **UILabel** 的 Custom Class，因为 `xib` 的构造过程默认会调用视图上组件对应的 Custom Class 的 `initWithCoder` 方法，而 `initWithCoder` 方法中我们又定义了加载和构造当前 `xib` 文件，从而导致 `initWithCoder` 方法被递归调用，直到栈溢出为止。[*Why you shouldn't set top-level view's custom class*](https://guides.codepath.com/ios/Custom-Views#why-you-shouldn-t-set-top-level-view-s-custom-class)

除了 `initWithCoder`，`Xamarin.iOS` 还提供了另外一个构造函数，该函数包含了一个 `IntPtr` 类型的参数：

```csharp
public XLabel (IntPtr handle) : base (handle)
{
}
```

关于 `IntPtr` 的解释如下
> The IntPtr constructor is required for Objective-C to create instances of managed types used in storyboards/xibs, and if you're missing the IntPtr constructor you'll get **Could not find an existing managed instance for this object, nor was it possible to create a new managed instance** exception.

如果我们同时提供了 `initWithCoder:` 和 `IntPtr` 的构造方式，默认 `initWithCoder:` 会被调用，否则会调用单独提供的那种方式。

关于使用 `initWithCoder:` 方式的[兼容性建议](http://kanlei.github.io/article/2016/06/25/improve-ios-9-compatibility)

[*Nib*](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/LoadingResources/CocoaNibs/CocoaNibs.html)