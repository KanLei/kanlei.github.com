---
layout: post
title: "Closure in C#"
description: ""
category: "Article"
tags: [c#]
---
{% include JB/setup %}

First things first, think out the output of following fragment.

```c#
var acts = new List<Action>();
for (int i = 1; i < 5; i++)
    acts.Add(() => Console.WriteLine(i));

acts.ForEach(act => act());
```

And this fragment in C# 5:

```c#
var acts = new List<Action>();
// run in c# 5
foreach (var i in Enumerable.Range(1, 5))
        acts.Add(() => Console.WriteLine(i));

acts.ForEach(act => act());
```

Do you think out the output?

Let me explain the first demo.The first demo will output:

```c#
6
6
6
6
6
```

But why? Because all the delegates reference **the same variable**(not value) i. Variable i last assignmented 6. A variable was captured by anonymous method will create a new class instance contains the variable with in for loop. And the delegate use the variable through the new clall instance. This called closure. Therefore, i not store on stack, but store on heap. So this tell us, not all local variables store on the stack.

How to slove this problem? We can write code in this way.

```c#
var acts = new List<Action>();
for (int i = 1; i < 5; i++)
{
    int j = i;
    acts.Add(() => Console.WriteLine(j));
}
acts.ForEach(act => act());

output:
1
2
3
4
5
```

Each loop will create a variable j, and j was captured by current delegate. So all delegates capture different j.

Foreach's behavior used to be similar to For loop, but this situation has changed by C# team in C# 5. Now closures will close over a fresh copy of the variable each time.
Second demo output:

```c#
1
2
3
4
5
```

[*Closing over the loop variable considered harmful*](http://ericlippert.com/2009/11/12/closing-over-the-loop-variable-considered-harmful-part-one/)   
[*The Beauty of Closures*](http://csharpindepth.com/Articles/Chapter5/Closures.aspx)
