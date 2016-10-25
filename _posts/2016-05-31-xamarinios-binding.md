---
layout: post
title: "Xamarin.iOS Binding"
description: ""
category: "article"
tags: [Xamarin.iOS]
---
{% include JB/setup %}

## 简介

`Xamarin.iOS Binding` 是对已有 `OC` 代码和组件重用的一种方式，使用这种方式，我们可以在 `C#` 中重用 `OC` 代码。

## Objective Sharpie

[Objective Sharpie](http://forums.xamarin.com/categories/ObjectiveSharpie) 是一个可以帮助我们将绑定过程自动化的工具，使用该工具可以帮助我们大大提高手动绑定效率，缩短绑定的时间。我们只需在工具自动生成绑定代码之后，手动调整工具无法正确生成绑定的部分代码即可。

## 绑定 OC 代码
在绑定之前，适当了解 `C#` 与 `OC` 之间语法映射关系，有助于我们更好的完成绑定和调整绑定结果（参考附录引用）。

绑定分为多种形式

- 绑定 Framework
- 绑定 Xcode Project
- 绑定 CocoaPods
- 绑定项目源码

前三种的绑定命令相对比较简单，分别如下：

绑定 Framework

> `sharpie bind -framework xxx.framework -sdk iphoneos9.3`

绑定 Xcode Project
> `sharpie bind xxx.xcodeproj -sdk iphoneos9.3`

绑定 Cocoapods
> `sharpie pod bind`

注意：当 Cocoapods 无法正常加载时，需要执行以下操作
> `rm -rf xxx.xcworkspace`  // 移除  
> `pod install`             // 重新安装依赖组件

如果终端提示 `command not found`，这时我们要先安装`Cocopods`，但由于被墙的原因，我们需要执行如下步骤
> `gem sources`  // 查看现有的源  
> `gem sources -r https://rubygems.org/`  // 移除 https 开头的源  
> `gem sources -a http://rubygems.org/`  // 增加以 http 开头的源  
> `sudo gem install cocoapods`  // 如果有问题，该命令需在根目录下运行  
> `rm -rf xxx.xcworkspace`  // 移除  
> `pod install`  // 在指定目录下执行安装依赖组件

如果绑定时出现如：*xxx.framework requires SDK iPhone9.2 which is not installed* 问题。需要首先安装对应 SDK 版本的 `Xcode`。`Xcode` 旧版[下载地址](https://developer.apple.com/downloads/)。安装 Xcode 时注意将现有 `Xcode` 提前重命名，避免现有 Xcode 被覆盖，安装完成后，恢复命名即可。

如果绑定中出现如：*unknown type name BOOL* 错误提示，应手动在 *.h 文件中添加相关头文件的引用如：`#import <UIKit/UIKit.h>` 然后重新执行绑定命令。

最后一种绑定方式，需要我们手动进行，根据我们现有项目文件结构的不同，分别包含以下步骤，生成 OC 静态链接库、创建 Fat.a 文件、指定绑定 .h 文件。

**生成 .a 文件**
打开 Xcode，File -> New -> Project -> iOS -> Framework & Library -> Cocoa Touch Static Library -> 输入项目名称（如 Test） -> Create。

选中 Test 文件夹右击 Move To Trash，将需要生成的 OC 源代码拖到 Test 项目所在面板下，在弹出的提示框中选择 Copy items if needed、Create groups 和 Add to targets:勾选 Test。

选中 Project 一栏下的 Test 项目，在 Info 选项卡中选择要部署的目标平台 iOS Deployment Target。

选中 Targets 一栏下的 Test 项目，根据原项目需要，配置 Build Settings，如 Other Linker Flags 等参数。在 Build Phases 下移除 Copy Files 一栏（由于我们生成的静态链接库直接用于绑定，不需要生成目录中包含头文件的生成），点击 + 添加 New Header Phrase，将需要的头文件添加到 Project 选项下（默认添加所有.h文件）。[Headers](http://stackoverflow.com/questions/10584936/understanding-xcodes-copy-headers-phase)，[Project Editor Help From Apple](https://developer.apple.com/library/ios/recipes/xcode_help-project_editor/_index.html)

选中顶部切换模拟器菜单栏中的 Test，在下拉选项中选中 Edit Scheme，在 Run 的 Info 一栏中的 Build Configuration 配置切换到 Release 模式。

选中模拟器，运行程序，此时我们会看到 Products 目录下生成 libTest.a 文件；然后切换模拟器到真机选项，再次运行模拟器。这时，已经生成了支持模拟器和真机架构的 Release 模式的静态链接库，接下来我们需要合并这两个.a文件。

右击 libTest.a 文件，Show In Finder。打开终端，执行以下命令
> `lipo -create A.a B.a -output C.a`

这里的 C.a 就是我们最终需要的静态链接库。

[lipo 指令参考](http://ss64.com/osx/lipo.html)
`lipo -info C.a`  // 查看库支持的架构

在终端中定位到 *.h 所在目录，执行如下命令
> `sharpie bind -sdk=iphoneos9.3 *.h`

会生成 APIDefinition 和 StructsAndEnums 文件。

## 创建 C# 绑定项目
上一节我们完成了绑定之前的所有准备工作，现在我们来着手创建绑定项目，该项目最终会生成动态链接库 (dll) 供我们使用。

打开 Xamarin Studio -> New Solution -> iOS(Library) -> Bindings Library，输入 Project Name:TestBinding.iOS 和 Solution Name:TestBinding，Create。

接着将我们上节中生成的 `ApiDefinition.cs` 和 `StructsAndEnum.cs` 中的代码分别复制并粘贴到对应的文件中。将生成的 C.a 文件拖放到 TestBinding.iOS 项目上，并选择复制到项目。这时我们会看到 Xamarin Studio 会自动帮我们生成了 C.linkwith.cs 文件，我们需要手动修改该文件，增加绑定过程中需要的参数配置信息。

**LinkTarget** 表示生成的库支持的架构，通过我们会指定
> `LinkTarget.ArmV7|LinkTarget.ArmV7s|`  
> `LinkTarget.Arm64|LinkTarget.Simulator|`  
> `LinkTarget.Simulator64`

**Frameworks** 表示该库中引用的 Framework，这时我们需要查看 OC 项目，在 Build Phases 的 Link Binary With Libraries 中查看所需要引用的 Framework，注意这里会列举出所有的引用，但此时我们只关注静态链接库，如

> `Frameworks = "Foundation UIKit CoreGraphics"`

**LinkerFlags** 表示我们要引用到的动态链接库和 Other Linker Flags，我们除了引用 Link Binary With Libraries 中的动态链接库，还要查看 Build Settings 中的 Linking 选项卡下的 Other Linker Flags 中是否指定额外的链接标记，如 -ObjC 等

> `LinkerFlags = "-lresolv -ObjC"`

*注意引用标记的写法，如 libz -> -lz，libbz2 -> -lbz2*

**生成项目并修复编译错误**

*Cannot create an instance of ... because it is an abstract class*

> 当绑定生成代码包含抽象类，用 `Protocol` 修饰时，如果生成的代码中要用到该类时，如 `Test`，需要手动添加一个对应的 `interface` 定义，如 `interface ITest{}` 并在其余用到 `Test` 类的地方，将其替换为`ITest`。此时编译器会在 `ITest` 接口中定义 `Test` 类中标注了 `Abstract` 的方法，其余方法以扩展方法的形式生成在一个新的如 `Test_Extension` 类中。[Protocol 生成代码](https://developer.xamarin.com/guides/cross-platform/macios/binding/binding-types-reference/#Protocols)
 
有时候我们无法 `new` 一个抽象类，此时可以采用 `ObjCRuntime.Runtime.GetNSObject()` 间接地获取该对象，并通过 `PerformSelector()` 的方式发送消息。

#### 附
[C# 与 Objective-C](https://developer.xamarin.com/guides/ios/advanced_topics/xamarin_for_objc/) /
[Events, Protocol and Delegates](https://developer.xamarin.com/guides/ios/application_fundamentals/delegates,_protocols,_and_events/) /
[Binding Objective-C](https://developer.xamarin.com/guides/cross-platform/macios/binding/)