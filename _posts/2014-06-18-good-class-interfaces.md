---
layout: post
title: "Good Class Interfaces"
description: ""
category: "Book"
tags: [book]
---
{% include JB/setup %}

## Class

    ADT(abstract data type)是指一些数据及对这些数据所进行的操作的集合。

> 你可以像在现实世界中那样操作实体，而不用在底层实现上操作它。

注：把底层实现隔离到一组子程序中，为外部世界提供更好的抽象层。把内部细节封装成方法，如：属性、栈、集合、文件等操作，我们无需关心底层的实现，只需要调用方法接口便可对其进行操作。

面向对象的编程语言中，如 c#，类就是一种抽象数据类型，我们通过属性，方法暴露内部接口给外部世界，从而对外部世界实现了内部细节的隐藏。

> 类中抽象操作应该保持一致性（即针对某一类数据的统一操作），避免包含混杂的函数。

一致性的例子：

``` c++
class EmployeeCenus {
    public:
        ...
        void AddEmployee(Employee employee);
		void RemoveEmployee(Employee employee);
		Employee NextEmployee();
		Employee FirstEmployee();
		Employee LastEmployee();
}
```

> 在考虑类的时候有一种很好的方法，就是把类看作是一种实现 ADT 的机制，每个类应该实现一个 ADT 并且仅实现这一个 ADT。

**良好的封装**

> 抽象通过提供一个可以让你忽略实现细节的模型来管理复杂度，而封装则强制组织你看到细节。两者应相辅相成，共同存在。

注：我们把一组操作抽象为一个方法，为外部提供接口，也就同时意味着这组操作被封装到方法中。

> 尽可能地限制类和成员的可访问性。

> 不要公开暴露成员数据。（感谢**属性**）

> 每当你发现是通过查看类的内部实现来得知如何使用这个类的时候，你就不是在针对接口编程了，而是透过接口针对内部实现编程了。

**继承**

**LSP(Liskov Substitution Principle)** 派生类必须能通过基类的接口而被使用，且使用者无需了解两者之间的差异。

> 避免过度设计(比如只有一个子类的继承结构)

尽量使用多态，避免大量的类型检查。

``` c#
switch(shape.type)
{
	case Shape_Circle:
		shape.DrawCircle();
		break;
	case Shape_Square:
		shape.DrawSquare();
		break;
}
```

在这个例子中，对 `shape.DrawCircle()` 和 `shape.DrawSquare()` 的调用应该用一个叫 `shape.Draw()` 的方法来替代，因为无论形状似圆还是方都可以调用这个方法来绘制。

但是，`case` 语句有时也用来把种类确实不同的对象或行为分开。下面就是一个在面向对象编程中合理采用 `case` 语句的例子：

``` c#
switch(ui.Command())
{
	case Command_OpenFile:
		OpenFile();
		break;
	case Command_Print:
		Print();
		break;
	case Command_Save:
		Save();
		break;
	case Command_Exit:
		ShutDown();
		break;
}
```

此时也可以创建一个基类并派生一些派生类，再用多态的 `DoCommand()` 方法来实现每一种命令。但在像这个例子一样简单的场合中，`DoCommand()` 意义是在不大，因此采用 `case` 语句才是更容易理解的方案。

**初始化成员**

初始化应尽量放到构造函数中，以下除外：

- 初始化为默认值，无需在构造函数中进行（如：c# 会把相应的类型赋初始值）
- 在多个构造函数中都要初始化为相同的值时，可以提取到构造函数外部进行，或者使用构造函数链


**何时使用继承，何时使用包含？**

- 如果多个类「共享数据」而非行为，应该创建这些类可以包含的共用对象。
- 如果多个类「共享行为」而非数据，应该让它们从共同的基类继承而来，并在基类里定义共用的子程序。
- 如果多个类「共享数据和行为」，应该让它们从一个共同的基类继承而来，并在基类里定义共用的数据和子程序。
- 当你想由基类控制接口时，使用继承；当你想自己控制接口时，使用包含。

**创建类的理由**

- 对现实世界中的对象建模
- 对抽象对象建模
- 降低复杂度
- 隔离复杂度
- 隐藏实现细节
- 限制变化所影响的范围
- 隐藏全局数据
- 让参数传递更顺畅
- 创建中心控制点
- 让代码更易于重用
- 为程序族做计划
- 把相关操作放到一起
- 实现特定的重构

Code Complete
