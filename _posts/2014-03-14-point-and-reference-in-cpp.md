---
layout: post
title: "Point and Reference in C++"
description: ""
category: "article"
tags: [c++]
---
{% include JB/setup %}

``` c++
#include <stdio.h>
#include <stdlib.h>

void SwapPointer(int* x, int* y)
{
	int temp = *x;
	*x = *y;
	*y = temp;
}

void SwapReference(int& x, int& y)
{
	int temp = x;
	x = y;
	y = temp;
}

void SwapValue(int x, int y)
{
	int temp = x;
	x = y;
	y = temp;
}

int _tmain(int argc, _TCHAR* argv[])
{
	int x = 1;
	int y = 2;
	printf("x:%d, y:%d\n", x, y);
	//SwapPointer(&x, &y);
	SwapReference(x, y);
	//SwapValue(x, y);
	printf("x:%d, y:%d\n", x, y);

	return 0;
}
```
