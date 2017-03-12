---
layout: post
title:  Functions
date:   2017-03-12 16:31:47 +08:00
categories: Python
tags:
- partial
- positional argument
- keyword argument
---

* content
{:toc}


## Writing Functions That Accept Any Number of Arguments

### `*`
accepts any number of **positional** arguments.


{% highlight python %}
def avg(first, *rest):
    return (first + sum(rest)) / (1 + len(rest))
{% endhighlight%}


{% highlight python %}
avg(1, 2)
{% endhighlight%}




    1.5




{% highlight python %}
avg(1, 2, 3, 4)
{% endhighlight%}




    2.5



### `**`
accepts any number of **keyword** arguments.<br>
e.g.
{% highlight python %}
tf.contrib.layers.stack(inputs, layer, stack_args, **kwargs)

{% endhighlight%}
`stack` allows you to repeatedly apply the same operation with different arguments stack_args[i].
{% highlight python %}
y = stack(x, fully_connected, [32, 64, 128], scope='fc')
# It is equivalent to:

x = fully_connected(x, 32, scope='fc/fc_1')
x = fully_connected(x, 64, scope='fc/fc_2')
y = fully_connected(x, 128, scope='fc/fc_3')
{% endhighlight%}

### `*` and `**`
can accept both any number of positional and keyword-only arguments.


{% highlight python %}
def anyargs(*args, **kwargs):
    # positional arguments are placed into a **tuple**
    print(args)
    # keyword arguments are placed into a **dictionary**
    print(kwargs)
{% endhighlight%}

### Discussion
To accept keyword-only arguments:<br>
place the keyword arguments after a `*` argument or a single unnamed `*` for greater code clarity.

## Returning Multiple Values from a Function

### `tuple`


{% highlight python %}
def foo():
    # Notice the comma forms a tuple
    return 1, 2, 3
{% endhighlight%}


{% highlight python %}
r = foo()
r
{% endhighlight%}




    (1, 2, 3)



## Defining Functions with Default Arguments

### common immutable objects

such as `None`, `True`, `False`, `numbers`, or `strings`.
+ Assign values in the definition.
+ Make sure that default arguments appear last.


{% highlight python %}
def foo(a, b=7):
    print(a, b)
{% endhighlight%}


{% highlight python %}
foo(1)
{% endhighlight%}

    1 7



{% highlight python %}
foo(1, 2)
{% endhighlight%}

    1 2


### mutable containers

such as a list, set, or dictionary, use `None` as the default.


{% highlight python %}
# Using a list as a default value
def foo(a, b=None):
    if b is None:
        # assign a empty list
        b = []
{% endhighlight%}

Otherwise, the default value might get modified accidently.


{% highlight python %}
def foo(a, b=[]):
    print(b)
    return b
{% endhighlight%}


{% highlight python %}
x = foo(1)
x
{% endhighlight%}

    []

    []


{% highlight python %}
x.append([1, 2, 3])
x
{% endhighlight%}




    [[1, 2, 3]]




{% highlight python %}
# changes will permanently alter the default value!
foo(100)
{% endhighlight%}

    [[1, 2, 3]]

    [[1, 2, 3]]


## Making an N-Argument Callable Work As a Callable with Fewer Arguments

### `functools.partial()`


{% highlight python %}
# Compute the distance between two points
import math
def distance(p1, p2):
    x1, y1 = p1
    x2, y2 = p2
    return math.hypot(x2 - x1, y2 - y1)
{% endhighlight%}

Sort points according to the distance from some other point.
+ `sort()` method of lists accpets a `key` argument, but it only works with functions that **take a single argument**.
+ `partial()` could fix one argument to make `distance` suitable.


{% highlight python %}
from functools import partial
original_p = (0, 0)
points = [(1, 2), (3, 4), (5, 6), (7, 8)]
points.sort(key=partial(distance, original_p))
points
{% endhighlight%}




    [(1, 2), (3, 4), (5, 6), (7, 8)]

<br><br><br>
> Take Reference from [Python Cookbook](http://shop.oreilly.com/product/0636920027072.do)<br><br>
> CHAPTER 7<br>
> Functions
