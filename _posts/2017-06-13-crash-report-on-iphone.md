---
layout: post
title: "Crash Report on iPhone"
description: ""
category: iOS
tags: []
---
{% include JB/setup %}

### 崩溃日志

当 iPhone 上的 app 发生崩溃时，默认会生成一份以 .crash 为后缀的崩溃日志文件，用于记录崩溃的原因。我们可以通过分析该崩溃日志，查找导致崩溃的原因，从而修复崩溃。

### 日志获取

检索崩溃日志主要分为以下两种方式：

1. 拥有存储崩溃日志的 iPhone
2. 不拥有存储崩溃日志的 iPhone

第 1 种情况，当我们能够获取崩溃日志的手机时，将手机通过 USB 连接到 Mac，打开 Xcode > Window > Devices > View Device Logs，Xcode 会自动同步手机上的 crash 文件到当前 Mac，并自动符号化崩溃日志(关于如何符号化日志文件参见日志符号化一节)。我们也可以通过 iPhone 查看崩溃日志文件，设置 > 隐私 > 分析 > 分析数据，即可查看崩溃日志列表，点击具体的日志文件，可以查看崩溃的详细原因记录。

第 2 种情况，当我们无法获取有崩溃日志的手机时，如果手机的 **隐私 > 分析 > 共享 iPhone 分析** 选项开启，崩溃日志会自动上传到 AppStore，我们可以在 Xcode > Organizer > Crashes 一栏中查看用户手机上的崩溃日志(低于 iOS 8.3 需要到 [iTunes Connect](https://itunesconnect.apple.com/) 查看)。如果以上选项未开启，那么只能让用户将崩溃日志手动发送给我们了。

操作系统与日志路径：

macOS
> ~/Library/Logs/CrashReporter/MobileDevice/<DEVICE_NAME>

Window XP
> C:\Documents and Settings\<USERNAME>\Application Data\Apple Computer\Logs\CrashReporter\MobileDevice\<DEVICE_NAME>
 
Windows Vista or 7
> C:\Users\<USERNAME>\AppData\Roaming\Apple Computer\Logs\CrashReporter\MobileDevice\<DEVICE_NAME>


### 日志分析

当我们获取到崩溃日志之后，就要进一步对崩溃日志进行分析，查找崩溃的原因。新建一个 CrashDemo 项目并触发以下崩溃，**未符号化**：

```
Incident Identifier: 164C551F-C24E-424E-87F6-344C1C9B90D5
CrashReporter Key:   06e2818d3b4ae7b9583b381fbb5755b5cae94c3d
Hardware Model:      iPhone9,1
Process:             CrashDemo [1156]
Path:                /private/var/containers/Bundle/Application/D7DE1582-D15C-451D-B841-5E6B8FDB80A4/CrashDemo.app/CrashDemo
Identifier:          com.companyname.crashdemo
Version:             1.0 (1.0)
Code Type:           ARM-64 (Native)
Role:                Foreground
Parent Process:      launchd [1]
Coalition:           com.companyname.crashdemo [1038]


Date/Time:           2017-06-13 15:28:13.7694 +0800
Launch Time:         2017-06-13 15:28:13.5834 +0800
OS Version:          iPhone OS 10.3.2 (14F89)
Report Version:      104

Exception Type:  EXC_CRASH (SIGABRT)
Exception Codes: 0x0000000000000000, 0x0000000000000000
Exception Note:  EXC_CORPSE_NOTIFY
Triggered by Thread:  0

Application Specific Information:
abort() called

Filtered syslog:
None found

Thread 0 name:  tid_403  Dispatch queue: com.apple.main-thread
Thread 0 Crashed:
0   libsystem_kernel.dylib      0x0000000185871014 0x185852000 + 126996
1   libsystem_pthread.dylib     0x000000018593b264 0x185936000 + 21092
2   libsystem_c.dylib           0x00000001857e59c4 0x185784000 + 399812
3   CrashDemo                   0x0000000100254924 0x10002c000 + 2263332
4   libsystem_platform.dylib    0x000000018593531c 0x18592f000 + 25372
5   libsystem_pthread.dylib     0x000000018593b264 0x185936000 + 21092
6   libsystem_c.dylib           0x00000001857e59c4 0x185784000 + 399812
7   CrashDemo                   0x0000000100345e24 0x10002c000 + 3251748
8   CrashDemo                   0x000000010028d05c 0x10002c000 + 2494556
9   CrashDemo                   0x000000010025453c 0x10002c000 + 2262332
10  CrashDemo                   0x0000000100253560 0x10002c000 + 2258272
11  CrashDemo                   0x000000010024bed8 0x10002c000 + 2227928
12  CrashDemo                   0x0000000100198dac 0x10002c000 + 1494444
13  CrashDemo                   0x0000000100030178 0x10002c000 + 16760
14  CrashDemo                   0x0000000100177b44 0x10002c000 + 1358660
15  CrashDemo                   0x0000000100261b64 0x10002c000 + 2317156
16  CrashDemo                   0x00000001002c17bc 0x10002c000 + 2709436
17  CrashDemo                   0x00000001002c1718 0x10002c000 + 2709272
18  CrashDemo                   0x0000000100220ec8 0x10002c000 + 2051784
19  CrashDemo                   0x0000000100220cc8 0x10002c000 + 2051272
20  UIKit                       0x000000018ca19204 0x18c998000 + 528900
21  UIKit                       0x000000018cc25738 0x18c998000 + 2676536
22  UIKit                       0x000000018cc2b1e0 0x18c998000 + 2699744
23  UIKit                       0x000000018cc3fd18 0x18c998000 + 2784536
24  UIKit                       0x000000018cc28474 0x18c998000 + 2688116
25  FrontBoardServices          0x000000018841f884 0x1883e5000 + 239748
26  FrontBoardServices          0x000000018841f6f0 0x1883e5000 + 239344
27  FrontBoardServices          0x000000018841faa0 0x1883e5000 + 240288
28  CoreFoundation              0x000000018682542c 0x18674a000 + 898092
29  CoreFoundation              0x0000000186824d9c 0x18674a000 + 896412
30  CoreFoundation              0x00000001868229a8 0x18674a000 + 887208
31  CoreFoundation              0x0000000186752da4 0x18674a000 + 36260
32  UIKit                       0x000000018ca12384 0x18c998000 + 500612
33  UIKit                       0x000000018ca0d058 0x18c998000 + 479320
34  CrashDemo                   0x0000000100204594 0x10002c000 + 1934740
35  CrashDemo                   0x00000001001e986c 0x10002c000 + 1824876
36  CrashDemo                   0x00000001001e982c 0x10002c000 + 1824812
37  CrashDemo                   0x0000000100030044 0x10002c000 + 16452
38  CrashDemo                   0x0000000100177b44 0x10002c000 + 1358660
39  CrashDemo                   0x0000000100261b64 0x10002c000 + 2317156
40  CrashDemo                   0x00000001002c17bc 0x10002c000 + 2709436
41  CrashDemo                   0x00000001002c4240 0x10002c000 + 2720320
42  CrashDemo                   0x000000010024b6e8 0x10002c000 + 2225896
43  CrashDemo                   0x0000000100349a3c 0x10002c000 + 3267132
44  CrashDemo                   0x000000010022338c 0x10002c000 + 2061196
45  libdyld.dylib               0x000000018576159c 0x18575d000 + 17820

// Thread ...

Thread 0 crashed with ARM Thread State (64-bit):
    x0: 0x0000000000000000   x1: 0x0000000000000000   x2: 0x0000000000000000   x3: 0x0000000000000022
    x4: 0x000000000000001b   x5: 0x000000016fdced90   x6: 0x0000000000000038   x7: 0xffffffffffffffec
    x8: 0x0000000008000000   x9: 0x0000000004000000  x10: 0x0000000000000000  x11: 0x0000000000003fff
   x12: 0x0000000100704000  x13: 0x0000000000009d62  x14: 0x0000000010000000  x15: 0x0000000000009d62
   x16: 0x0000000000000148  x17: 0x0000000186771c10  x18: 0x0000000000000000  x19: 0x0000000000000006
   x20: 0x00000001ac9d8b40  x21: 0x000000010039dfdd  x22: 0x000000010039e02d  x23: 0x000000016fdcf178
   x24: 0x0000000000000000  x25: 0x000000010108b558  x26: 0x0000000000010001  x27: 0x00000001006008d0
   x28: 0x0000000100813408   fp: 0x000000016fdcf110   lr: 0x000000018593b264
    sp: 0x000000016fdcf0f0   pc: 0x0000000185871014 cpsr: 0x00000000

Binary Images:
0x10002c000 - 0x1003bbfff CrashDemo arm64  <03ee425a6c0830ae8e7a3cf03462c168> /var/containers/Bundle/Application/D7DE1582-D15C-451D-B841-5E6B8FDB80A4/CrashDemo.app/CrashDemo
0x1004d8000 - 0x10050bfff dyld arm64  <a3339f99c2ea39d8beb70b8ff2e84061> /usr/lib/dyld
0x185258000 - 0x185259fff libSystem.B.dylib arm64  <2e9654eb84903bd7aee0815fd9d27591> /usr/lib/libSystem.B.dylib
0x18525a000 - 0x1852affff libc++.1.dylib arm64  <da0f6a86db853140b2d79e3b36f28795> /usr/lib/libc++.1.dylib

// ...

EOF

```

**进程信息**:
>
**Incident Identifier**: 是崩溃日志的唯一标识符，每个崩溃日志文件都不同。  
**CrashReporter Key**: 设备唯一标识，但该标识是匿名的，不对应真实的设备标识。我们可以根据该标识来分析崩溃覆盖的设备范围。  
**Hardware Model**: 硬件设备，可以用于分析当前崩溃覆盖的设备类型。  
**Process**: 进程名和 ID。

**基本信息**:
>
**Date/Time**: 崩溃时间。  
**Launch Time**: 启动时间。  
**OS Version**: 操作系统版本，可以通过分析版本来确定是否使用不兼容 API 导致。

**异常**:
>
**Exception Type**: 区分异常类型，如内存访问，程序错误等。  
**Triggered by Thread**: 指示异常发生的线程 ID，通常能指示异常发生的正确线程，ABRT 异常除外。

更多 iOS 中定义的 Exception Type 可以在 `<mach/exception_types.h>` 中查看。


**方法调用栈**:
>
```
1 2                      3                  4
0 libsystem_kernel.dylib 0x0000000185871014 0x185852000 + 126996
```
>
第 1 列表示帧号(frame number)，第 2 列表示二进制的文件名，第 3 列表示方法的调用地址，第 4 列表示文件名+行号。

**寄存器值**:
>
[ARMv6 Function Calling Conventions](https://developer.apple.com/library/content/documentation/Xcode/Conceptual/iPhoneOSABIReference/Articles/ARMv6FunctionCallingConventions.html)

**二进制映像**:
>
展示崩溃时加载的所有二进制文件。  
第一行中的 <03ee425a6c0830ae8e7a3cf03462c168> 表示的是唯一标识 UUID，如果需要匹配对应的 dSYM 文件，需要首先将其按 (XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX) 进行格式化，即 (03EE425A-6C08-30AE-8E7A-3CF03462C168)

### 日志符号化

上一节展示了未符号化的崩溃日志信息，未符号化的信息中我们只能获取 16 进制的方法地址，文件名和行号，而无法获取用于分析方法的调用顺序、崩溃发生的位置等信息的可读性符号，因此我们需要将这些 16 进制表示的信息转化为对应的方法地址，文件名和行号，这个过程就叫做符号化(symbolization)。

#### 如何符号化

符号化需要 2 个文件:

1. crash report 文件
2. dSYM 文件

从检索日志一节中，我们已经知道了如何获取 crash report 文件，现在我们只需要再获取 dSYM 文件即可。

当程序由源代码经 Xcode 在 Realse 模式下编译后会生成 .dSYM 和 .app 二进制文件，两个文件中各包含一个 UUID，用于唯一标识和匹配，执行归档(archive)操作后，Xcode 会将上面两个文件打包并放置到 developer > xcode > archives 目录下，以便于后续符号化的时候访问。

在提交 app 到 iTunes Connect 时，我们可以选择是否上传 dSYM，如果我们未勾选，则需要将获取到的崩溃日志通过 Xcode 查看，此时 Xcode 会访问 archives 目录，自动符号化当前崩溃日志；如果我们勾选了，当用户手机开启**共享 iPhone 分析**后，日志会自动发送给 Apple 并完成符号化，我们可以通过 Xcode > Origanizer > Crashes 同步符号化之后的日志文件或通过 iTunes Connect 访问。

**Notice**: 如果我们启用了 Bitcode，编译生成的是中间代码 Bitcode，而不再是最终的机器码，这种编译方式为了 Apple 后续能对提交到 AppStore 上的 App 进行优化而无需重复提交 App。但这种编译方式会导致本地生成的 dSYM 只针对 Bitcode 有用，无法再用于符号化包含机器码的崩溃日志，此时，Apple 为我们提供了两种方式获取最终编译为机器码后的 dSYM，从 Xcode 提交界面中下载，或者从 iTunes Connect 下载。

符号化后的调用栈如下:

```
...
Thread 0 name:  tid_403  Dispatch queue: com.apple.main-thread
Thread 0 Crashed:
0   libsystem_kernel.dylib      0x0000000185871014 __pthread_kill + 8
1   libsystem_pthread.dylib     0x000000018593b264 pthread_kill + 112
2   libsystem_c.dylib           0x00000001857e59c4 abort + 140
3   CrashDemo                   0x0000000100250924 mono_handle_native_crash (mini-exceptions.c:2560)
4   libsystem_platform.dylib    0x000000018593531c _sigtramp + 52
5   libsystem_pthread.dylib     0x000000018593b264 pthread_kill + 112
6   libsystem_c.dylib           0x00000001857e59c4 abort + 140
7   CrashDemo                   0x0000000100341e24 xamarin_printf (runtime.m:2167)
8   CrashDemo                   0x000000010028905c mono_invoke_unhandled_exception_hook (exception.c:1120)
9   CrashDemo                   0x000000010025053c mono_handle_exception_internal (mini-exceptions.c:1893)
10  CrashDemo                   0x000000010024f560 mono_handle_exception (mini-exceptions.c:2126)
11  CrashDemo                   0x0000000100247ed8 mono_arm_throw_exception (exceptions-arm64.c:410)
12  CrashDemo                   0x0000000100194dac throw_corlib_exception + 172
13  CrashDemo                   0x000000010002c178 0x100028000 + 16760
14  CrashDemo                   0x0000000100173b44 wrapper_runtime_invoke_object_runtime_invoke_dynamic_intptr_intptr_intptr_intptr + 244
15  CrashDemo                   0x000000010025db64 mono_jit_runtime_invoke (mini-runtime.c:2510)
16  CrashDemo                   0x00000001002bd7bc do_runtime_invoke (object.c:2860)
17  CrashDemo                   0x00000001002bd718 mono_runtime_invoke (object.c:3018)
18  CrashDemo                   0x000000010021cec8 native_to_managed_trampoline_5(objc_object*, objc_selector*, _MonoMethod**, UIApplication*, NSDictionary*, unsigned int) (registrar.m:207)
19  CrashDemo                   0x000000010021ccc8 -[AppDelegate application:didFinishLaunchingWithOptions:] (registrar.m:831)
20  UIKit                       0x000000018ca19204 -[UIApplication _handleDelegateCallbacksWithOptions:isSuspended:restoreState:] + 380
21  UIKit                       0x000000018cc25738 -[UIApplication _callInitializationDelegatesForMainScene:transitionContext:] + 3452
22  UIKit                       0x000000018cc2b1e0 -[UIApplication _runWithMainScene:transitionContext:completion:] + 1684
23  UIKit                       0x000000018cc3fd18 __84-[UIApplication _handleApplicationActivationWithScene:transitionContext:completion:]_block_invoke.3151 + 48
24  UIKit                       0x000000018cc28474 -[UIApplication workspaceDidEndTransaction:] + 168
25  FrontBoardServices          0x000000018841f884 __FBSSERIALQUEUE_IS_CALLING_OUT_TO_A_BLOCK__ + 36
26  FrontBoardServices          0x000000018841f6f0 -[FBSSerialQueue _performNext] + 176
27  FrontBoardServices          0x000000018841faa0 -[FBSSerialQueue _performNextFromRunLoopSource] + 56
28  CoreFoundation              0x000000018682542c __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__ + 24
29  CoreFoundation              0x0000000186824d9c __CFRunLoopDoSources0 + 540
30  CoreFoundation              0x00000001868229a8 __CFRunLoopRun + 744
31  CoreFoundation              0x0000000186752da4 CFRunLoopRunSpecific + 424
32  UIKit                       0x000000018ca12384 -[UIApplication _run] + 652
33  UIKit                       0x000000018ca0d058 UIApplicationMain + 208
34  CrashDemo                   0x0000000100200594 wrapper_managed_to_native_UIKit_UIApplication_UIApplicationMain_int_string___intptr_intptr (/<unknown>:1)
35  CrashDemo                   0x00000001001e586c UIKit_UIApplication_Main_string___intptr_intptr (UIApplication.cs:79)
36  CrashDemo                   0x00000001001e582c UIKit_UIApplication_Main_string___string_string (UIApplication.cs:63)
37  CrashDemo                   0x000000010002c044 CrashDemo_Application_Main_string__ (Main.cs:13)
38  CrashDemo                   0x0000000100173b44 wrapper_runtime_invoke_object_runtime_invoke_dynamic_intptr_intptr_intptr_intptr + 244
39  CrashDemo                   0x000000010025db64 mono_jit_runtime_invoke (mini-runtime.c:2510)
40  CrashDemo                   0x00000001002bd7bc do_runtime_invoke (object.c:2860)
41  CrashDemo                   0x00000001002c0240 do_exec_main_checked (object.c:4681)
42  CrashDemo                   0x00000001002476e8 mono_jit_exec (driver.g.c:1037)
43  CrashDemo                   0x0000000100345a3c xamarin_main (monotouch-main.m:480)
44  CrashDemo                   0x000000010021f38c main (main.m:41)
45  libdyld.dylib               0x000000018576159c start + 4

...
```

##### 手动符号化

* symbolicatecrash
* atos

symbolicatecrash 所在位置 */Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash*


[dSYM](https://stackoverflow.com/questions/22460058/how-is-a-dsym-file-created)

### 崩溃原因

##### 1.Watchdog timeout

Launch/Resume/Suspend/Quit 超时

超时日志的 Exception Codes 是 0x8badf00d

##### 2.User force-quit

无响应强制退出

强制退出的 Exception Codes 是 0xdeadfa11

##### 3.Low Memory termination

该日志信息与其它异常信息不同，主要由{进程信息、页使用情况、当前所有进程占用页数}组成。

**注意**: 该崩溃无法直接捕获。

当设备检测到低内存时，会发送 low memory 通知给运行中进程，如果内存仍然不够使用，系统会 kill 后台进程，如果内存依然不够使用，系统会 kill 当前使用的进程。

1 page = 4kb

[Out of Memory Terminations](https://docs.fabric.io/apple/crashlytics/OOMs.html)  
[Reducing FOOMs in the Facebook iOS app](https://code.facebook.com/posts/1146930688654547/reducing-fooms-in-the-facebook-ios-app/)

##### 4.Bugs

EXC\_BAD\_ACCESS(SIGSEGV)
> 试图访问不拥有的内存地址，如多次释放同一个对象

EXC\_BAD\_ACCESS(SIGBUS)
> 如解空引用
 
EXC\_CRASH(SIGABRT)
> 应用要求内核终止当前进程，内核收到通知后，kill 当前应用，如抛出异常。该异常日志中，通常需要寻找调用 abort 方法的进程，而非异常描述中的进程


### 三方崩溃报告组件

崩溃报告组件通常是拦截程序中**未捕获异常**以及 **Signals** (low memory 崩溃除外)的方式获取 `NSException` 参数的 `callStackSymbols` 的属性所描述的调用栈信息。(在 App 中无法直接获取崩溃日志文件)

[Fabric](https://docs.fabric.io/apple/crashlytics/overview.html)  
[CrashProbe](https://github.com/bitstadium/crashprobe)


[*Understanding and Analyzing Application Crash Reports*](https://developer.apple.com/library/content/technotes/tn2151/_index.html) / 
[*Understanding Crash Reports on iPhone OS*](https://developer.apple.com/videos/play/wwdc2010/317/) / 
[*Advanced Debugging and the Address Sanitizer*](https://developer.apple.com/videos/play/wwdc2015/413/) / 
[*Analyzing Crash Reports*](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/AppDistributionGuide/AnalyzingCrashReports/AnalyzingCrashReports.html) / 
[*Getting Crash Logs*](https://developer.apple.com/library/content/qa/qa1747/_index.html) / 
[*Demystifying iOS Application Crash Logs*](https://www.raywenderlich.com/23704/demystifying-ios-application-crash-logs) / 
[*iOS Crash Symbolication for dummies*](https://www.bugsee.com/blog/ios-crash-symbolication-dummies-part-1/)