---
layout: post
title: "C# Communicate With Objective C"
description: ""
category: "article"
tags: [Xamarin.iOS]
---
{% include JB/setup %}

## 消息发送

在 `Objective-C` 中方法调用最终会在编译阶段转化为 `objc_msgSend` 来执行。
`[receiver message]` 会转化为 `objc_msgSend(receiver, selector)`，`receiver` 表示接收消息的对象，`selector` 表示消息的名称（OC中方法调用称为消息发送）。如果带有参数，则将参数添加到 `selector` 之后即可。

## C# 调用 OC

从 `objc_msgSend` 中我们可以想象，如果如果我们在 `C#` 代码中直接通过这种方式来调用 `OC` 代码，是不是就可以与 `OC` 进行通信了吗？是的。

1. 首先我们需要先声明要方法调用的方式


    [DllImport("/usr/lib/libobjc.dylib", EntryPoint = "objc_msgSend")]
    public static extern void ShowAlert_objc_msgSend(InPtr receiver, IntPtr selector);

第一个参数表示当前对象实例所在的内存地址，第二个参数表示方法所在的内存地址。关于 `IntPtr` 更多内容，可以参考托管代码与非托管代码的互操作。

2.接着，我们定义要调用的方法，这里我们没有直接调用 `OC` 中的方法，而是通过 `Export` 的方式，将自定义方法暴露到 `OC` 的环境中。

    [Export("ShowAlert")]
    private void ShowAlert()
    {
        var alert = new UIAlertView("Alert", "Message", null, "OK", null);
        alert.Show();
    }

3.最后我们执行方法的调用

    ShowAlert_objc_msgSend(this.Handle, ObjCRuntime.Selector.GetHandle("ShowAlert"));

`C#` 调用 `OC` 的声明有多种方式，具体可参考 `ObjCRuntime.Messaging` 类中声明的各种方法调用的格式。

[Objective-C Messaging](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtHowMessagingWorks.html) / 
[Objective-C Selector](https://developer.xamarin.com/guides/ios/advanced_topics/objective-c_selectors/) /
[MSDN IntPtr](https://msdn.microsoft.com/en-us/library/system.intptr(v=vs.110).aspx)