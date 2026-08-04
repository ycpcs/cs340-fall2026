---
layout: default
title: "Lab 15: Clojure macros"
---

# Your task

Create Clojure macros are described below.

## `applyn`

This macro should take three arguments: a function, a value, and a *n* (an integer count).  It should return a form which will apply the function to *n* copies of the value.

Example use:

    => (applyn str "HA" 5)
    HAHAHAHAHA
    => (defn splot [] (println "Splot!") 3)
    #'cs340-lab15.core/splot
    => (applyn + (splot) 3)
    Splot!
    Splot!
    Splot!
    9

Note that the way that `applyn` works is subtly different than how a function works.  If `applyn` were a function, each argument would be evaluated exactly once.  As you can see from the second example above, the *val* argument is evaluated *n* times.

Suggestion: you can use the built-in `repeat` function to generate a sequence with multiple copies of a specified value.  E.g.,

{% highlight clojure %}
(repeat n val)
{% endhighlight %}

produces a sequence with *n* copies of *val*.  Also note that using `conj` to add an element to a sequence returned by `repeat` does a prepend, as though the sequence were a list.

## `my-and`

This macro should take 0 or more arguments, which are expressions.  When used, it should evaluate the arguments in order.  If any argument evaluates as falsey (`false` or `nil`), then `false` should be returned, *without evaluating the remaining arguments*.  If all of the arguments evaluate as truthy, `true` should be returned.

The behavior described above is not possible using a function, because a function application would evaluate all of the arguments eagerly.

Restriction: your macro should not use the built-in `and` macro in its expansion.  Instead, use the `if` form.

Because the macro takes a variable number of arguments, it should use the variable parameter list syntax.  Also, you will probably want to write a helper function to generate the expanded form of the macro, since for multiple arguments you will need to generate a nested `if` form.  For example, the application

{% highlight clojure %}
(my-and (foo) (bar) (baz))
{% endhighlight %}

could be expanded into the code

{% highlight clojure %}
(if (not (foo))
  false
  (if (not (bar))
    false
    (if (not (baz))
      false
      true)))
{% endhighlight %}

Your `my-and` macro could look something like:

{% highlight clojure %}
(defmacro my-and [& args]
  (my-and-help args))
{% endhighlight %}

Hint: it may be helpful to use syntax quoting in `my-and-help`.

Example use:

    => (defn ned [] (println "I'm Ned!") true)
    #'cs340-lab15.core/ned
    => (defn ted [] (println "I'm Ted!") false)
    #'cs340-lab15.core/ted
    => (my-and)
    true
    => (my-and (ned) (ned) (ned))
    I'm Ned!
    I'm Ned!
    I'm Ned!
    true
    => (my-and (ned) (ned) (ned) (ted))
    I'm Ned!
    I'm Ned!
    I'm Ned!
    I'm Ted!
    false
    => (my-and (ned) (ted) (ned) (ted))
    I'm Ned!
    I'm Ted!
    false


<!--
## `unless`

This macro is similar to the **if** special form.  Its syntax is

> (**unless** *cond* *if-false* *if-true*)

First, *cond* should be evaluated.  If *cond* yields a false value, *if-false* should be evaluated and returned.  Otherwise, *if-true* should be evaluated and returned.

Note that only one of *if-false* and *if-true* should be evaluated, not both.

Hint: generate an **if** form as a result of the macro.

Example use:

    => (unless (< 5 4) "yip" "yap")
    yip
    => (unless (< 4 5) "yip" "yap")
    yap
    => (defn ned [] (println "I'm Ned!") 44)
    #'cs340-lab15.core/ned
    => (defn ted [] (println "I'm Ted!") 55)
    #'cs340-lab15.core/ted
    => (unless (< 5 4) (ned) (ted))
    I'm Ned!
    44
    => (unless (< 4 5) (ned) (ted))
    I'm Ted!
    55
-->
