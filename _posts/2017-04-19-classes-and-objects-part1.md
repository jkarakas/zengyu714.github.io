---
author: kimmy
layout: post
title:  "Treat Functions as Objects Part1"
date:   2017-04-19 23:49:23 +08:00
categories: Python
tags:
- Python
- __init__()
- lazyproperty
---

* content
{:toc}



## Changing the String Representation of Instances

### `__str()__`  /  `__repr()__`


{% highlight python %}
class Pair:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    def __repr__(self):
        return 'Pair({0.x!r}, {0.y!r})'.format(self)
    def __str__(self):
        return '({0.x!s}, {0.y!s})'.format(self)
{% endhighlight %}

Format code `{0.x}` specifies the x-attribute of argument 0, actually is the instance `self`. <br>
Or equivalently use `return 'Pair(%r, %r)' % (self.x, self.y)`

## Saving Memory When Creating a Large Number of Instances

###  `__slots__`


{% highlight python %}
class Date:
    __slots__ = ['year', 'month', 'day']
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day
{% endhighlight %}

+ Use a **compact** internal representation for instances.
+ Can not add new attributes to instances.

## Encapsulating Names in a Class

+ single leading underscore `_`, indicates **internal** implementation.
+ two leading underscores `__`, on names within class definitions.<br>


{% highlight python %}
class B:

    def __init__(self):
        self.__private = 0

    def __private_method(self):
        pass

    def public_method(self):
        self.__private_method()

class C(B):

    def __init__(self):
        super().__init__()
        self.__private = 1       # does not override B.__private

    def __private_method(self):  # does not override B.__private_method()
        pass
{% endhighlight %}

Here, the private names `__private` and `__private_method` get **renamed** to `_C__private` and `_C__private_method`.

### Discussion

1. Prefer to use single underscore `_` unless the code involve **subclassing** and there are **internal attributes** that should be hidden from subclasses, then use the double underscore `__` instead.
2. Sometimes, use a single **trailing** underscore to avoid clash with keywords. E.g. `lambda_ = 2.0`

## Calling a Method on a Parent Class

### `super()`


{% highlight python %}
class A:

    def spam(self):
        print('A.spam')

class B(A):

    def spam(self):
        print('B.spam')
        super().spam()     # Call parent spam()
{% endhighlight %}

1. `super().__init__()` to make sure that parents are properly initialized.
2. `super().__setattr__(name, value)` overrides any of Python’s **special** methods.

### Discussion

`super()` works even though there is no explicit base class listed.

## Using Lazily Computed Properties


{% highlight python %}
class lazyproperty:

    def __init__(self, func):
        self.func = func

    def __get__(self, instance, cls):
        if instance is None:
            return self
        else:
            value = self.func(instance)
            setattr(instance, self.func.__name__, value)
            return value
{% endhighlight %}


{% highlight python %}
import math
class Circle:

    def __init__(self, radius):
        self.radius = radius

    @lazyproperty
    def area(self):
        print('Computing area')
        return math.pi * self.radius ** 2

    @lazyproperty
    def perimeter(self):
        print('Computing perimeter')
        return 2 * math.pi * self.radius
{% endhighlight %}


{% highlight python %}
c = Circle(4.0)
c.radius

print(c.area)
print(c.area)  # “Computing area” only appear once.

print(c.perimeter)
print(c.perimeter)  # “Computing perimeter” only appear once.
{% endhighlight %}

    Computing area
    50.26548245743669
    50.26548245743669
    Computing perimeter
    25.132741228718345
    25.132741228718345


### Discussion

1. `lazyproperty` is a **read-only** attribute as a property that only gets computed on access.
2. When a descriptor is placed into a class definition, its `__get__()`, `__set__()`, and `__delete__()` methods get triggered on attribute access. However, if a descriptor only defines a `__get__()` method, it only fires if the attribute being accessed is not in the underlying instance dictionary.
3. The `lazyproperty` class exploits this by having the `__get__()` method store the computed value on the instance dictionary.




{% highlight python %}
vars(c)  # # Get instance variables.
{% endhighlight %}




    {'area': 50.26548245743669, 'perimeter': 25.132741228718345, 'radius': 4.0}




{% highlight python %}
del c.area
vars(c)
{% endhighlight %}




    {'perimeter': 25.132741228718345, 'radius': 4.0}




{% highlight python %}
c.area  # Compute area again.
{% endhighlight %}

    Computing area





    50.26548245743669



4. One possible downside to this recipe is that the computed value becomes mutable after it’s created.


{% highlight python %}
c.area = 25
c.area  # Store the incorrect value.
{% endhighlight %}




    25



## Simplifying the Initialization of Data Structures


{% highlight python %}
class Structure:

    # Class variable that specifies expected fields.
    _fields= []
    def __init__(self, *args):
        if len(args) != len(self._fields):
            raise TypeError('Expected {} arguments'.format(len(self._fields)))

        # Set the arguments.
        for name, value in zip(self._fields, args):
            setattr(self, name, value)
{% endhighlight %}


{% highlight python %}
class Point(Structure):
    _fields = ['x', 'y']

class Circle(Structure):
    _fields = ['radius']

    def area(self):
        return math.pi * self.radius ** 2
{% endhighlight %}


{% highlight python %}
p = Point(1, 2)
c = Circle(3)
{% endhighlight %}


{% highlight python %}
p = Point(4)
{% endhighlight %}


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-28-215f3d655263> in <module>()
    ----> 1 p = Point(4)


    <ipython-input-25-64fd61f5d974> in __init__(self, *args)
          5     def __init__(self, *args):
          6         if len(args) != len(self._fields):
    ----> 7             raise TypeError('Expected {} arguments'.format(len(self._fields)))
          8
          9         # Set the arguments


    TypeError: Expected 2 arguments


### Support keyword arguments

+ Map the keyword arguments corresponding to the attribute names specified in `_fields`.


{% highlight python %}
class Structure:
    _fields= []

    def __init__(self, *args, **kwargs):
        if len(args) > len(self._fields):
            raise TypeError('Expected {} arguments'.format(len(self._fields)))

        # Set all of the positional arguments.
        for name, value in zip(self._fields, args):
            setattr(self, name, value)

        # Set the remaining keyword arguments.
        for name in self._fields[len(args):]:
            setattr(self, name, kwargs.pop(name))

        # Check for any remaining unknown arguments
        if kwargs:
            raise TypeError('Invalid argument(s): {}'.format(','.join(kwargs)))
{% endhighlight %}


{% highlight python %}
class ABC(Structure):
    _fields = ['A', 'B', 'C']
{% endhighlight %}


{% highlight python %}
s1 = ABC('a', 'b', 'c')
s2 = ABC('a', 'b', C='c')
s3 = ABC('a', B='b', C='c')
{% endhighlight %}

+ Use keyword arguments as additional attributes to the structure **not** specified in `_fields`.


{% highlight python %}
class Structure:
    _fields= []

    def __init__(self, *args, **kwargs):
        if len(args) != len(self._fields):
            raise TypeError('Expected {} arguments'.format(len(self._fields)))

        # Set the arguments.
        for name, value in zip(self._fields, args):
            setattr(self, name, value)

        # Set the additional arguments (if any).
        extra_args = kwargs.keys() - self._fields
        for name in extra_args:
            setattr(self, name, kwargs.pop(name))
        if kwargs:
            raise TypeError('Duplicate values for {}'.format(','.join(kwargs)))
{% endhighlight %}


{% highlight python %}
# Example use
class Stock(Structure):
    _fields = ['name', 'shares', 'price']

s1 = Stock('ACME', 50, 91.1)
s2 = Stock('ACME', 50, 91.1, date='8/2/2012')
{% endhighlight %}

<br><br><br>
> Take Reference from [Python Cookbook](http://shop.oreilly.com/product/0636920027072.do)<br><br>
> CHAPTER 8<br>
> Treat Functions as Objects
