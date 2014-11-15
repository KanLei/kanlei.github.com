---
layout: post
title: "Expression Evaluation"
description: ""
category: "Article"
tags: [c#,datastructure]
---
{% include JB/setup %}

## Postfix Exrepssion ##

Postfix Expression(aka Reverse Polish Epxression), it put the operators behind the operands. And we do not require parentheses. We always use Infix Expressoin in our life. This will cause a bit difficult to figure out. Enough with the chatter，let's see what is a 「Postfix Expression」(Polish Expression).

    2 2 *
    4 2 2 * +
    4 2 2 * + 1 3 * -
    2 4 2 2 * + 1 3 * - ^
    3 2 4 2 2 * + 1 3 * - ^ *
    3 2 4 2 2 * + 1 3 * - ^ * 5 -

 Let us change the Postfix Expression to Infix Expression. Like this:

	2 * 2
	4 + 2 * 2
	4 + 2 * 2 - 1 * 3
	2 ^ ( 4 + 2 * 2 - 1 * 3 )
	3 * 2 ^ ( 4 + 2 * 2 - 1 * 3 )
	3 * 2 ^ ( 4 + 2 * 2 - 1 * 3 ) - 5

Postfix Expression is difficult to understand, but it is easier to implementation. We merely need one stack to save the operand. Think about this:

> 1. read the expression from left to right
2. push it to stack when read an operand
3. when read an operator, pop two operands from the stack and use the operator performs arithmetic
4. push the result value into the stack
5. execute 1~4

```c#
/// <summary>
/// 计算后缀表达式(逆波兰表达式)
/// 如: 3 2 4 2 2 * + 1 3 * - ^ * 5 -
/// </summary>
/// <param name="text">表达式</param>
/// <returns>表达式的结果</returns>
static int CalculatorPostfixExpression(string text)
{
    var dicts = new Dictionary<string, Func<int, int, int>>();
    dicts.Add("+", (x, y) => x + y);
    dicts.Add("-", (x, y) => x - y);
    dicts.Add("*", (x, y) => x * y);
    dicts.Add("/", (x, y) => x / y);
    dicts.Add("%", (x, y) => x % y);
    dicts.Add("^", (x, y) => (int)Math.Pow(x, y));


    var s = new Stack<int>();  // 只入栈操作数

    int ope;
    foreach (var item in text.Split(' '))
    {
        if (int.TryParse(item, out ope))
        {
            s.Push(ope);
        }
        else if (dicts.ContainsKey(item))
        {
            int op1 = s.Pop();
            int op2 = s.Pop();
            s.Push(dicts[item].Invoke(op2, op1));
        }
        else
        {
            throw new ArgumentOutOfRangeException(item, "operator not exist");
        }
    }
    return s.Pop();
}
```

## Prefix Expression ##

Prefix Expression is similar to the Postfix Expression, we do not need parentheses, but it is a little  more complexity than Prefix Expression. Because we need a stack of string type, we need to push operators and operands to the stack, yet.

	 * 2 2
	 + 4 * 2 2
	 - + 4 * 2 2 * 1 3
	 ^ 2 - + 4 * 2 2 * 1 3
	 * 3 ^ 2 - + 4 * 2 2 * 1 3
	 - * 3 ^ 2 - + 4 * 2 2 * 1 3 5

Step by step for coding:

> 1. read the expression from left to right
2. push it to stack when read an operator
3. when read an operand, if the top element of the stack is an operand, pop the operand and the operator, then performs arithmetic; otherwise push the operand into the stack
4. push the result value into the stack
5. execute 1~4

```c#
/// <summary>
/// 计算前缀表达式(波兰表达式)
/// 如: - * 3 ^ 2 - + 4 * 2 2 * 1 3 5
/// </summary>
/// <param name="text">表达式</param>
/// <returns>表达式的结果</returns>
static int CalculatorPrefixExpression(string text)
{
    var dicts = new Dictionary<string, Func<int, int, int>>();
    dicts.Add("+", (x, y) => x + y);
    dicts.Add("-", (x, y) => x - y);
    dicts.Add("*", (x, y) => x * y);
    dicts.Add("/", (x, y) => x / y);
    dicts.Add("%", (x, y) => x % y);
    dicts.Add("^", (x, y) => (int)Math.Pow(x, y));

    var s = new Stack<string>();  // 操作数和运算符都要入栈

    int ope;
    foreach (var item in text.Split(' '))
    {
        if (int.TryParse(item, out ope))
        {
            int ope2;
            while (s.Count > 0 && int.TryParse(s.Peek(), out ope2))  // 如果栈
            {
                ope2 = int.Parse(s.Pop());
                string opt = s.Pop();
                ope = dicts[opt].Invoke(ope2, ope);
            }
            s.Push(ope.ToString());
        }
        else if (dicts.ContainsKey(item))
        {
            s.Push(item);
        }
        else
        {
            throw new ArgumentOutOfRangeException(item, "operator not exist");
        }
    }
    return int.Parse(s.Pop());
}
```

You will see the Prefix Expression when you using a program language called `LISP`.

## Infix Expression ##

Last but not least. Infix Expression is the most difficult to implementation than the others above. With Infix Expression, we should think about the parentheses and we should need two stacks; One for operands and another for operators. Also, we should consider the operators' priorities to push into stack or pop out of stack for performing arithmetic.

We all know the Infix Expression, just to mention the expression.

	3 * 2 ^ ( 4 + 2 * 2 - 1 * 3 ) - 5

Now let us begin to design the priority for the operators:

<table border="1" style="border: green 2px solid;border-collapse:collapse;text-align:center" >
<tr>
<td>operator</td>
<td>priority in stack</td>
<td>priority out of stack</td>
<tr>
<tr>
<td>+ -</td>
<td>1</td>
<td>1</td>
</tr>
<tr>
<td>* / %</td>
<td>2</td>
<td>2</td>
</tr>
<tr>
<td>^</td>
<td>3</td>
<td>4</td>
</tr>
<tr>
<td>(</td>
<td>0</td>
<td>4</td>
</tr>
<tr>
<td>)</td>
<td>-1</td>
<td>-1</td>
</tr>
</table>


Translate to code, we can use Type Dictionary<Tkey, Tvalue> store operator and priority in C#:

```c#
// 栈内操作符的优先级
var inStackPriority = new Dictionary<string, int>() 
{
    { "+", 1 }, { "-", 1 }, { "*", 2 }, { "/", 2 }, 
    { "%", 2 }, { "^", 3 }, { "(", 0 }, { ")", -1}
};
// 栈外操作符的优先级
var outofStackPriority = new Dictionary<string, int>()
{
    { "+", 1 }, { "-", 1 }, { "*", 2 }, { "/", 2 }, 
    { "%", 2 }, { "^", 4 }, { "(", 4 }, { ")", -1 }
};
```

Step by step for coding:

> 1. read the expression from left to right
2. push it into stack when read an `operand`
3. when read an `operator` except `")"`, if the priority of the operator is higher than the top operator's in the operator stack, push it into the operator stack.Or pop two operands from the operand stack, then performs arithmetic; Finally, push the result value into the operand stack.
4. When read a ")", does not push it into the operator stack. Instead of performing arithmetic until pop the "(".
5. execute 1~4

Okay, now it's coding time...

```c#
/// <summary>
/// 计算中缀表达式
/// 如: 3 * 2 ^ ( 4 + 2 * 2 - 1 * 3 ) - 5
/// </summary>
/// <param name="text"></param>
/// <returns></returns>
static int CalculatorInfixExpression(string text)
{
    var dicts = new Dictionary<string, Func<int, int, int>>();
    dicts.Add("+", (x, y) => x + y);
    dicts.Add("-", (x, y) => x - y);
    dicts.Add("*", (x, y) => x * y);
    dicts.Add("/", (x, y) => x / y);
    dicts.Add("%", (x, y) => x % y);
    dicts.Add("^", (x, y) => (int)Math.Pow(x, y));

    // 栈内操作符的优先级
    var inStackPriority = new Dictionary<string, int>() 
    {
        { "+", 1 }, { "-", 1 }, { "*", 2 }, { "/", 2 }, 
        { "%", 2 }, { "^", 3 }, { "(", 0 }, { ")", -1}
    };
    // 栈外操作符的优先级
    var outofStackPriority = new Dictionary<string, int>() 
    {
        { "+", 1 }, { "-", 1 }, { "*", 2 }, { "/", 2 }, 
        { "%", 2 }, { "^", 4 }, { "(", 4 }, { ")", -1 }
    };



    var stackOperand = new Stack<int>();
    var stackOperator = new Stack<string>();

    stackOperator.Push("(");  // 栈中预设最低级运算符
    foreach (var item in text.Split(' '))
    {
        int ope;
        if (int.TryParse(item, out ope))
        {
            stackOperand.Push(ope);
        }
        else if (inStackPriority.ContainsKey(item))
        {
            // 栈外运算符比栈顶运算符优先级高则入栈
            int priority = outofStackPriority[item];
            if (priority > inStackPriority[stackOperator.Peek()])
            {
                stackOperator.Push(item);
            }
            else
            {
                if (item == ")")
				{
                    do
                    {
						int op1 = stackOperand.Pop();
						int op2 = stackOperand.Pop();

                        stackOperand.Push(dicts[stackOperator.Pop()].Invoke(op2, op1));
                    } while (stackOperator.Peek() != "(");

                    stackOperator.Pop();
                }
                else
                {
					do
                    {
                        int op1 = stackOperand.Pop();
                        int op2 = stackOperand.Pop();

                        stackOperand.Push(dicts[stackOperator.Pop()].Invoke(op2, op1));
                    } while (priority <= inStackPriority[stackOperator.Peek()]);

                    stackOperator.Push(item);
                }
            }
        }
        else
        {
            throw new ArgumentOutOfRangeException(item, "operator not exist");
        }
    }

    // 将剩余的操作符(除 "(" 外)全部出栈
    while (stackOperator.Count > 0 && stackOperator.Peek() != "(")
    {
        int op1 = stackOperand.Pop();
        int op2 = stackOperand.Pop();

        stackOperand.Push(dicts[stackOperator.Pop()].Invoke(op2, op1));
    }
    return stackOperand.Pop();
}
```

## Optimeze the Infix Expression ##

Look at the above code between `else if` clause:

```c#
int op1 = stackOperand.Pop();
int op2 = stackOperand.Pop();

stackOperand.Push(dicts[stackOperator.Pop()].Invoke(op2, op1));`
```

These codes are duplicated. Let us modify the condition and delete the duplicated code.

*Finally version*

```c#
bool isLeftBracket;
do
{
    int op1 = stackOperand.Pop();
    int op2 = stackOperand.Pop();

    stackOperand.Push(dicts[stackOperator.Pop()].Invoke(op2, op1));
} while (isLeftBracket = item == ")" ? stackOperator.Peek() != "(" :
    priority <= inStackPriority[stackOperator.Peek()]);

if (item == ")")
{
    stackOperator.Pop();
}
else
{
    stackOperator.Push(item);
}
```
