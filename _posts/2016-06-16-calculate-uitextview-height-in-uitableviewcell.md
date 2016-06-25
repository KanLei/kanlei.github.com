---
layout: post
title: "Calculate UITextView Height in UITableViewCell"
description: ""
category: "article"
tags: [Xamarin.iOS, UITextView]
---
{% include JB/setup %}

## 需求简介

通常我们需要在 `UITableViewCell` 中用到 `UITextView` 显示多行文本内容，但是由于 `UITextView` 文本内容并不是固定不变的，所有我们需要动态获取其内容，并计算其高度，来决定 `UITableViewCell` 的最终高度

## 解决方案

``` c#
var size = TextView.SizeThatFits(new CGSize(width, nfloat.MaxValue));
```

注意，`width` 可通过 `UIScreen.MainScreen.Bounds.Width` 计算得到，如果在 `GetHeightForRow` 中提前计算，则当前 `TableCell` 的 `ContentView.Frame`, `ContentView.Bounds`, `TextView.Frame` 以及 `TextView.Bounds` 不能反映出实际显示在屏幕中的尺寸。