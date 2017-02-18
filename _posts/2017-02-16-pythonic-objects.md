---
author: kimmy
layout: post
title:  "Pythonic Object Attributes Part1"
date:   2017-02-16 19:20:33 +08:00
categories: Python
tags:
- Python
- Notes
- Object
- Doctest
---


* content
{:toc}



## `Vector2d` Class

### Didactic Example


{% highlight python %}
"""
A two-dimensional vector class
    >>> v1 = Vector2d(3, 4)
    >>> print(v1.x, v1.y)
    3.0 4.0
    >>> x, y = v1
    >>> x, y
    (3.0, 4.0)
    >>> v1
    Vector2d(3.0, 4.0)
    >>> v1_clone = eval(repr(v1))
    >>> v1 == v1_clone
    True
    >>> print(v1)
    (3.0, 4.0)
    >>> octets = bytes(v1)
    >>> octets
    b'd\\x00\\x00\\x00\\x00\\x00\\x00\\x08@\\x00\\x00\\x00\\x00\\x00\\x00\\x10@'
    >>> abs(v1)
    5.0
    >>> bool(v1), bool(Vector2d(0, 0))
    (True, False)


Test of ``.frombytes()`` class method:
    >>> v1_clone = Vector2d.frombytes(bytes(v1))
    >>> v1_clone
    Vector2d(3.0, 4.0)
    >>> v1 == v1_clone
    True


Tests of ``format()`` with Cartesian coordinates:
    >>> format(v1)
    '(3.0, 4.0)'
    >>> format(v1, '.2f')
    '(3.00, 4.00)'
    >>> format(v1, '.3e')
    '(3.000e+00, 4.000e+00)'


Tests of the ``angle`` method::
    >>> Vector2d(0, 0).angle()
    0.0
    >>> Vector2d(1, 0).angle()
    0.0
    >>> epsilon = 10**-8
    >>> abs(Vector2d(0, 1).angle() - math.pi/2) < epsilon
    True
    >>> abs(Vector2d(1, 1).angle() - math.pi/4) < epsilon
    True


Tests of ``format()`` with polar coordinates:
    >>> format(Vector2d(1, 1), 'p') # doctest:+ELLIPSIS
    '<1.414213..., 0.785398...>'
    >>> format(Vector2d(1, 1), '.3ep')
    '<1.414e+00, 7.854e-01>'
    >>> format(Vector2d(1, 1), '0.5fp')
    '<1.41421, 0.78540>'


Tests of `x` and `y` read-only properties:
    >>> v1.x, v1.y
    (3.0, 4.0)
    >>> v1.x = 123
    Traceback (most recent call last):
    ...
    AttributeError: can't set attribute


Tests of hashing:
    >>> v1 = Vector2d(3, 4)
    >>> v2 = Vector2d(3.1, 4.2)
    >>> hash(v1), hash(v2)
    (7, 384307168202284039)
    >>> len(set([v1, v2]))
    2


"""

from array import array
import math

class Vector2d:
    typecode = 'd'

    def __init__(self, x, y):
        self.__x = float(x)
        self.__y = float(y)

    @property
    def x(self):
        return self.__x

    @property
    def y(self):
        return self.__y

    def __iter__(self):
        # iterable makes unpacking work
        return (i for i in (self.x, self.y))

    def __repr__(self):
        # *self feeds the x and y components to format, kind of unpacking
        class_name = type(self).__name__
        return "{}({!r}, {!r})".format(class_name, *self)

    def __str__(self):
        return str(tuple(self))
        # return "({0[0]},{0[1]})".format(self)

    def __bytes__(self):
        return (bytes([ord(self.typecode)])+
               bytes(array(self.typecode,self)))

    def __eq__(self, other):
        return tuple(self) == tuple(other)

    def __hash__(self):
        return hash(self.x) ^ hash(self.y)

    def __abs__(self):
        return math.hypot(self.x, self.y)

    def __bool__(self):
        return bool(self.x or self.y)

    def __add__(self, other):
        self.x += other.x
        self.y += other.y
        return Vector2d(self.x, self.y)

    def angle(self):
        return math.atan2(self.y, self.x)

    def __format__(self, fmt_spec=''):
        # polar
        if fmt_spec.endswith('p'):
            fmt_spec = fmt_spec[:-1]
            coords = (abs(self),self.angle())
            outer_fmt="<{}, {}>"
        else:
            coords = self
            outer_fmt = "({}, {})"
        componets = (format(c, fmt_spec) for c in coords)
        return outer_fmt.format(*componets)

    @classmethod
    def frombytes(cls, octets):
        # No self argument, instead is the class itself
        typecode = chr(octets[0])
        memv = memoryview(octets[1:]).cast(typecode)
        return cls(*memv)
{% endhighlight %}

### `__bytes__`

+ convert the typecode to `bytes`
+ concatenate `bytes` converted from an `array` built by iterating over the instance


{% highlight python %}
def __bytes__(self):
    return (bytes([ord(self.typecode)]) +
    bytes(array(self.typecode, self)))
{% endhighlight %}

### `@classmethod`
+ defines a method that operates on the **class** and not on instances.
+ is used mostly for alternative constructors.
+ `cls(*memv)` actually builds a new instance.


{% highlight python %}
@classmethod
def frombytes(cls, octets):
    # No self argument, instead is the class itself
    typecode = chr(octets[0])
    memv = memoryview(octets[1:]).cast(typecode)
    return cls(*memv)
{% endhighlight %}

### `__format__`


{% highlight python %}
def __format__(self, fmt_spec=''):
    components = (format(c, fmt_spec) for c in self)
    return "({},{})".format(*components)
{% endhighlight %}

### `__hash__`

#### 1. immutable


{% highlight python %}
class Vector2d:
    typecode = 'd'

    def __init__(self, x, y):
        self.__x = float(x)
        self.__y = float(y)

    @property
    def x(self):
        return self.__x

    @property
    def y(self):
        return self.__y

    def __iter(self):
        return (i for i in (self.x, self.y))
{% endhighlight %}

#### 2. `__eq__`

#### Implement

suggests using the bitwise XOR operator `^` to mix the hashes of the components.


{% highlight python %}
def __hash__(self):
    return hash(self.x)^hash(self.y)
{% endhighlight %}

### KEYNOTE:
---
1. Make sure to implement methods that have real use.
2. `Vector2d` is mainly used to show special methods, not a template for every user-defined class.
3. `doctest` is sensitive of the whitespace in the formatted string.

## Private and Protected Attributes

Python stores the name in the instance `__dict__` prefixed with a leading underscore and the class name.
`__x` becomes `_Vector2d__x`


{% highlight python %}
v1 = Vector2d(3, 4)
v1.__dict__
{% endhighlight %}




    {'_Vector2d__x': 3.0, '_Vector2d__y': 4.0}




{% highlight python %}
v1._Vector2d__x
{% endhighlight %}




    3.0



WARNING:
Don't assign a value to a private component.


{% highlight python %}
v1._Vector2d__x = 9
v1
{% endhighlight %}




    Vector2d(9, 4.0)



## `__slots__` Class Attribute

By default, Python stores instance attributes in a per-instance dict named `__dict__`.
Using a `tuple` instead of a `dict` could save memory if don't keep hashable,
in the condition of dealing with millions of instances with few attributes.


{% highlight python %}
class vector2d:
    __slots__ = ('__x', '__y')
    # write down all the instance attributes in the class

    typecode = 'd'

    # ...
{% endhighlight %}

NOTE:
1. `Numpy` is suitable to process numeric data.
2. Module `resource` is only available in Unix system :)

## Overriding Class Attributes

### Solution 1: Customizing an instance


{% highlight python %}
v1 = Vector2d(3.1,4.15)
v1_double_byte = bytes(v1)
print("v1 :%s\t len: %s" % (v1_double_byte, len(v1_double_byte)))
{% endhighlight %}

    v1 :b'd\xcd\xcc\xcc\xcc\xcc\xcc\x08@\x9a\x99\x99\x99\x99\x99\x10@'	 len: 17



{% highlight python %}
v1.typecode = 'f'
v1_float_byte = bytes(v1)
print("v1 :%s\t len: %s" % (v1_float_byte, len(v1_float_byte)))
{% endhighlight %}

    v1 :b'fffF@\xcd\xcc\x84@'	 len: 9


NOTE: the default typecode is `d`, unchanged


{% highlight python %}
Vector2d.typecode
{% endhighlight %}




    'd'



### Solution 2: Set on the subclass directly


{% highlight python %}
class floatVector2d(Vector2d):
    typecode = 'f'
{% endhighlight %}


{% highlight python %}
fv = floatVector2d(3.14,1.5)
print("%r\tlen: %s" % (fv, len(bytes(fv))))
{% endhighlight %}

    floatVector2d(3.14, 1.5)	len: 9
