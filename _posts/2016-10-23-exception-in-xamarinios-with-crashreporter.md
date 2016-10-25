---
layout: post
title: "Exception in Xamarin.iOS with CrashReporter"
description: ""
category: 
tags: [Xamarin.iOS]
---
{% include JB/setup %}


### 异常捕获

当我们在 `Xamarin.iOS` 中使用第三方异常报告组件（如`Bugtags`、`Hockey`、`TestFlight` 等）时。由于这些组件重写了 `signal handler`，当诸如 `SIGBUS`，`SIGSEGE`，`SIGPIPE` 发生时，这些信号会被异常报告组件处理，并导致 `app` 崩溃，在此过程中 `Xamarin.iOS` 无法捕获该异常。

#### 可捕获的异常

当我们手动触发时，该异常在 `Xamarin.iOS` 中可被捕获的，如

``` c#
throw new Exception("测试异常是否被捕获")
```

#### 无法捕获的异常

在代码执行中遇到诸如空引用异常，以及 `Native Code` 中触发的异常会被第三方异常报告组件拦截，从而导致 `Xamarin.iOS` 无法捕获该异常。如

``` c#
string test = null;
test.ToLower();
```

关于 `MonoTouch` 处理空异常的过程如下：
> A null reference exception is actually a SIGSEGV signal at first. Usually the mono runtime handles this and translates it into a nullreference exception, allowing the execution to continue. The problem is that SIGSEGV signals are a very bad thing in ObjC apps (and when it occurs outside of managed code), so any crash reporting solution will report it as a crash (and kill the app) - this happens before MonoTouch gets a chance to handle the SIGSEGV, so there is nothing MonoTouch can do about this.

### 如何解决

正确的处理方式我们需要在启用了异常报告组件后，恢复 `Xamarin.iOS` 对这些信号的捕获和处理。

参考示例：[Demo1](https://github.com/stampsy/hockeyapp-monotouch/blob/master/src/SampleApp/AppDelegate.cs) | [Demo2](https://github.com/DefinitelyBound/xamarin-ios/blob/master/TestFlight/binding/testflight-threadsafe.cs) | [Demo3](https://github.com/GNOME/banshee/blob/stable-2.2/src/Core/Banshee.Core/Banshee.Base/PlatformHacks.cs)

``` c#
enum Signal
{
	SIGBUS = 10,
	SIGSEGV = 11,
	SIGPIPE = 13
}

[DllImport ("libc")]
private static extern int sigaction (Signal sig, IntPtr act, IntPtr oact);

private void EnableCrashReporting ()
{
	IntPtr sigbus = Marshal.AllocHGlobal (512);
	IntPtr sigsegv = Marshal.AllocHGlobal (512);
	IntPtr sigpipe = Marshal.AllocHGlobal (512);

	// Store Mono SIGSEGV and SIGBUS and SIGPIPE handlers
	sigaction (Signal.SIGBUS, IntPtr.Zero, sigbus);
	sigaction (Signal.SIGSEGV, IntPtr.Zero, sigsegv);
	sigaction (Signal.SIGPIPE, IntPtr.Zero, sigpipe);

	// Enable crash reporting libraries


	// Restore Mono SIGSEGV and SIGBUS handlers
	sigaction (Signal.SIGBUS, sigbus, IntPtr.Zero);
	sigaction (Signal.SIGSEGV, sigsegv, IntPtr.Zero);
	sigaction (Signal.SIGPIPE, sigpipe, IntPtr.Zero);

	Marshal.FreeHGlobal (sigbus);
	Marshal.FreeHGlobal (sigsegv);
	Marshal.FreeHGlobal (sigpipe);
}
```

在 `AppDelegate` 中的 `FinishedLaunching` 方法开始出调用 `EnableCrashReporting`。

如果你使用的 `Bugtags`，那么可以通过设置 `Bugtags` 忽略 `Signal` 异常来手动禁用信号量异常的捕获。

### 附

[Unix Signal](https://en.wikipedia.org/wiki/Unix_signal)

[*How to prevent iOS crash reporters from crashing MonoTouch apps?*](http://stackoverflow.com/questions/14499334/how-to-prevent-ios-crash-reporters-from-crashing-monotouch-apps/14499336#14499336)  /
[*Avoiding Common Networking Mistakes*](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/NetworkingOverview/CommonPitfalls/CommonPitfalls.html)