---
layout: post
title: "Count word or line"
description: ""
category: "snippycode"
tags: [c]
---
{% include JB/setup %}

## 计算输入一行中单词的个数

``` c
/* version 1 */
#include <stdio.h>
#include <ctype.h>

#define IN 1	/* inside a word */
#define OUT 0	/* outside a word */

/* count words */
int main()
{
	int c, state, nw;

	state = OUT;
	nw = 0;
	while ((c = getchar()) != EOF)
	{
		if (!isalpha(c))
			state = OUT;
		else if (state == OUT)
		{
			state = IN;
			++nw;
		}
	}
	printf("%d", nw);
	return 0;
}
```

``` c
/* version 2 */
int countWord(char *s)
{
    int nw = 0;

    while(*s)
        if(isalpha(*s++) && !isalpha(*s))
            nw++;
    return nw;
}
```

## 找出一行中最长的单词并打印

``` c
/* version 1 */
#include <stdio.h>
#include <ctype.h>

#define MAXWORD 100	/* maximum input word length */

int getword(char *word, int maxvalue, int *f);
void copy(char *dest, const char *src);

/* print the longest input word */
int main()
{
	int len;		/* current word length */
	int max;		/* maximum length seen so far */
	int f[1] = { 0 };	/* 1 represent '\n' received */
	char word[MAXWORD];	/* current input word */
	char longest[MAXWORD];	/* longest word saved here */


	max = 0;
	while ((len = getword(word, MAXWORD, f)) > 0)
	{
		if (len > max){
			max = len;
			copy(longest, word);
		}
		if (*f)		/* 如果接收到回车符结束程序 */
			break;
	}
	if (max > 0)
		printf("%s\n", longest);

	return 0;
}

int getword(char *word, int maxvalue, int *f)
{
	int c, i;
	char *p = word;

	for (i = 0; i < maxvalue - 1 && isalpha(c = getchar()); i++)
		*p++ = c;
	*p = '\0';
	if (c == '\n')
		*f = 1;
	return i;
}

/* copy char from src to dest */
void copy(char *dest, const char *src)
{
	char *p = dest;
	while (*p++ = *src++);
}
```

``` c
/* version 2 */
void prinLongestWord(char *s)
{
	char longest[1000] = { 0 };
	char word[1000] = { 0 }, *p = word;

	do
	{
		if (isalpha(*s))
			*p++ = *s;
		else
		{
			if (strlen(word) > strlen(longest))
				strcpy(longest, word);
			p = word;
		}
		s++;
	} while (s[-1]);
	puts(longest);
}
```

``` c
/* versoin 3 */
void prinLongestWord(char *s)
{
	int i, maxCount, index, count, len;

	maxCount = index = count = 0;
	len = strlen(s);
	for (i = 0; i <= len; i++)
	{
		if (isalpha(s[i]))
			count++;
		else if (count > maxCount || (count = 0))
			maxCount = count, index = i - count, count = 0;
		/*else
			count = 0;*/
	}

	while (isalpha(s[index]))
		putchar(s[index++]);
}
```

## 找出输入多行中，最长的一行字符并打印

``` c
#include <stdio.h>

#define MAXLINE 1000	/* maximum input line length */

int getline(char *line, int maxvalue);
void copy(char *dest, const char *src);

/* print the longest input line */
int main()
{
	int len;				/* current line length */
	int max;				/* maximum length seen so far */
	char line[MAXLINE];		/* current input line */
	char longest[MAXLINE];	/* longest line saved here */

	max = 0;
	while ((len = getline(line, MAXLINE)) > 0)
	if (len > max){
		max = len;
		copy(longest, line);
	}
	if (max > 0)
		printf("%s", longest);

	return 0;
}


int getline(char *line, int maxvalue)
{
	int c, i;
	char *p = line;

	for (i = 0; i < maxvalue - 1 && (c = getchar()) != EOF && c != '\n'; i++)
		*p++ = c;
	if (c == '\n')
		*p++ = '\n';
	*p = '\0';
	return i;
}

/* copy char from src to dest */
void copy(char *dest, const char *src)
{
	char *p = dest;
	while (*p++ = *src++);
}
```

## 统计字母、空格、数字和其它字符的个数


``` c
/* 统计输入中字母、空格、数字和其它字符的个数 */
void count()
{
	int alpha, space, digit, other, c;

	alpha = space = digit = other = 0;
	while ((c=getchar())!=EOF)
	{
		if (c >= 'A' && c <= 'Z' || c >= 'a' && c <= 'z')
			++alpha;
		else if (c == ' ')
			++space;
		else if (c >= '0' && c <= '9')
			++digit;
		else
			++other;
	}
	printf("字母: %d, 空格: %d, 数字: %d, 其它: %d", alpha, space, digit, other);
}
```

## 输出整数，统计位数并逆序输出

``` c
void countAndReverseNumber(void)
{
	int c, count, number;

	number = 0;
	for (count = 0; (c = getchar()) != '\n'; count++)
	{
		putchar(c);
		number = number * 10 + (c - '0');
	}
	printf("\n%d位数\n", count);
	while (number)
	{
		printf("%d", number % 10);
		number /= 10;
	}
}
```

*reference: The c programming language*
