---
layout: post
title: "Calcute one's age"
description: "Calcute one's age in C#"
category: "SnippyCode"
tags: [c#, stackoverflow]
---
{% include JB/setup %}


``` c#
// DateTime birthday = DateTime.Parse("1990/12/23");
static int GetAge(DateTime birthday)
{
    DateTime today = DateTime.Today; 
    int age = today.Year - birthday.Year;

    if (birthday > today.AddYears(-age))
        --age;
    return age;
}
```

[*reference*](http://stackoverflow.com/questions/9/how-do-i-calculate-someones-age-in-c)
