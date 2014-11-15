---
layout: post
title: "FindAll vs Where"
description: ""
category: "article"
tags: [c#]
---
{% include JB/setup %}

I have worked as an intern in a mobile game company about one year ago. Then I found they always used Find()、FindAll() methods in game's project. When I begin to learn LINQ, I found there just one method is similar to FindAll(), named Where(), this is an extension method. 

Question:

    what's the differences between FindAll() and Where()?
    Which is faster?

Demo

``` c#
List<string> fruit = new List<string> { "pear", "apple", "banana", "orange" };

// FindAll
fruit.FindAll(x => x.StartsWith("a")).ForEach(x => Console.WriteLine(x));

// Where
foreach (var item in fruit.Where(x => x.StartsWith("a")))
{
    Console.WriteLine(item);
}

// parallel, this will be fastest
foreach(var item in fruit.AsParallel().Where(x=>x.StartsWith("a")))
{
    Console.WriteLine(item);
}
```

First of all, let's observe the **definition** of these two methods.

``` c#
// instance method, define in List<T> class
public List<T> FindAll(Predicate<T> match);
```

``` c#
// extension method,define in Enumerable class
public static IEnumerable<TSource> Where<TSource>(this IEnumerable<TSource> source,
                                                  Func<TSource, bool> predicate);
```

Now, let's observe the **implementation** of these two methods.

``` c#
[__DynamicallyInvokable]
public List<T> FindAll(Predicate<T> match)
{
    if (match == null)
    {
        ThrowHelper.ThrowArgumentNullException(ExceptionArgument.match);
    }
    List<T> list = new List<T>();
    for (int i = 0; i < this._size; i++)
    {
        if (match(this._items[i]))
        {
            list.Add(this._items[i]);
        }
    }
    return list;
}
```

``` c#
[__DynamicallyInvokable]
public static IEnumerable<TSource> Where<TSource>(this IEnumerable<TSource> source,
                                                  Func<TSource, bool> predicate)
{
    if (source == null)
    {
        throw Error.ArgumentNull("source");
    }
    if (predicate == null)
    {
        throw Error.ArgumentNull("predicate");
    }
    if (source is Iterator<TSource>)
    {
        return ((Iterator<TSource>) source).Where(predicate);
    }
    if (source is TSource[])
    {
        return new WhereArrayIterator<TSource>((TSource[]) source, predicate);
    }
    if (source is List<TSource>)
    {
        return new WhereListIterator<TSource>((List<TSource>) source, predicate);
    }
    return new WhereEnumerableIterator<TSource>(source, predicate);
}


private class WhereListIterator<TSource> : Enumerable.Iterator<TSource>
{
    // Fields
    private List<TSource>.Enumerator enumerator;
    private Func<TSource, bool> predicate;
    private List<TSource> source;

    // Methods
    [TargetedPatchingOptOut("Performance critical to inline this type of
                             method across NGen image boundaries")]
    public WhereListIterator(List<TSource> source, Func<TSource, bool> predicate)
    {
        this.source = source;
        this.predicate = predicate;
    }

    public override Enumerable.Iterator<TSource> Clone()
    {
        return new Enumerable.WhereListIterator<TSource>(this.source,
                                                         this.predicate);
    }

    public override bool MoveNext()
    {
        switch (base.state)
        {
            case 1:
                this.enumerator = this.source.GetEnumerator();
                base.state = 2;
                break;

            case 2:
                break;

            default:
                goto Label_0069;
        }
        while (this.enumerator.MoveNext())
        {
            TSource current = this.enumerator.Current;
            if (this.predicate(current))
            {
                base.current = current;
                return true;
            }
        }
        this.Dispose();
    Label_0069:
        return false;
    }

    public override IEnumerable<TResult> Select<TResult>(Func<TSource, TResult> selector)
    {
        return new Enumerable.WhereSelectListIterator<TSource, TResult>(this.source,
                                                                        this.predicate,
                                                                        selector);
    }

    public override IEnumerable<TSource> Where(Func<TSource, bool> predicate)
    {
        return new Enumerable.WhereListIterator<TSource>(this.source,
                   Enumerable.CombinePredicates<TSource>(this.predicate, predicate));
    }
}
```

> The FindAll method of the List<T> class actually constructs a new list object, and adds results to it. The Where extension method for IEnumerable<T> will simply iterate over an existing list and yield an enumeration of the matching results without creating or adding anything (other than the enumerator itself.)

> Given a small set, the two would likely perform comparably. However, given a larger set, Where should outperform FindAll, as the new List created to contain the results will have to dynamically grow to contain additional results. Memory usage of FindAll will also start to grow exponentially as the number of matching results increases, where as Where should have constant minimal memory usage (in and of itself...excluding whatever you do with the results.) 

[*reference*](http://stackoverflow.com/questions/2260220/c-sharp-findall-vs-where-speed)
