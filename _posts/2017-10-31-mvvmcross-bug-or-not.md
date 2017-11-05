---
layout: post
title: "MvvmCross bug or not ?"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### Bug?

最近在编写 `MvvmCross` 绑定代码时发现一个比较奇怪的现象，绑定代码书写的顺序不同会导致最终产生不同的结果。

测试代码如下：

```csharp
set.Bind(Button).For(b => b.Enabled).To(vm => vm.GoEnable);
set.Bind(Button).To(vm => vm.GoCommand);
```

`ViewModel` 定义：

```csharp
private bool goEnable;
public bool GoEnable
{
    get => goEnable;
    set => SetProperty(ref goEnable, value);
}

public IMvxCommand GoCommand => new MvxCommand(() => { });
```

`goEnable` 变量默认值为 `false`，但由于先绑定 `GoEnable` 后绑定 `GoCommand` 导致 `Button` 的初始状态为**可用**，如果切换绑定代码的顺序，`Button` 状态则表现正常，为**不可用**状态。

绑定顺序怎么会影响最终绑定结果呢？难道是 `MvvmCross` 中的 bug？毕竟使用 `MvvmCross` 的过程中的确偶尔会遇到一些未修复的 bug。

### 分析源码

对于这种奇怪的现象，查看代码是最有效的一种解决方式，通过阅读代码我们可以清晰地了解问题产生的前因后果。

首先 Clone [MvvmCross](https://github.com/MvvmCross/MvvmCross) 源码到本地，设置 `Playground.iOS` 为启动项目，接着修改 `RootView` 添加测试按钮以及 `ViewModel` 中的属性定义，完成这些前置工作后，我们就可以开始启动调试，寻找问题产生的原因了。

```
1. MvxFluentBindingDescriptionSet<TOwning, TSource>
2. MvxBaseFluentBindingDescription<TTarget>
3. MvxBindingContextOwnerExtensions
4. MvxFromTextBinder
5. MvxFullBinding
6. MvxTargetBindingFactoryRegistery
7. MvxCustomBindingFactory
8. MvxSourceStep
9. MvxPathSourceStep
10. MvxLeafPropertyInfoSourceBinding
11. MvxConvertingTargetBinding
12. MvxUIControlTargetBinding
13. MvxTaskBaseBindingContext

1. -----Apply-----------------------
        |
2. -----Apply-----------------------
        |
3. -----AddBindings---------------------------------------------------------------------------------------------------------------------------------------------------------------
        |                                                                                                                                                       |
4. -----Bind----BindSingle---------                                                                                                                             |
                |                                                                                                                                               |
5. -------------CreateTargetBinding-----------------------------------------CreateSourceBinding---UpdateTargetOnBind----UpdateGargetFromSource----              |
                |                                                           |                                           |                                       |
6. -------------CreateBinding---TryCreateSpecificFactoryBinding------       |                                           |                                       |
                                |                                           |                                           |                                       |
7. -----------------------------CreateBinding------------------------       |                                           |                                       |
                                                                            |                                           |                                       |
8. -------------------------------------------------------------------------GetValue-----------                         |                                       |
                                                                            |                                           |                                       |
9. -------------------------------------------------------------------------GetSourceValue-----                         |                                       |
                                                                            |                                           |                                       |
10. ------------------------------------------------------------------------GetValue-----------                         |                                       |
                                                                                                                        |                                       |
11. --------------------------------------------------------------------------------------------------------------------SetValue-------                         |
                                                                                                                        |                                       |
12. --------------------------------------------------------------------------------------------------------------------SetValueImpl---RefreshEnableStatus--    |
                                                                                                                                                                |
13. ------------------------------------------------------------------------------------------------------------------------------------------------------------RegisterBinding-----
```


以上列举的是调用 Apply 之后类与方法调用的时序图。可以看出，整个调用过程还是比较复杂的，不过我们只关注 **12** 的 `RefreshEnableStatus` 方法，该方法实现如下：

```csharp
private void RefreshEnabledState()
{
    var view = Control;
    if (view == null)
        return;

    var shouldBeEnabled = false;
    if (_command != null)
    {
        shouldBeEnabled = _command.CanExecute(null);
    }
    view.Enabled = shouldBeEnabled;
}
```

由该方法，我们很容易发现，原来在绑定 `UIButton` 到 `Command` 时，默认会对 `Command` 的 `CanExecute` 进行求值，并将结果设为 `UIButton` 的 `Enabled` 默认值。因此，这并不是一个 bug，而是一个特定的实现。到此，我们的源码分析就结束了，以后在绑定 `Command` 时，只需注意同时设置 `Command` 的 `CanExecute` 属性即可避免上面出现的 “bug”。