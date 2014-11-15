---
layout: post
title: "String in C#"
description: ""
category: "Article"
tags: [c#]
---
{% include JB/setup %}

## string and String

string is an alias for String in the .Net Framework.  
eg:

```c#
using string = System.String
```

## Null strings and empty strings

> A null string is different from an empty string, which is a string whose value is "" or String.Empty.

IsNUllOrEmpty, which indicates whether a string is either null or is equal to String.Empty.  
eg:

```c#
string str = "";

// if (str == null || str.Equals(String.Empty))
if(String.IsNullOrEmpty(str)
```

IsNullOrWhiteSpace, which indicates whether a string is null, equals String.Empty, or consists exclusively of
white-space characters.  
eg:

```c#
string str = " ";

// if (str == null || str.Equals(String.Empty) || str.Trim().Equals(String.Empty))
if(String.IsNullOrWhiteSpace(str))
```

## Strings are immutable(like delegate)
>  Concatenation two strings does not re-use the content of either of the original strings; it creates a new string and copies the two strings into the new string. Taking the substring of a string does not re-use the content of the original string. Again, it just makes a new string of the right size and makes a full copy of the data. 

[*stackoverflow*](http://stackoverflow.com/questions/2365272/why-net-string-is-immutable)
[*immutable*](http://blogs.msdn.com/b/ericlippert/archive/tags/immutability/default.aspx?PageIndex=1)

## String Compare

> When the == operator is used to compare two strings, the Equals method is called, which checks for the equality
of the contents of the strings rather than the references themselves. Note that operator overloading only works
here if both sides of the operator are string expressions at complie time. If either side of the operator is of
type object as far as the complier is concerned, the normal == operator will be applied, and simple reference equality
will be tested.

String.Equals() compare the values when compare two strings. If '==' use to compare two strings, will compare values, too. But if either side is type object '==' will compare references. String.ReferenceEquals() compare the references.

```c#
string a = "hello";
string b = "h";
b += "ello";

Console.WriteLine(a == b);                              // True :compare value
Console.WriteLine(String.Equals(a, b));                 // True :compare value
Console.WriteLine((object)a == (object)b);              // False:compare reference
Console.WriteLine(String.Equals((object)a, (object)b)); // True :compare value
Console.WriteLine(String.ReferenceEquals(a, b));        // False:compare reference
```

## Intern Pool

> The common language runtime conserves string storage by maintaining a table, called the intern pool, that contains a single reference to each unique literal string declared or created programmatically in your program. Consequently, an instance of a literal string with a particular value only exists once in the system.

For example, if you assign the same literal string to several variables, the runtime retrieves the same reference to the literal string from the intern pool and assigns it to each variable.

**Intern()**: The Intern method uses the intern pool to search for a string equal to the value of str. If such a string exists, its reference in the intern pool is returned. *If the string does not exist, a reference to str is added to the intern pool, then that reference is returned.*

**IsInterned()**: This method looks up the value of str in the intern pool. If the value of str has already been interned, a reference to that instance is returned; otherwise, null is returned.

Let me show you a small fragment:

```c#
string s1 = "abc";
string s2 = "ab";
string s3 = s2 + "c";

// 这里是检测intern pool中是否已经存在 s3 的值
Console.WriteLine(String.IsInterned(s3));           // abc

// s3 与 s1 引用不同
Console.WriteLine(String.ReferenceEquals(s1, s3));  // False

// 使用 s3 引用与 s1 相同
Console.WriteLine(String.ReferenceEquals(s1, String.Intern(s3)));  // True
```

[*stackoverflow*](http://stackoverflow.com/questions/22270545/why-string-interned-but-has-different-references) / [*msdn*](http://msdn.microsoft.com/en-us/library/system.string%28v=vs.110%29.aspx) / [*anytao*](http://www.cnblogs.com/anytao/archive/2008/08/27/must_net_22.html) / [*eric lippert*](http://blogs.msdn.com/b/ericlippert/archive/2009/09/28/string-interning-and-string-empty.aspx) / [*eric lippert*](http://blogs.msdn.com/b/ericlippert/archive/2011/07/19/strings-immutability-and-persistence.aspx?Redirected=true)
