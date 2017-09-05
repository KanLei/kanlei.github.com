---
layout: post
title: "Custom NavigationBar"
description: ""
category: 
tags: [iOS]
---
{% include JB/setup %}


### 为什么要自定义导航条

在开发 iOS APP 的过程中，通常我们会对 `UINavigationController` 中的 `UINavigationBar` 样式做些调整，以便于满足我们 APP 的主题风格。比如，隐藏，自定义颜色、字体、设置透明，增加渐变等。由于 `UINavigationBar` 由 Apple 提供，我们只能访问其部分公有的 API，同时一个导航栈只有一个共有的 `NavigationBar`，导致多个页面间自定义不同样式的导航条实现起来不仅麻烦而且极其繁琐。

比如，我们如果要在页面切换间隐藏导航条，并且保持动画不突兀，通常需要这样做：

```csharp
public override void ViewWillAppear(bool animated)
{
    NavigationController?.SetNavigationBarHidden(true, animated);
    base.ViewWillAppear(animated);
}
```

确保在调用 `base.ViewWillAppear` 之前隐藏，显示则在 `ViewWillDisappear` 中设置，行为让人难以捉摸。

如果我们需要透明导航条，要写的代码会更多。由于 `UINavigationBar` 视图结构关系，我们需要先遍历其子视图，找到控制背景色的子视图，然后做出调整，像这样：

```csharp
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

这种做法并不安全，因为 Apple 可能在后续的版本中对 `UINavigationBar` 内部视图结构进行调整，最终导致线上程序崩溃或出现难以预测的 Bug。

以及无法方便的控制单个 `BarButtonItem` 的样式等以上原因，促使我们实现自己的导航条。

### 如何自定义导航条

#### 添加子视图作为导航条
              
简单地做法是在当前视图上添加一个自定义视图即可，可以将其定义在父类中，这样继承自父类的每个子类都拥有自己单独的导航条，同时也可以方便地对视图调整，添加不同的子视图。

```
UIViewController --- UIView
                        |
                        ---- NavigationBar
```

这种实现方式虽然简单，但却存在一个问题，由于导航条作为主视图的子视图呈现，默认导航条会覆盖在主视图之上，如果我们从 `xib` 中加载视图，则需要在每个 `xib` 中预留出导航条的高度，才能保证内容视图不被导航条所覆盖。

#### 添加容器视图替换主视图

为了方便的控制内容视图是否被导航条所覆盖，我们可以通过增加一个容器视图来实现。

```
UIViewController --- ContainerView
                        |
                        ---- OriginView
                        |
                        ---- NavigationBar
```

首先自定义一个 `ContainerView`，获取 `ViewController` 的主视图 `OriginView` 之后，接着将其作为子视图添加到 `ContainerView` 子视图树中，最后添加自定义 `NavigationBar`，注意添加顺序，保证 `NavigationBar` 呈现于内容视图之上。此时，`xib` 中所有视图直接存在于 `OriginView` 中，我们只需调整 `OriginView` 的位置即可控制整个内容视图的位置。[实现代码参考](https://github.com/KanLei/Xamarin.iOS/blob/master/CustomNavigationBar/BaseView.cs)

这种实现方式方便我们控制主视图的位置，但同时引入了另一个问题：我们在添加完容器视图之后，后续访问 `View` 时访问的是我们自定义 `ContainerView`，而不是初始的 `OriginView`，对 `View` 的修改和添加操作都会发生在 `ContainerView` 之上，因此不得不在父类中提供一个 `RealView` 的属性，并在编码时人为约定使用 `RealView`。(重写 View 属性，在 Get 方法中，当 View 不为空时，获取第一个子视图并返回貌似可以解决这个问题，但由于系统对 View 的访问和操作的不确定性，导致这种方式会产生不确定行为)。

隐藏 `UINavigationBar` 会带来另外一个问题：**Interactive Pop Gesture** 失效。

#### 恢复 Interactive Pop Gesture

支持 Slide-back 需要如下步骤：

* 自定义`NavigationController`实现`IUIGestureRecognizerDelegate`
* 在 `ViewDidLoad` 方法中设置`InteractivePopGestureRecognizer.Delegate = this;`
* 实现`ShouldBegin`当且仅当大于 1 个 ViewController 时支持返回
 
```csharp
[Export("gestureRecognizerShouldBegin:")]
public bool ShouldBegin(UIGestureRecognizer recognizer)
{
    if ((recognizer is UIScreenEdgePanGestureRecognizer)
        && (this.ViewControllers.Length <= 1))
    {
        return false;
    }
    return true;
}
```

* 实现`ShouldRecognizeSimultaneously`避免手势被子视图拦截

```csharp
[Export("gestureRecognizer:shouldRecognizeSimultaneouslyWithGestureRecognizer:")]
public bool ShouldRecognizeSimultaneously(UIGestureRecognizer gestureRecognizer, UIGestureRecognizer otherGestureRecognizer)
{
    return true;  // 保证与子视图同时识别
}
```

[*Swipe Back when hiding NavigationBar*](https://stackoverflow.com/questions/24710258/no-swipe-back-when-hiding-navigation-bar-in-uinavigationcontroller#answer-32131088) / 
[*Interactive Pop Gesture*](http://holko.pl/ios/2014/04/06/interactive-pop-gesture/) / 
[*一次性解决导航栏的所有问题*](http://www.jianshu.com/p/31f177158c9e) /
[*用Reveal分析网易云音乐的导航控制器切换效果*](http://jerrytian.com/2016/01/07/%E7%94%A8Reveal%E5%88%86%E6%9E%90%E7%BD%91%E6%98%93%E4%BA%91%E9%9F%B3%E4%B9%90%E7%9A%84%E5%AF%BC%E8%88%AA%E6%8E%A7%E5%88%B6%E5%99%A8%E5%88%87%E6%8D%A2%E6%95%88%E6%9E%9C/)
[*为每个控制器自定义 UINavigationBar*](http://www.jianshu.com/p/88bc827f0692) /
[*完全自定义导航栏的思路*](http://soledad.me/2017/01/18/about-tiptoes/)