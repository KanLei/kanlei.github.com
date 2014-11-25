---
layout: post
title: "BeforeFieldInit"
description: ""
category: "Article" 
tags: [c#]
---
{% include JB/setup %}


## 静态构造函数初始化

众所周知，类中的静态构造函数的执行规则是：

- 任何实例对象被初始化之前
- 任何静态成员方法被调用之前
- 任何静态成员变量被使用的时候{因为初始化静态成员变量是放在静态构造函数中执行的}

按照上面的规则，我们来测试一段代码：

``` c#
class Foo
{
    public static string test = Bar();

    public static string Bar()
    {
        WriteLine("test...");
        return "test";
    }
}

static void Main()
{
    WriteLine("starting...");

    var t = Foo.test;
}
```

输出结果是：

`test`...  
`starting`...  

惊讶吗，如果我们把静态构造函数添加到 `Foo` 中，再进行测试：

``` c#
class Foo
{
    public static string test = Bar();

    static Foo() { }

    public static string Bar()
    {
        WriteLine("test...");
        return "test";
    }
}
```

此时的输出结果是：

`starting`...  
`test`...

难道是 `var t = Foo.test;` 导致了静态构造函数被提前执行了，但是，当我们显示的定义了静态构造函数之后，为什么执行顺序又正常了呢？现在我们改成调用静态方法，再进行测试：

``` c#
static void Main()
{
    WriteLine("starting...");

    var t = Foo.Bar();
}
```

不管有没有有定义静态构造函数，输出结果一样：

`starting`...    // 执行 `Main`  
`test`...        // 执行 `test` 变量的初始化  
`test`...        // 执行 `Bar` 方法的调用

调用静态方法并没有导致出现上面奇怪的初始化现象，可见，调用静态成员变量可能会导致静态构造函数的提前执行。

初始化实例对象也符合正常的执行顺序。

这次，我们将静态成员变量的调用放置到一个新的方法中{去除静态构造函数}：

``` c#
static void Main(string[] args)
{
    WriteLine("starting...");

    Test();
}

static void Test()
{
    var t = Foo.test;
}
```
这次，静态构造函数并没有提前执行，输出结果是：

`starting`...  
`test`...

## BeforeFieldInit

在上面的测试中，之所以出现如此奇怪的现象，是因为，没有显示定义静态构造函数的类被标记为 `beforefieldinit` ，在使用静态成员变量时，会导致静态构造函数被提前初始化；而显示定义了静态构造函数后，类并没有被标记 `beforefieldinit` ，因此，静态构造函数按应有的顺序执行。{可查看 `IL` 代码}

> Specifies that calling static methods of the type does not force the system to initialize the type.  -`MSDN`

这个解释太简略了，这里有一个更详细的回答。
[stackoverflow](http://stackoverflow.com/questions/610818/what-does-beforefieldinit-flag-do)

## 单例模式与静态变量初始化

通常我们会优先使用静态成员变量来定义单例模式，实现无锁的单例。结合上面所讲解的内容，可以通过显示地定义静态构造函数来保证静态成员变量不会被提前初始化。

``` c#
sealed class Foo
{
    public static Foo f { get; } = new Foo();

    static Foo() { }
    private Foo() { }
}
```