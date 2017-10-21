---
layout: post
title: "Use Fody To Simplify Code"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### Fody

> Extensible tool for weaving .net assemblies, attempts to eliminate that plumbing code through an extensible add-in model. - Fody

Fody 可以在编译阶段修改 IL 代码，使用 Fody，我们可以不必在代码文件中编写大量的样板代码，Fody 会将样板代码在编译期插入到最终的代码文件中。

#### PropertyChanged

在使用 MVVM 模式中，通常我们会将 ViewModel 实现 INotifyPropertyChanged

```csharp
public class BaseViewModel : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;

    protected void OnPropertyChanged([CallerMemberName]
                                     string property = "")
    {
        PropertyChanged?.Invoke(this,
         new PropertyChangedEventArgs(property));
    }
}
```

在每个 Property 中调用 OnPropertyChanged 方法

```csharp
private string foo;
public string Foo
{
    get { return foo; }
    set
    {
        foo = value;
        OnPropertyChanged();
    }
}
```

每一个需要通知的属性，都需要定义成上面这样，不仅增加了键盘敲击次数，也增加了代码的阅读难度。我们也可以通过其它方式来消除样板代码，如 [BindableProperty](http://www.sullinger.us/blog/2014/12/20/good-bye-onpropertychanged-hello-bindableproperty)，但 Fody 会让代码的定义更加简洁直观。

##### 使用 PropertyChanged.Fody 消除样板代码

PropertyChanged.Fody 会自动在编译期为实现了 INotifyPropertyChanged 的类插入 OnPropertyChanged() 的调用，比如像上面的属性定义可以定义成这样

```csharp
public string Foo { get; set; }
```

使用 PropertyChanged.Fody 需要3步

1. 使用 nuget 安装 PropertyChanged.Fody
2. 配置 FodyWeavers.xml
3. Release 下设置 Debug information

FodyWeavers.xml 配置用于控制代码生成的行为，通过

```xml
<Weavers>
	<PropertyChanged EventInvokerNames="RaisePropertyChanged"/>
</Weavers>
```

以上配置，Fody 默认会为 Property Set 添加 RaisePropertyChanged() 方法调用。

由于 Fody 需要获取项目的调试文件，所以 Realse 模式下，依然需要设置保留调试符号，如 Symbols only，Fody 才能正常工作。

[*Fody PropertyChanged*](https://github.com/Fody/PropertyChanged)  
[*Pdb full vs only*](https://stackoverflow.com/questions/7713514/should-i-compile-release-builds-with-debug-info-as-full-or-pdb-only)  
[*Debugging with Symbols*](https://msdn.microsoft.com/en-us/library/windows/desktop/ee416588(v=vs.85).aspx#using_symbols_for_debugging)