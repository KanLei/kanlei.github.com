---
layout: post
title: "ObjCRuntime in Xamarin.iOS"
description: ""
category: 
tags: [Xamarin.iOS]
---
{% include JB/setup %}

## Class

> [Class is managed representation for an Objective-C class](https://developer.xamarin.com/api/type/ObjCRuntime.Class/)

当我们需要检测某一个类是否存在于 `Objective C` 运行环境中时，可以使用 `Class.GetHandle` 方法，该方法等价于 `Objective C` 中的 `NSClassFromString("")` 方法。

> [GetHandle("")](https://developer.xamarin.com/api/member/ObjCRuntime.Selector.GetHandle/p/System.String/): IntPtr.Zero if the class does not exist or has not been registered with the Objective-C runtime, or a token pointing to the Objective-C Class object otherwise.

比如在 iOS 中，我们可以使用如下方式获取 `NavigationBar` 中的背景视图并隐藏。

``` c#
IntPtr handle = Class.GetHandle (@"_UINavigationBarBackground");
if (handle == IntPtr.Zero) // >= iOS 10.0
{
    return;
}

if (superView.IsKindOfClass (new Class (@"_UINavigationBarBackground")))
{
    // 移除分割线
    foreach (UIView view in superView.Subviews)
    {
        if (view is UIImageView)
        {
            view.Hidden = hide;
            return;
        }
    }
}
```
