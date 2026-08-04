---
layout: default
title: "Lab 9: Calculator functions"
---

# Your task

Note that you do not need to get a sign-off on this lab.

Continue working on [Assignment 4](../assign/assign04.html).

Your goal should be to implement functions and function calls.  Note that you will need to have variable assignment working, since the only way to call a function is to name the variable whose value is the function.

## Accessing parameter(s) and argument(s)

To simplify the task, you can start by assuming that each function will have a single parameter.  (Eventually, you should generalize your code so that any number of parameters/arguments are supported.)  When creating a function, you will need to think about how to access the identifier naming the first parameter.  When a function is called, you will need to think about how to access the expression which computes the first argument value.

Here's an example of running the Calculator and defining a function:

<pre>
> <b>add1 = fn(x) { x + 1 }</b>
ASSIGN
+--PRIMARY
|  +--IDENTIFIER("add1")
+--PRIMARY
   +--FN_KEYWORD("fn")
   +--LPAREN("(")
   +--OPT_PARAMETER_LIST
   |  +--PARAMETER_LIST
   |     +--IDENTIFIER("x")
   +--RPAREN(")")
   +--LBRACE("{")
   +--PLUS
   |  +--PRIMARY
   |  |  +--IDENTIFIER("x")
   |  +--PRIMARY
   |     +--INT_LITERAL("1")
   +--RBRACE("}")
=> &lt;&lt;function&gt;&gt;
</pre>

Notice that the third child of the `PRIMARY` node representing the function is an `OPT_PARAMETER_LIST`.  Its first child is a `PARAMETER_LIST`, and in turn, its first child is an `IDENTIFIER` naming the function's parameter.  You could retrieve the parameter name as follows:

{% highlight java %}
Node fnPrimary = ...
Node optParamList = fnPrimary.getChildren().get(2);
Node paramList = optParamList.getChildren().get(0);
Node firstParam = paramList.getChildren().get(0);
{% endhighlight %}

This code assumes that the variable <code>fnPrimary</code> refers to the `PRIMARY` node representing the function.

Here's an example of calling this function:

<pre>
> add1(4)
PRIMARY
+--IDENTIFIER("add1")
+--LPAREN("(")
+--OPT_ARGUMENT_LIST
|  +--ARGUMENT_LIST
|     +--PRIMARY
|        +--INT_LITERAL("4")
+--RPAREN(")")
=> 5
</pre>

The structure of a function call is quite similar to a function, except that instead of an `OPT_PARAMETER_LIST`, there is an `OPT_ARGUMENT_LIST`.  So, to get the expression computing the value of the first argument, you could use code like:

{% highlight java %}
Node funcCall = ...
Node optArgList = funcCall.getChildren().get(2);
Node argList = optArgList.getChildren().get(0);
Node firstArg = argList.getChildren().get(0);
{% endhighlight %}

## Creating a `FunctionValue`

The result of evaluating a function expression should be a `FunctionValue`.

A `FunctionValue` is

* A `List` of parameter names
* A body expression

## Calling a function

To call a function:

1. A new `Environment` should be created whose parent environment is the current environment: this is the *function call environment*
2. Each argument expression should be evaluated and assigned to the corresponding function parameter
3. The function's body expression should be evaluated in the function call environment

<!-- vim:set wrap: ­-->
<!-- vim:set linebreak: -->
<!-- vim:set nolist: -->
