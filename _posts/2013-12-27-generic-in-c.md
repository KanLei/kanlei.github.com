---
layout: post
title: "Generic in C"
description: ""
category: "snippycode"
tags: [c]
---
{% include JB/setup %}

## swap: version 1

``` c
/* int type */
void swap(int *a, int *b)
{
	int temp = *a;
	*a = *b;
	*b = temp;
}
```

## swap: version 2

``` c
#include <stdio.h>
#include <string.h>
#include <malloc.h>

/* 'generic type' swap the value of two same size element.
	C++ 中的 template 会为每种类型生成一份代码，如 int, string...
	C 中只存在同样一份代码
	*/
void swap(void *a, void *b, int size)
{
	void *temp = malloc(size);

	memcpy(temp, a, size);
	memcpy(a, b, size);
	memcpy(b, temp, size);

	free(temp);
}

int main()
{
	/* swap value */
	char arr1[] = "Hello";
	char arr2[] = "World";
	swap(arr1, arr2, 5 * sizeof(char));
	printf("%s  %s\n", arr1, arr2);

	/* swap pointer */
	char *a = "Husband";
	char *b = "Wife";
	swap(&a, &b, sizeof(char *));
	printf("%s  %s\n", a, b);

	return 0;
}
```
***

## Stack:version 1

``` c
/* stack.h
   stack with int */
typedef struct{
	int *elems;
	int logicalLength;
	int allocLength;
}Stack;

void StackNew(Stack *s);
void StackDispose(Stack *s);
void StackPush(Stack *s, int value);
int StackPop(Stack *s);
```

> realloc 函数会首先在原始的内存位置检查是否有足够的空间扩充，
  如果有，则扩充，并返回原始内存的首地址；否则，重新开辟一块新
  的内存块，并拷贝原始的数据，然后释放原始内存，返回新内存
  块的首地址。

``` c
/* stack.c 
   stack with int */
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>

#include "stack.h"

void StackNew(Stack *s)
{
    s->logicalLength = 0;
    s->allocLength = 4;
    s->elems = malloc(4 * sizeof(int));
    assert(s->elems != NULL);
}

void StackDispose(Stack *s)
{
    free(s->elems);
}

void StackPush(Stack *s, int value)
{
    if(s->logicalLength == s->allocLength)
    {
        s->allocLength *= 2;
        s->elems = realloc(s->elems, s->allocLength * sizeof(int));
        assert(s->elems != NULL);
    }
    s->elems[s->logicalLength++] = value;
}

int StackPop(Stack *s)
{
    assert(s->logicalLength > 0);
    s->logicalLength--;
    return s->elems[s->logicalLength];
}

int main()
{
    Stack s;

    StackNew(&s);
    StackPush(&s, 2);
    StackPush(&s, 4);
    printf("%d", StackPop(&s));
    return 0;
}
```
