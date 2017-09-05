---
layout: post
title: "iOS APP 中使用自定义字体"
description: ""
category: 
tags: []
---
{% include JB/setup %}

开发应用过程中，经常会使用一些特殊样式的字体，如果这种字体 iOS 本身又不支持的话，我们只能通过手动添加字体文件来解决，在添加的过程中需要注意一些事项。

以我们需要在 APP 中使用 **DIN Alternate** 字体为例。首先需要找到[这款字体](https://github.com/msmuenchen/skynetrss/tree/master/css/fonts)，下载 **din-alternate-normal.ttf** 字体后，接着打开字体并安装，**注意字体文件名可能与真实字体名不同，因此需要查看字体真实的文件名。**如这款字体在 Mac 上显示名称为 **DIN Alternate**。

打开 Info.plist 添加新配置选项 **Fonts provided by application**，字体名称为 **DIN Alternate.ttf**。

通过以下代码查看字体名称：

```csharp
string[] families = UIFont.FamilyNames;
foreach (var family in families)
{
    Console.WriteLine($"---{family}---");
    string[] fonts = UIFont.FontNamesForFamilyName(family);
    foreach (var font in fonts)
    {
        Console.WriteLine(font);
    }
}
```

最后使用 `UIFont.FromName("DIN Alternate", 20)` 引用字体。
