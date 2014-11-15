---
layout: post
title: "Merge Linklist in C"
description: ""
category: "Article"
tags: [c, datastructure]
---
{% include JB/setup %}

``` c
#include <assert.h>
#include <stdio.h>
#include <stdlib.h>

typedef struct node
{
	int data;
	struct node *next;
} Node, *LinkList;

LinkList InitLinkList();	// 初始化单链表
void InsertNode(LinkList l, int num);	// 插入结点
int GetLength(LinkList l);	// 获取链表长度
LinkList MergeLinkList(LinkList first, LinkList second);	// 合并链表


int _tmain(int argc, _TCHAR* argv[])
{
	int i, len;
	LinkList A = InitLinkList();
	LinkList B = InitLinkList();
	LinkList C = InitLinkList();
	Node *p, *q, *r;
	p = A;
	q = B;
	r = C;

	InsertNode(A, 2);
	InsertNode(A, 4);
	InsertNode(A, 8);
	InsertNode(A, 5);

	InsertNode(B, 2);
	InsertNode(B, 5);
	InsertNode(B, 6);
	InsertNode(B, 9);

	len = GetLength(A);
	for (i = 0; i < len; i++)
	{
		printf("%d  ", p->next->data);
		p = p->next;
	}
	printf("\n");
	len = GetLength(B);
	for (i = 0; i < len; i++)
	{
		printf("%d  ", q->next->data);
		q = q->next;
	}

	printf("\n");

	r = MergeLinkList(A, B);
	len = GetLength(r);
	for (i = 0; i < len; i++)
	{
		printf("%d  ", r->next->data);
		r = r->next;
	}

	return 0;
}

/* 初始化链表 */
LinkList InitLinkList()
{
	LinkList L;
	L = (LinkList)malloc(sizeof(Node));
	assert(L != NULL);
	L->next = NULL;
	return L;
}

/* 插入结点 */
void InsertNode(LinkList l, int num)
{
	Node *p = l;
	Node *node = (LinkList)malloc(sizeof(Node));
	assert(node != NULL);
	node->data = num;
	node->next = NULL;

	if (l->next == NULL)
		l->next = node;
	else
	{
		while (p->next != NULL && p->next->data < num)
			p = p->next;
		// Insert
		node->next = p->next;
		p->next = node;
	}
}

/* 获取链表长度 */
int GetLength(LinkList l)
{
	Node *p = l->next;
	int i;
	for (i = 0; p!=NULL; i++)
	{
		p = p->next;
	}
	return i;
}

/* 合并两个有序链表，得到有序链表 */
LinkList MergeLinkList(LinkList first, LinkList second)
{
	// 如果result作为参数传入，则必须返回值返回，因为此处对指针重新赋值
	LinkList result = first;
	
	Node *p, *q, *r;
	p = first->next;
	q = second->next;
	r = result;

	// 释放表头结点
	free(second);

	while (p && q)
	{
		if (p->data <= q->data)
		{
			r->next = p;
			p = p->next;
		}
		else
		{
			r->next = q;
			q = q->next;
		}
		r = r->next;
	}

	if (p)
		r->next = p;
	if (q)
		r->next = q;

	return result;
}
```
