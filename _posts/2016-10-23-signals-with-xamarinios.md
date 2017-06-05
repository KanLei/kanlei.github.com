---
layout: post
title: "Signals with Xamarin.iOS"
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

参考示例：[Demo1](https://github.com/stampsy/hockeyapp-monotouch/blob/master/src/SampleApp/AppDelegate.cs) / [Demo2](https://github.com/DefinitelyBound/xamarin-ios/blob/master/TestFlight/binding/testflight-threadsafe.cs) / [Demo3](https://github.com/GNOME/banshee/blob/stable-2.2/src/Core/Banshee.Core/Banshee.Base/PlatformHacks.cs)

~~~ c#
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
~~~

在 `AppDelegate` 中的 `FinishedLaunching` 方法开始出调用 `EnableCrashReporting`。

如果你使用的 `Bugtags`，那么可以通过设置 `Bugtags` 忽略 `Signal` 异常来手动禁用信号量异常的捕获。

### Signals

#### Signals 简介

Signals 是软中断(software interrupts)，提供了一种处理异步事件的方式。每一个 Signal 都有其对应的名称，名称默认以 **SIG** 开头。

生成 Signal 的方式主要有以下几种：

- 终端生成，比如你在终端上按下 Ctrl+C，会停止当前进程继续执行
- 硬件生成，比如除0指令操作，当硬件检测到该操作时，会首先通知内核(kernel)，然后内核负责生成合适的 Signal 通知当前执行的进程
- kill 命令，我们可以手动指定要中断的进程 ID，并传递指定的 Signal，可通过 `man kill` 查看具体使用方式
- 软件生成，比如当进程写入一个没有读者的管道时，软件通知内核并生成 Signal 通知给当前进程

针对 Signal，我们也可以不同的处理方式：

- 忽略 Signal，但是 SIGKILL 和 SIGSTOP 不能被忽略，因为这是保证终结进程的唯一方式
- 捕获 Signal，我们需要提供处理对应 Signal 的 handler，以便于当 Signal 发生时供内核调用
- 执行默认操作，为了安全起见，通常默认操作会终止当前进程

打开 mac 上的 terminal，输入 `man signal` 可以查看 signal 定义描述以及 mac 支持 31 中不同的 signal。

#### Signals 使用方式

新建一个 C 项目，引入 \<signal.h> 头文件后，我们就可以定义对应的 Signals Handler 了。

> Signals 的实现实际定义在 \<sys/signal.h> 头文件中，由于 Signal 同时供内核和用户级程序使用，所以通常做法是，实现被放置在内核的头文件中，被用户级头文件所引用。

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

void sig_handler(int);

int main(int argc, const char * argv[]) {
    printf("Hello, World!\n");
    
    if (signal(SIGUSR1, sig_handler) == SIG_ERR)
        printf("can't catch SIGUSR1\n");
    if (signal(SIGUSR2, sig_handler) == SIG_ERR)
        printf("can't catch SIGUSR2\n");
    for( ; ; )
        pause();
    
    return 0;
}

void sig_handler(int signo)
{
    if (signo == SIGUSR1)
        printf("received SIGUSR1\n");
    else if (signo == SIGUSR2)
        printf("received SIGUSR2\n");
    else
        printf("received undefined signal\n");
    
}
```

首先我们编译上面的 main.c 文件，`gcc main.c` 默认会在当前目录下生成 a.out 文件，接着我们执行该文件：

```bash
$ ./a.out &
[1] 6154
Hello, World!
$ kill -USR1 6154
received SIGUSR1
$ kill -USR2 6154
received SIGUSR2
$ kill 6154
[1]+  Terminated: 15          ./a.out
$ 
```

通过执行上面的命令，可以发现，我们手动处理了 `SIGUSR1` 和 `SIGUSR2`，所以终端输出对应的处理信息。

[*Unix Signal*](https://en.wikipedia.org/wiki/Unix_signal) /
[*Handling unhandled exceptions and signals*](https://www.cocoawithlove.com/2010/05/handling-unhandled-exceptions-and.html) / 
[*How to prevent iOS crash reporters from crashing MonoTouch apps?*](http://stackoverflow.com/questions/14499334/how-to-prevent-ios-crash-reporters-from-crashing-monotouch-apps/14499336#14499336)  /
[*Avoiding Common Networking Mistakes*](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/NetworkingOverview/CommonPitfalls/CommonPitfalls.html)