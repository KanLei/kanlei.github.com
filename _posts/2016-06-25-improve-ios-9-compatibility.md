---
layout: post
title: "Improve iOS 9 Compatibility"
description: ""
category: "article" 
tags: [Xamarin.iOS]
---
{% include JB/setup %}


在兼容 iOS 9 时编写代码需注意一些规则。

## 以纯代码的方式构造

#### UITableViewCell

当我们使用 `RegisterClassForCellReuse` 注册 `Cell`，用 `DequeueReusableCell` 获取 `Cell`， 如果此时 `Cell` 需要构造则默认会调用 `initWithStyle:reuseIdentifier:` 方法创建

``` c#
[Export ("initWithStyle:reuseIdentifier:")]
public TestTableViewCell (UITableViewCellStyle style, NSString reuseIdentifier)
	 : base (style, reuseIdentifier)
{

}
```

#### UICollectionViewCell

当我们使用 `RegisterClassForCellReuse` 注册 `Cell`，用 `DequeueReusableCell` 获取 `Cell`， 如果此时 `Cell` 需要构造则默认会调用 `initWithFrame:` 方法创建

``` c#
[Export ("initWithFrame:")]
public TestCollectionViewCell (CGRect frame)
	: base (frame)
{

}
```

## 以 xib + cs 构造

1. 当我们使用 `xib` 文件布局构造 `UITableViewCell` 或 `UICollectionViewCell` 
2. 当自定义继承 `UIView` 的子类，并通过布局视图的 `class` 方式引用时

**需添加 `initWithCoder:` 方法**

``` c#
[Export ("initWithCoder:")]
public TestView (NSCoder coder)
	:base(coder)
{
			
}
```

> **Reason**: The initWithCoder: constructor is the one called when loading a view from an Interface Builder Xib file. If this constructor is not exported unmanaged code can’t call our managed version of it. Previously (eg. in iOS 8) the IntPtr constructor was invoked to initialize view.

[iOS 9 Compatibility](https://developer.xamarin.com/guides/ios/platform_features/ios9/#compat)