---
layout: post
title: "Custom NavigationBar"
description: ""
category: 
tags: [iOS]
---
{% include JB/setup %}


### 为什么要自定义导航条

在开发 iOS APP 的过程中，通常我们会对 `UINavigationController` 中的 `UINavigationBar` 样式做些调整，以便于满足我们 APP 的主题风格。比如，隐藏、自定义颜色、字体、设置透明、渐变等效果。由于 `UINavigationBar` 由 Apple 提供，我们只能访问其部分公有的 API，同时一个导航栈只有一个共有的 `NavigationBar`，导致多个页面间自定义不同样式的导航条实现起来不仅麻烦而且极其繁琐。

比如，我们如果要在页面切换间隐藏导航条，并且保持动画不突兀，通常需要这样做：

```csharp
public override void ViewWillAppear(bool animated)
{
    NavigationController?.SetNavigationBarHidden(true, animated);
    base.ViewWillAppear(animated);
}
```

确保在调用 `base.ViewWillAppear` 之前隐藏，显示则在 `ViewWillDisappear` 中设置，行为让人难以捉摸。

如果我们需要透明导航条，要写的代码会更多。由于 `UINavigationBar` 视图结构关系，我们需要先遍历其子视图，找到控制背景色的子视图，然后对齐修改，像这样：

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

这种写法极其不安全，因为 Apple 可能在后续的版本中对 `UINavigationBar` 内部视图结构进行调整，最终导致线上程序崩溃或出现难以预测的 Bug。

以及无法方便的控制单个 `BarButtonItem` 的样式等，促使我们实现自己的导航条。

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

这种实现方式方便我们控制主视图的位置，但同时引入了另一个问题：我们在添加完容器视图之后，后续访问 `View` 时访问的是我们自定义 `ContainerView`，而不是真实地 `OriginView`，对 `View` 的修改和添加操作都会发生在 `ContainerView` 之上，因此不得不在父类中提供一个 `RealView` 的属性，并在编码时人为约定使用 `RealView`。(重写 View 属性，在 Get 方法中，当 View 不为空时，获取第一个子视图并返回貌似可以解决这个问题，但由于系统对 View 的访问和操作的不确定性，因此这种方式行为不明确)。

隐藏 `UINavigationBar`