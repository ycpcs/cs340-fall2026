---
layout: default
title: "Assignment 4: Calculator Language Interpreter"
---

**Due**: <strike>Friday, Oct 12th</strike> Wednesday, Oct 17th by 11:59 PM

Getting Started
===============

Download [CS340\_Assign04.zip](CS340_Assign04.zip) and unzip it.

Import it into Eclipse **Using File&rarr;Import...&rarr;General&rarr;Existing projects into workspace&rarr;Archive File**.  You should see a project called **CS340\_Assign04** in your Eclipse workspace.

The starting code for the assignment is based on the precedence climbing parser from Lab 6, with some improvements.

## Lexer changes

The lexical analyzer has been changed so that

* Parentheses and exponentiation are supported (changes which you also implemented in Lab 6)
* Arbitrary Java-style identifiers are allowed, not just **a** and **b**.  Also, **fn** is recognized as a keyword (and thus is not considered to be an identifier)
* Arbitrary integer literals are allowed, not just the digits **0** through **9**
* The equals sign (**=**), meaning assignment operator, is a valid token
* A comma (**,**) is a valid token
* Left and right curly braces (**{** and **}**) are valid tokens

## Parser changes

The precedence climbing parser supports exponentiation (higher precedence than <b>*</b> and **/**, and right associative) and **=** for assignment (lower precedence than **+** and **-**, and is right associative).

The grammar is expanded to add support for functions and function calls.  In the productions below, note that *Expression* refers to an arbitrary infix expression handled by the precedence climbing parser, and so should be considered to be the start symbol, even though it isn't on the left hand side of any of the productions.  Also note that <b>int_literal</b> and **identifier** are terminal symbols (tokens), as are the symbols

> **(** **)** **{** **,**

Also note that ε means the empty string.

Here are the grammar productions:

> *PrimaryExpression* &rarr; <b>int_literal</b>
>
> *PrimaryExpression* &rarr; **identifier**
>
> *PrimaryExpression* &rarr; **identifier** **(** *OptArgumentList* **)**
>
> *PrimaryExpression* &rarr; **(** *Expression* **)**
>
> *OptArgumentList* &rarr; *ArgumentList*
>
> *OptArgumentList* &rarr; ε
>
> *ArgumentList* &rarr; *Expression*
>
> *ArgumentList* &rarr; *Expression* **,** *ArgumentList*
>
> *PrimaryExpression* &rarr; *Expression*
>
> *PrimaryExpression* &rarr; **fn** **(** *OptParameterList* **)** **{** *Expression* **}**
>
> *OptParameterList* &rarr; *ParameterList*
>
> *OptParameterList* &rarr; ε
>
> *ParameterList* &rarr; **identifier**
>
> *ParameterList* &rarr; **identifier** **,** *ParameterList*

The parser fully implements all of these productions: you will not need to modify the parser.  However, understanding the grammar rules will be helpful in understanding the structure of the parse trees that your **Interpreter** class will evaluate.

Your Task
=========

Your task is to turn the infix expression grammar we developed in [Lecture 4](../lectures/lecture04.html) and [Lecture 5](../lectures/lecture05.html) and also [Lab 5](../labs/lab05.html) and [Lab 6](../labs/lab06.html) and turn it into a calculator program supporting:

* evaluating arbitrary expressions
* functions and function calls

While not *quite* a full programming language, it will be pretty close, and could be turned into one with a bit of extra work.

## Basic ideas and concepts

The classes provided in the Eclipse project embody some important concepts.

**Interpreter** is the interpreter class.  An instance of **Interpreter** evaluates expressions.

The **Value** interface is the supertype for values.  There are two subclasses: **NumberValue**, representing an integer value, and **FunctionValue** representing a function.

The **ValueType** enumeration is used to distinguish the two different kinds of values.  You can call the **getType** method on any **Value** object to determine its **ValueType**.  If it reports **NUMBER**, then the value is a **NumberValue**.  If it reports **FUNCTION**, it is a **FunctionValue**.

For **NumberValue** objects, you can use the **getNumber** method to find out the integer value.

For **FunctionValue** objects, you get use the **getFunctionParameters** method to get the list of parameter names, and the **getFunctionBody** method to get the function's body expression.

There are fairly detailed javadoc comments for each of these classes and methods.

## Step 1

Your first step should be to add support for literals, variable references, and binary operators other than assignment.

When these features implemented, the program will serve as a basic calculator.  Note that the only variable that will exist is called **theAnswer**, and its value is **42**.

Example session (user input in **bold**):

<pre>
Enter expressions (type 'quit' when finished):
> <b>4*5+3</b>
PLUS
+--TIMES
|  +--PRIMARY
|  |  +--INT_LITERAL("4")
|  +--PRIMARY
|     +--INT_LITERAL("5")
+--PRIMARY
   +--INT_LITERAL("3")
=> 23
> <b>2^(4-(2*1))</b>
EXP
+--PRIMARY
|  +--INT_LITERAL("2")
+--PRIMARY
   +--LPAREN("(")
   +--MINUS
   |  +--PRIMARY
   |  |  +--INT_LITERAL("4")
   |  +--PRIMARY
   |     +--LPAREN("(")
   |     +--TIMES
   |     |  +--PRIMARY
   |     |  |  +--INT_LITERAL("2")
   |     |  +--PRIMARY
   |     |     +--INT_LITERAL("1")
   |     +--RPAREN(")")
   +--RPAREN(")")
=> 4
> <b>theAnswer</b>
PRIMARY
+--IDENTIFIER("theAnswer")
=> 42
> <b>theAnswer / 2</b>
DIVIDES
+--PRIMARY
|  +--IDENTIFIER("theAnswer")
+--PRIMARY
   +--INT_LITERAL("2")
=> 21
</pre>

### Hints

Add code to the `evaluate` method in the **Interpreter** class to handle the following kinds of parse nodes:

* **PRIMARY** (modify suport parenthesized expressions)
* **PLUS**
* **MINUS**
* **TIMES**
* **DIVIDES**
* **EXP**
* **IDENTIFIER**

Note that **IDENTIFIER** nodes are variable references.  To handle them, look up the value of the variable using the `env` parameter, which is an **Environment** object.

Note that handling the **PLUS**, **MINUS**, **TIMES**, **DIVIDES**, and **EXP** node types will involve recursive evaluation of the left and right child subexpressions.

## Step 2

Your second step should be to implement the assignment operator by changing the `evaluate` method to handle **ASSIGN** nodes.

Example session (user input in **bold**):

<pre>
> <b>a = 4</b>
ASSIGN
+--PRIMARY
|  +--IDENTIFIER("a")
+--PRIMARY
   +--INT_LITERAL("4")
=> 4
> <b>b = 5</b>
ASSIGN
+--PRIMARY
|  +--IDENTIFIER("b")
+--PRIMARY
   +--INT_LITERAL("5")
=> 5
> <b>a+b*3</b>
PLUS
+--PRIMARY
|  +--IDENTIFIER("a")
+--TIMES
   +--PRIMARY
   |  +--IDENTIFIER("b")
   +--PRIMARY
      +--INT_LITERAL("3")
=> 19
> <b>theAnswer=43</b>
ASSIGN
+--PRIMARY
|  +--IDENTIFIER("theAnswer")
+--PRIMARY
   +--INT_LITERAL("43")
=> 43
> <b>theAnswer</b>
PRIMARY
+--IDENTIFIER("theAnswer")
=> 43
</pre>

### Hints

Your code should recursively evaluate the right subexpression to find its value.  Then, it should find the identifier in the left subtree.  If there isn't an identifier on the left hand side of the assignment, throw an **EvaluationException**.  If an idenifier is found on the left hand side of the assignment, call the `put` method on the `env` object to bind (assign) the evaluated value to the variable.

If you have a node whose symbol is **IDENTIFIER**, it will have a token which in turn will contain the name of the variable as its lexeme.  So, you can use code like the following to extract the variable name:

{% highlight java %}
Node identNode = ...

String varName = identNode.getToken().getLexeme();
{% endhighlight %}

Also note: the result of evaluating an **ASSIGN** node should be the value found by recursively evaluating the subexpression on the right hand side of the assignment.

## Step 3

The third step is to add support for functions and function calls.

A function is a list of parameter names and a body expression.  The created function will be an instance of **FunctionValue**, and so can be assigned to a variable (in order to give the function a name.)

A function call finds the function associated with the function name, evaluates the list of arguments, creates a new environment (with the current environment as its parent), binds each of the called function's parameter names to its corresponding argument value (in the new environment), and finally evaluates the called function's body expression in the new environment.

Example session (user input in **bold**):

<pre>
> <b>f = fn(x) { x * 2 }</b>
ASSIGN
+--PRIMARY
|  +--IDENTIFIER("f")
+--PRIMARY
   +--FN_KEYWORD("fn")
   +--LPAREN("(")
   +--OPT_PARAMETER_LIST
   |  +--PARAMETER_LIST
   |     +--IDENTIFIER("x")
   +--RPAREN(")")
   +--LBRACE("{")
   +--TIMES
   |  +--PRIMARY
   |  |  +--IDENTIFIER("x")
   |  +--PRIMARY
   |     +--INT_LITERAL("2")
   +--RBRACE("}")
=> &lt;&lt;function&gt;&gt;
> <b>f(3)</b>
PRIMARY
+--IDENTIFIER("f")
+--LPAREN("(")
+--OPT_ARGUMENT_LIST
|  +--ARGUMENT_LIST
|     +--PRIMARY
|        +--INT_LITERAL("3")
+--RPAREN(")")
=> 6
> <b>f(f(5))</b>
PRIMARY
+--IDENTIFIER("f")
+--LPAREN("(")
+--OPT_ARGUMENT_LIST
|  +--ARGUMENT_LIST
|     +--PRIMARY
|        +--IDENTIFIER("f")
|        +--LPAREN("(")
|        +--OPT_ARGUMENT_LIST
|        |  +--ARGUMENT_LIST
|        |     +--PRIMARY
|        |        +--INT_LITERAL("5")
|        +--RPAREN(")")
+--RPAREN(")")
=> 20
> <b>g = fn(y, z) { f(y) - z }</b>
ASSIGN
+--PRIMARY
|  +--IDENTIFIER("g")
+--PRIMARY
   +--FN_KEYWORD("fn")
   +--LPAREN("(")
   +--OPT_PARAMETER_LIST
   |  +--PARAMETER_LIST
   |     +--IDENTIFIER("y")
   |     +--COMMA(",")
   |     +--PARAMETER_LIST
   |        +--IDENTIFIER("z")
   +--RPAREN(")")
   +--LBRACE("{")
   +--MINUS
   |  +--PRIMARY
   |  |  +--IDENTIFIER("f")
   |  |  +--LPAREN("(")
   |  |  +--OPT_ARGUMENT_LIST
   |  |  |  +--ARGUMENT_LIST
   |  |  |     +--PRIMARY
   |  |  |        +--IDENTIFIER("y")
   |  |  +--RPAREN(")")
   |  +--PRIMARY
   |     +--IDENTIFIER("z")
   +--RBRACE("}")
=> &lt;&lt;function&gt;&gt;
> <b>g(8, 3)</b>
PRIMARY
+--IDENTIFIER("g")
+--LPAREN("(")
+--OPT_ARGUMENT_LIST
|  +--ARGUMENT_LIST
|     +--PRIMARY
|     |  +--INT_LITERAL("8")
|     +--COMMA(",")
|     +--ARGUMENT_LIST
|        +--PRIMARY
|           +--INT_LITERAL("3")
+--RPAREN(")")
=> 13
</pre>

### Hints

To evaluate a function:

1. Gather all of the parameter names into a **List**
2. Create a **FunctionValue** object with the collected parameter list and the body expression as its body

The result of evaluating a function is the **FunctionValue**.

Note that a traversal of the **OPT\_PARAMETER\_LIST** subtree will be needed to collect all of the parameter names.

To evaluate a function call:

1. Recursively evaluate the identifier naming the function to find its value (which should be a **FunctionValue**)
2. Gather all of the argument values (storing them in a **List**)
3. Create a new **Environment** with the current environment (`env`) as its parent
4. Use the **put** method to bind (assign) each argument value to its corresponding parameter
5. Return the result of recursively evaluating the called function's body in the new environment

See [Lab 9](../labs/lab09.html) for more hints related to functions and function calls.

# Grading

The grading for the standard features is as follows:

* Evaluate expressions with literal values: 40%
* Variable assignment: 5%
* Variable references: 10%
* Functions with a single parameter: 10%
* Calls to functions with a single parameter: 20%
* Functions with 0 or multiple parameters: 5%
* Calls to functions with 0 or multiple parameters: 10%

## Extra credit

For up to 40 points extra credit, implement

* comparisons (less than, greater than, equality, inequality)
* **if**/**else** expressions

These will require changes to the lexer, parser, and interpreter.

<div class="callout">
<b>Important</b>: extra credit features must be 100% your own work.
</div>

Example:

<pre>
&gt; <b>fib = fn(n) { if (n &lt; 2) { n } else { fib(n-2) + fib(n-1) } }</b>
ASSIGN
+--PRIMARY
|  +--IDENTIFIER("fib")
+--PRIMARY
   +--FN_KEYWORD("fn")
   +--LPAREN("(")
   +--OPT_PARAMETER_LIST
   |  +--PARAMETER_LIST
   |     +--IDENTIFIER("n")
   +--RPAREN(")")
   +--LBRACE("{")
   +--PRIMARY
   |  +--IF_KEYWORD("if")
   |  +--LPAREN("(")
   |  +--LESS_THAN
   |  |  +--PRIMARY
   |  |  |  +--IDENTIFIER("n")
   |  |  +--PRIMARY
   |  |     +--INT_LITERAL("2")
   |  +--RPAREN(")")
   |  +--LBRACE("{")
   |  +--PRIMARY
   |  |  +--IDENTIFIER("n")
   |  +--RBRACE("}")
   |  +--ELSE_KEYWORD("else")
   |  +--LBRACE("{")
   |  +--PLUS
   |  |  +--PRIMARY
   |  |  |  +--IDENTIFIER("fib")
   |  |  |  +--LPAREN("(")
   |  |  |  +--OPT_ARGUMENT_LIST
   |  |  |  |  +--ARGUMENT_LIST
   |  |  |  |     +--MINUS
   |  |  |  |        +--PRIMARY
   |  |  |  |        |  +--IDENTIFIER("n")
   |  |  |  |        +--PRIMARY
   |  |  |  |           +--INT_LITERAL("2")
   |  |  |  +--RPAREN(")")
   |  |  +--PRIMARY
   |  |     +--IDENTIFIER("fib")
   |  |     +--LPAREN("(")
   |  |     +--OPT_ARGUMENT_LIST
   |  |     |  +--ARGUMENT_LIST
   |  |     |     +--MINUS
   |  |     |        +--PRIMARY
   |  |     |        |  +--IDENTIFIER("n")
   |  |     |        +--PRIMARY
   |  |     |           +--INT_LITERAL("1")
   |  |     +--RPAREN(")")
   |  +--RBRACE("}")
   +--RBRACE("}")
=&gt; &lt;&lt;function&gt;&gt;
&gt; <b>fib(0)</b>
PRIMARY
+--IDENTIFIER("fib")
+--LPAREN("(")
+--OPT_ARGUMENT_LIST
|  +--ARGUMENT_LIST
|     +--PRIMARY
|        +--INT_LITERAL("0")
+--RPAREN(")")
=&gt; 0
&gt; <b>fib(1)</b>
PRIMARY
+--IDENTIFIER("fib")
+--LPAREN("(")
+--OPT_ARGUMENT_LIST
|  +--ARGUMENT_LIST
|     +--PRIMARY
|        +--INT_LITERAL("1")
+--RPAREN(")")
=&gt; 1
&gt; <b>fib(6)</b>
PRIMARY
+--IDENTIFIER("fib")
+--LPAREN("(")
+--OPT_ARGUMENT_LIST
|  +--ARGUMENT_LIST
|     +--PRIMARY
|        +--INT_LITERAL("6")
+--RPAREN(")")
=&gt; 8
&gt; <b>fib(7)</b>
PRIMARY
+--IDENTIFIER("fib")
+--LPAREN("(")
+--OPT_ARGUMENT_LIST
|  +--ARGUMENT_LIST
|     +--PRIMARY
|        +--INT_LITERAL("7")
+--RPAREN(")")
=&gt; 13
</pre>

# Submitting

When you are done, submit the assignment to the Marmoset server using one of the methods below.

## From Eclipse

If you have the [Simple Marmoset Uploader Plugin](../resources/index.html) installed, select the project (**CS340\_Assign04**) in the package explorer and then press the blue up arrow button in the toolbar. Enter your Marmoset username and password when prompted.

This is the recommended way to submit your work.

## From a web browser

Save the project (**CS340\_Assign04**) to a zip file by right-clicking it and choosing

> **Export...&rarr;Archive File**

Upload the saved zip file to the Marmoset server as **assign04**. The server URL is

> <https://cs.ycp.edu/marmoset/>

Use this method only if there is some reason why you can't use the plugin.

<!-- vim:set wrap: ­-->
<!-- vim:set linebreak: -->
<!-- vim:set nolist: -->
