---
layout: post
title: "LINQ vs SQL"
description: "Article"
category: 
tags: [linq, sql]
---
{% include JB/setup %}


## 声明式语言

LINQ 语言集成查询{Language Integrated Query}，有 LINQ to XML、LINQ to Object、LINQ to SQL。SQL 结构化查询语言{Structured Query Language}，主要用于 RDBMS。它们都是声明式语言，即专注于 how to do 而不是 what to do ，大大提升了代码的可读性和生产力。

## 投影

`LINQ` 以 `from` 开始：

``` c#
from f in fruit select f.Name
```

`SQL` 以 `select` 开始：

``` sql
select name from fruit
```

LINQ 和 SQL 都是用 select 投影出最终的结果。LINQ 之所以以 from 开始是因为查询表达式最终会依次转化为方法的调用，上面就可以转化为 fruit.Select(name => name)，因此书写顺序即执行顺序；而 SQL 的书写顺序是以 select 开始，但执行顺序却是以 from 开始，因此 SQL 的书写顺序与执行顺序不同。LINQ 的这个特性同时还会提供智能感知，通过数据源推断出 name 的类型后，后续的表达式就可以得知 name 的具体类型，从而调用其本身的方法。

## 筛选

LINQ:

``` C#
from f in fruit where f.Name == "apple" select f
```

SQL:

``` sql
select * from fruit where name = 'apple'
```

LINQ 和 SQL 都是使用 where 筛选满足条件的项/记录。

## 排序

### 升序

LINQ:

``` c#
from f in fruit orderby f.Name select f 
```

SQL:

``` sql
select * from fruit order by name
```

### 降序

LINQ:

``` c#
from f in fruit orderby f.Name descending select f 
```

SQL:

``` sql
select * from fruit order by name desc
```

## 分组

LINQ:

``` c#
from f in fruit group f by f.Price
```

SQL:

``` sql
select price from fruit group by price
```

需要注意的是，LINQ 中的查询表达式以 group by 或者 select 结尾，如果需要执行后续操作，需使用 into xxx。SQL 中的 select 中的列只能是 group by 中出现过的，这是由于 SQL 执行顺序所决定的，select 在 group by 之后执行，所以只能看到 group by 后的结果。与 SQL 相同，使用 into 的 LINQ 后续查询操作也只能看到 xxx 变量。


## 内联操作

LINQ:

``` c#
from s in student
join c in course on s.courseid equals c.id
select new { Name = s.Name, Course = c.Name }
```

SQL:

``` sql
select * from student join course on student.courseid = course.id
```

## 左外连接

LINQ:

``` c#
from s in student
join c in course on s.courseid equals c.id
into selectCourse
select new { Name = s.Name, Count = selectCourse.Count() }
```

SQL:

``` sql
select * from student left join course on student.courseid = course.id
```
LINQ 中的没有提供 SQL 中的 right join，可以调整左右两边序列位置来实现右外连接。


## 笛卡儿积

LINQ:

``` c#
from s in student
from c in course
select new { Name = s.Name, Course = c.Name }
```

SQL:

``` sql
select * from student, course
```


## 补充

当使用 LINQ 时，如过两个序列进行组合操作，会先缓存右边序列，然后遍历左边序列，所以，应当考虑把大的序列放在左边遍历，小的序列放在右边缓存。