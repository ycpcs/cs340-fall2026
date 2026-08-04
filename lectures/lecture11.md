---
layout: default
title: "Lecture 11: Interpretation of functions and function calls"
---

# Variables

In [Assignment 4](../assign/assign04.html), variables are managed by the `Environment` class.  An `Environment` object maintains a map of variable names to a `Value`.  When a variable reference is encountered, the interpreter can call the `lookup` method on the current `Environment` object to retrieve the named variable's `Value`.  To execute an assignment, the interpreter can call the `put` method on the current `Environment` object to "bind" the value on the right hand side of the assignment to the variable named on the left hand side of the assignment.

For example, the expression

    a = 3

would assign the computed value 3 to the variable called **a**, in the "global" environment.  The subsequent expression

    a * 2

would yield the value 6, since the variable reference to **a** would cause the previously assigned value of 3 to be retrieved, and then that value would be multiplied by 6.

# Functions

In the Assignment 4 interpreter, a function consists of

* 0 or more *parameters*
* an expression which is the function's *body*

Functions are values, and can be assigned to variables, which is how the Calculator language allows the user to give names to functions.  Here is an example function

    fn (x) { x * 2 }

This function has one parameter, called `x`, and it has `x * 2` as its body expression.  Let's give the function a name by assigning it to a variable named `mul2`:

    mul2 = fn (x) { x * 2 }

A function call expression passes the value of 0 or more argument expressions to be assigned to the function's parameters in a new `Environment`, and then evaluates the function's body expression.  So, for example, the function call

    mul2(3)

would yield the result 6 because

* the argument expression 3 will be assigned to the parameter `x`
* when the body expression `x * 2` is evaluated, the result is 6

Internally within the interpreter, a function is represented as a `FunctionValue` object, which is simply a list of the parameter names, and a `Node` which is the root of the function's body expression.

# Scopes

Let's consider a slightly more interesting series of expressions.  Assume that the following expressions are evaluated in order:

    b = 3
    
    f = fn(x) { x * b }
    
    f(2)
    
    b = 4
    
    f(2)

Note that two variables are referenced in the body of the function: `x` and `b`.  Since `x` is a parameter, its value will always be determined by the argument passed to the function.  However, `b` is a *free variable*, meaning that its value is determined by a definition elsewhere in the program.  The first time `f(2)` is evaluated, the result is 6, because at that point, `b` has the value 3, as defined in the global environment.  The second time `f(2)` is evaluated, the value of `b` has been changed to 4, so the result is 8.

The term *scope* refers to a region of a program in which a particular variable or variables are defined.  In the Calculator language, the body of a function is a scope in which the function's parameter variables are defined.  Outside that scope, those variables don't exist.  The *global scope* of the interpreter is the "outermost" scope, and encompasses the entire "program" (meaning all expressions that are evaluated).

Let's look at one more example, which illustrates something important about functions:

    x = 17
    
    f = fn(x) { x }
    
    f(42)

What happens when `f(42)` is evaluated?  Is the result 42 &mdash; the value of the argument to the function &mdash; or is the result 17 &mdash; the value of the variable `x` in the global scope?

The answer is 42, for the following reason: when a variable is defined in an "inner" scope, it "hides" the definition of any identically-named variable in an "outer" scope.  So, within the body of the function, `x` refers to the function's parameter, not the global variable.

One last example:

    x = 17

    f = fn(x) { x = 42 }

    f(99)

    x

Some things to note:

* The result of an assignment expression is the result computed by the right hand side of the assignment
* The argument passed to `f` is ignored, since its body immediately assigns 42 to `x`

The question is: what is the value of the global variable `x` after the function call?  Is it 42 or 17?  In this case, the result is 17.  The explanation is that the value of the global variable `x` doesn't change because within the body of the function, `x` refers to the parameter, not the global variable.

# Implementing function calls

In the interpreter, function calls should be evaluated as follows:

1. A new `Environment` should be created whose parent environment is the current environment: this is the *function call environment*
2. Each argument expression should be evaluated and assigned to the corresponding function parameter
3. The function's body expression should be evaluated in the function call environment

This approach ensures that parameters will take precedence over identically-named variables elsewhere in the program.  But, because the function call environment has a link to the parent environment, free variables in the body of the function will resolve to their definition in whichever outer scope is appropriate.

<!-- vim:set wrap: ­-->
<!-- vim:set linebreak: -->
<!-- vim:set nolist: -->
