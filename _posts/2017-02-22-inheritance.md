---
author: kimmy
layout: post
title:  "Inheritance"
date:   2017-02-22 21:37:00 +08:00
categories: Python
tags:
- Python
- Notes
- dict
---


* content
{:toc}




## Subclassing built-in Types is Tricky

### Derive from `dict`


{% highlight python %}
class DoubleDict(dict):
    def __setitem__(self, key, value):
        super().__setitem__(key, [value] * 2)
{% endhighlight %}


{% highlight python %}
dd = DoubleDict(one=1)
dd
{% endhighlight %}




    {'one': 1}




{% highlight python %}
dd['two'] = 2
dd
{% endhighlight %}




    {'one': 1, 'two': [2, 2]}




{% highlight python %}
dd.update(three=3)
dd
{% endhighlight %}




    {'one': 1, 'three': 3, 'two': [2, 2]}



+ `DoubleDict.__setitem__` duplicates values when storing. It works by delegating to the superclass
+ `__init__` method inherited from `dict` is IGNORED:<br>
    the value of 'one' is not duplicated.
+ `[]` operator calls `__setitem__`:<br>
    'two' maps to the duplicated value [2, 2].
+ `update` method from `dict` does not use `DoubleDict.__setitem__` either:<br>
    the value of 'three' was not duplicated.

Subclassing built-in types like `dict` or `list` or `str`  is errorprone because the built-in methods mostly ignore user-defined overrides.

### Derive from `collections`


{% highlight python %}
import collections
class DoubleDict2(collections.UserDict):
    def __setitem__(self, key, value):
        super().__setitem__(key, [value] * 2)
{% endhighlight %}


{% highlight python %}
dd = DoubleDict2(one=1)
dd
{% endhighlight %}




    {'one': [1, 1]}




{% highlight python %}
dd['two'] = 2
dd
{% endhighlight %}




    {'one': [1, 1], 'two': [2, 2]}




{% highlight python %}
dd.update(three=3)
dd
{% endhighlight %}




    {'one': [1, 1], 'two': [2, 2], 'three': [3, 3]}



Note the problem described above only affects: <br>
+ methods delegation within the C language implementation of the built-in types,
+ and userdefined classes derived directly from those types.

### KEYPOINT
---
1. When we need a custom `list`, `dict`, or `str` type, <br>
    it’s easier to subclass `UserList`, `UserDict`, or `UserString`—all defined in the `collections` module.
2. If the desired behavior is very different from what the built-ins offer, <br>
    it may be easier to subclass the appropriate ABC from `collections.abc` and write your own implementation.

## Multiple Inheritance and Method Resolution Order (MRO)

Classes have an attribute called ``__mro__`` holding a tuple of references <br>
to the superclasses in MRO order, from the current class all the way to the object class.


{% highlight python %}
def print_mro(cls):
    print(','.join(c.__name__ for c in cls.__mro__))
{% endhighlight %}


{% highlight python %}
import numbers
print_mro(numbers.Integral)
{% endhighlight %}

    Integral,Rational,Real,Complex,Number,object



{% highlight python %}
import tkinter
print_mro(tkinter.Text)
{% endhighlight %}

    Text,Widget,BaseWidget,Misc,Pack,Place,Grid,XView,YView,object



> Reference from Fluent Python Chapter 12
