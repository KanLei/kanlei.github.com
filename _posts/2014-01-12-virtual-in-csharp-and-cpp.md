---
layout: post
title: "virtual in c# and c++"
description: ""
category: "snippycode"
tags: [c#, c++]
---
{% include JB/setup %}

## virtual

* c#

``` c#
﻿using System;

namespace Program
{

    public class Program
    {
        static void Main(string[] args)
        {
            A a = new A();
            B b = new B();
            A c = new B();  // notice
            a.fun();
            b.fun();
            c.fun();
        }
    }

    class A
    {
        public virtual void fun()
        {
            Console.WriteLine("fun in A");
        }
    }

    class B:A
    {
        public virtual void fun()
        {
            Console.WriteLine("fun in B");
        }
    }
}
```

c# code will output:
> fun in A  
  fun in B  
  fun in A

* * *

* c++

``` c++
#include <iostream>
using namespace std;

class A
{
	public:
		virtual void fun()
		{
			cout << "fun in A" << endl;
		}
};

class B: public A
{
	public:
		virtual void fun()
		{
			cout << "fun in B" << endl;
		}
};

int main()
{
    A *a = new A();
    B *b = new B();
    A *c = new B();  // notice
    a->fun();
    b->fun();
    c->fun();
    return 0;
}
```

c++ code will output:
> fun in A  
  fun in B  
  fun in B

* * *
