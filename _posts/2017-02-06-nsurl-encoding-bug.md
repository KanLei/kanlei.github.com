---
layout: post
title: "记一次 NSUrl 中文字符导致的 bug"
description: ""
category:
tags: [Xamarin.iOS]
---
{% include JB/setup %}

## 前提简介

App 中有一个提交页面，当点击提交按钮操作后导致崩溃。

经测试，出现条件如下：

1. 只有特定账户才会复现
2. 相同账号和环境，只有在 adhoc 和 appstore 模式下才会复现

由条件1可推断大致由于账户不同导致数据不同，最终导致部分数据空异常；由条件2可推断跟数据无关，与条件1互斥。这就尴尬了-_-。因此只能通过反复注释代码和重新打包进行测试，却依然没能找出原因。
最后，通过对提交前后的操作进行梳理和排查，发现在提交后服务端立即向该设备推送了一条消息，而且是只有在发布模式才推送，因此，暂且断定消息推送才是罪魁祸首，立即注释后端推送代码，果然不再崩溃。但为什么有些账号不会崩溃呢？经过进一步查看手机中的崩溃日志，发现导致崩溃的原因是构造 `NSUrl` 失败，原来推送中带有额外的参数，而部分账户中参数包含中文字符，在接收到推送后，默认会提取附带参数并用来构造 `NSUrl` ，原来是由于 `NSUrl` 接收到不匹配的字符抛出异常导致 app 崩溃。

经过测试发现，在 `Xamarin.iOS` 中，`new NSUrl(...)` 包含中文字符时默认会抛出异常；而 `NSUrl.FromString(...)` 包含中文字符时，默认会返回空，并不抛出异常。

解决方式，构造 `NSString` 实例，通过实例方法 `CreateStringByAddingPercentEscapes (NSStringEncoding.UTF8)` 转义中文字符为 UT8 编码，构造 `NSUrl`，恢复中文字符使用实例方法 `CreateStringByReplacingPercentEscapes (NSStringEncoding.UTF8)`。

[*RFC 3986*](https://tools.ietf.org/html/rfc3986)  
[*NSURL*](https://developer.apple.com/reference/foundation/nsurl)