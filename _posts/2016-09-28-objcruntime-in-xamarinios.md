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

当我们检测某一个类是否是继承自 `NSObject`，或用于检测该类型是否存在时，我们可以使用 `Class.GetHandle` 方法。

比如在 iOS 中，我们可以使用如下方式让 `NavigationBar` 背景透明。

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
