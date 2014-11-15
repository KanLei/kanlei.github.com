---
layout: post
title: "Stack and Palindrome in C"
description: ""
category: "Article"
tags: [c, datastructure]
---
{% include JB/setup %}

> 利用栈判断是否是回文

``` c
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>


#define MAXSIZE 100
typedef char DataType;

typedef struct
{
    DataType elems[MAXSIZE];
    int currentPos;
} SeqStack;

SeqStack* Stack_Init(); // 初始化栈
int Stack_Push(SeqStack* s, DataType data);     // 压栈
int Stack_Pop(SeqStack* s, DataType* value);    // 出栈


int main()
{
	DataType arr[MAXSIZE]={0}, *p = arr, value;
	int isPalindrome = 1;
    SeqStack *s = Stack_Init();
	printf("请输入要判断回文的字符串：\n");
	scanf("%s", arr);

	/* 将字符串依次压栈 */
	while(*p)  Stack_Push(s,*p++);

	p = arr;

	/* 字符串依次出栈 */
	while(Stack_Pop(s, &value) != 0)
	{
		if(value != *p++){
			isPalindrome = 0;
			break;
		}
	}

	isPalindrome ? printf("是回文\n") : printf("不是回文\n");

    return 0;
}

/* 初始化栈 */
SeqStack* Stack_Init()
{
    SeqStack *s = (SeqStack*)malloc(sizeof(SeqStack));
    assert(s != NULL);  // 判断内存是否申请成功
    s->currentPos = -1;
    return s;
}

/* 压栈 */
int Stack_Push(SeqStack* s, DataType data)
{
    int error;
    if(s == NULL) error = 0;
    else if(s->currentPos >= MAXSIZE) error = 0;
    else
    {
        s->elems[++s->currentPos] = data;
        error = 1;
    }
    return error;
}

int Stack_Pop(SeqStack* s, DataType* value)
{
	int error;
    if(s == NULL) error = 0;
    else if(s->currentPos == -1) error = 0;
    else
    {
        *value = s->elems[s->currentPos--];
        error = 1;
    }
    return error;
}
```

---

> 利用指针判断是否是回文

``` c
int IsPalindrome1(const char* str)
{
	const char *p = str;

	while (*p != '@') p++;

	--p;
	while (str <= p)
		if (*str++ != *p--)
			return 0;

	return 1;
}
```

``` c
int IsPalindrome2(const char* str)
{
	const char *p = str;

	while (*p != '@') p++;
	for (--p; str <= p && *str == *p; str++, p--);

	return str > p ? 1 : 0;
}
```
