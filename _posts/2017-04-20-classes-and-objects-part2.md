---
author: kimmy
layout: post
title:  "Treat Functions as Objects Part2"
date:   2017-04-20 14:51:37 +08:00
categories: Python
tags:
- Python
- Notes
- collections
- getattr()
---

* content
{:toc}



## Implementing Custom Containers

### `collections` library


{% highlight python %}
# Want a class to support iteration
import collections

class A(collections.Iterable):
    pass
{% endhighlight %}


{% highlight python %}
a = A()
{% endhighlight %}


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-2-144b248f218a> in <module>()
    ----> 1 a = A()


    TypeError: Can't instantiate abstract class A with abstract methods __iter__


Thus the special feature about inheriting from `collections.Iterable` is that it ensures you implement all of the required special methods.


{% highlight python %}
import collections
import bisect

collections.Sequence()
{% endhighlight %}


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-4-ce071b06e2e3> in <module>()
          2 import bisect
          3
    ----> 4 collections.Sequence()


    TypeError: Can't instantiate abstract class Sequence with abstract methods __getitem__, __len__



{% highlight python %}
# Implement with __getitem__ and __len__

class SortedItems(collections.Sequence):

    def __init__(self, initial=None):
        self._items = sorted(initial) if initial is not None else []

    # Require sequence methods
    def __getitem__(self, index):
        return self._items[index]

    def __len__(self):
        return len(self._items)

    # Method for adding an item in the right location
    def add(self, item):
        bisect.insort(self._items, item)
{% endhighlight %}


{% highlight python %}
items = SortedItems([7, 1, 4])

list(items)
{% endhighlight %}




    [1, 4, 7]




{% highlight python %}
items[0]
items[-1]
{% endhighlight %}




    7




{% highlight python %}
items.add(1996)
list(items)
{% endhighlight %}




    [1, 4, 7, 1996]




{% highlight python %}
items[1:3]
{% endhighlight %}




    [4, 7]



1. `SortedItems` support all of the usual sequence operations, including indexing, iteration, `len()`, `in` and even slicing.
2. `bisect.insort()` inserts an item into a list so that the list **remains in order**.

### Discussion

1. Custom container will satisfy various type checks like `isinstance(items, collections.Iterable/Sequence/Sized)`.
2. The `numbers` module provides a similar collection of abstract classes related to numeric data types.

## Delegating Attribute Access

### `__getattr__()`


{% highlight python %}
class A:
    def spam(self, x):
        pass
    def foo(self):
        pass

{% endhighlight %}

Solution 1


{% highlight python %}
class B:
    def __init__(self):
        self._a = A()
    def spam(self, x):
        # Delegate to the internal self._a instance
        return self._a.spam(x)
    def foo(self):
        # Delegate to the internal self._a instance
        return self._a.foo()
    def bar(self):
        pass
{% endhighlight %}

Solution 2


{% highlight python %}
class B:
    def __init__(self):
        self._a = A()  # private member
    def bar(self):
        pass

    # Expose all of the methods defined on class A
    def __getattr__(self, name):
        return getattr(self._a, name)
{% endhighlight %}

`__getattr__()` gets called if code tries to access an attribute that doesn’t exist.


{% highlight python %}
b = B()
b.bar()      # Calls B.bar() (exists on B)
b.spam(42)   # Calls B.__getattr__('spam') and delegates to A.spam
{% endhighlight %}

## Defining More Than One Constructor in a Class

### `@classmethod`


{% highlight python %}
import time

class Date:

    # Primary constructor
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    # Alternate constructor
    @classmethod
    def today(cls):
        t = time.localtime()
        return cls(t.tm_year, t.tm_mon, t.tm_mday)
{% endhighlight %}


{% highlight python %}
a = Date(2017, 4, 20)  # Primary
b = Date.today()       # Alternate
{% endhighlight %}

A class method receives **this** class as the first argument (`cls`).

## Implementing Stateful Objects or State Machines

### Ugly implement
common operations (e.g., `read()` and `write()`) always check the state before proceeding.


{% highlight python %}
class Connection:

    def __init__(self):
        self.state = 'CLOSED'

    def read(self):
        if self.state != 'OPEN':
            raise RuntimeError('Not open')
            print('reading')

    def write(self, data):
        if self.state != 'OPEN':
            raise RuntimeError('Not open')
            print('writing')

    def open(self):
        if self.state == 'OPEN':
            raise RuntimeError('Already open')
            self.state = 'OPEN'

    def close(self):
        if self.state == 'CLOSED':
            raise RuntimeError('Already closed')
            self.state = 'CLOSED'
{% endhighlight %}

### Elegant approach
encode each operational state as a separate **class** and arrange for the `Connection` class to delegate to the state class.


{% highlight python %}
class Connection:

    def __init__(self):
        self.new_state(ClosedConnectionState)

    def new_state(self, newstate):
        self._state = newstate

    # Delegate to the state class
    def read(self):
        return self._state.read(self)

    def write(self, data):
        return self._state.write(self, data)

    def open(self):
        return self._state.open(self)

    def close(self):
        return self._state.close(self)
{% endhighlight %}


{% highlight python %}
# Connection state base class
class ConnectionState:

    @staticmethod
    def read(conn):
        raise NotImplementedError()

    @staticmethod
    def write(conn, data):
        raise NotImplementedError()

    @staticmethod
    def open(conn):
        raise NotImplementedError()

    @staticmethod
    def close(conn):
        raise NotImplementedError()
{% endhighlight %}


{% highlight python %}
# Implementation of different states
class ClosedConnectionState(ConnectionState):

    @staticmethod
    def read(conn):
        raise RuntimeError('Not open')

    @staticmethod
    def write(conn, data):
        raise RuntimeError('Not open')

    @staticmethod
    def open(conn):
        conn.new_state(OpenConnectionState)

    @staticmethod
    def close(conn):
        raise RuntimeError('Already closed')
{% endhighlight %}


{% highlight python %}
class OpenConnectionState(ConnectionState):

    @staticmethod
    def read(conn):
        print('reading')

    @staticmethod
    def write(conn, data):
        print('writing')

    @staticmethod
    def open(conn):
        raise RuntimeError('Already open')

    @staticmethod
    def close(conn):
        conn.new_state(ClosedConnectionState)
{% endhighlight %}


{% highlight python %}
c = Connection()
c._state
{% endhighlight %}




    __main__.ClosedConnectionState




{% highlight python %}
c.read()
{% endhighlight %}


    ---------------------------------------------------------------------------

    RuntimeError                              Traceback (most recent call last)

    <ipython-input-59-993af947a018> in <module>()
    ----> 1 c.read()


    <ipython-input-49-9af8f90a4e50> in read(self)
          9     # Delegate to the state class
         10     def read(self):
    ---> 11         return self._state.read(self)
         12
         13     def write(self, data):


    <ipython-input-51-f9172a827b97> in read(conn)
          4     @staticmethod
          5     def read(conn):
    ----> 6         raise RuntimeError('Not open')
          7
          8     @staticmethod


    RuntimeError: Not open



{% highlight python %}
c.open()
c._state
{% endhighlight %}




    __main__.OpenConnectionState




{% highlight python %}
c.read()
{% endhighlight %}

    reading



{% highlight python %}
c.write('hello')
{% endhighlight %}

    writing



{% highlight python %}
c.close()
c._state
{% endhighlight %}




    __main__.ClosedConnectionState



Each state takes an instance of `Connection` as the first argument.

## Calling a Method on an Object Given the Name As a String

### `getattr()`


{% highlight python %}
import math
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return 'Point({!r:},{!r:})'.format(self.x, self.y)

    def distance(self, x, y):
        return math.hypot(self.x - x, self.y - y)
{% endhighlight %}


{% highlight python %}
p = Point(1, 2)

getattr(p, 'distance')(0, 0)  # Calls p.distance(0, 0)
{% endhighlight %}




    2.23606797749979



### `operator.methodcaller()`


{% highlight python %}
import operator
operator.methodcaller('distance', 0, 0)(p)
{% endhighlight %}




    2.23606797749979



especially useful in looking up a method by name and supply the same arguments many times


{% highlight python %}
points = [
    Point(1, 2),
    Point(3, 0),
    Point(10, -3),
    Point(-5, -7),
    Point(-1, 8),
    Point(3, 2)
]
{% endhighlight %}

want sort these points by distance from origin (0, 0)


{% highlight python %}
points.sort(key=operator.methodcaller('distance', 0, 0))
{% endhighlight %}

### Discussion

Calling a method is actually two separate steps involving
1. an attribute lookup
2. a function call

## Making Classes Support Comparison Operations

### `functools.total_ordering`


{% highlight python %}
from functools import total_ordering
class Room:
    def __init__(self, name, length, width):
        self.name = name
        self.length = length
        self.width = width
        self.square_feet = self.length * self.width
{% endhighlight %}


{% highlight python %}
@total_ordering
class House:

    def __init__(self, name, style):
        self.name = name
        self.style = style
        self.rooms = list()

    @property
    def living_space_footage(self):
        return sum(r.square_feet for r in self.rooms)

    def add_room(self, room):
        self.rooms.append(room)

    def __str__(self):
        return '{}: {} square foot {}'.format(self.name,
                                              self.living_space_footage,
                                              self.style)
    def __eq__(self, other):
        return self.living_space_footage == other.living_space_footage

    def __lt__(self, other):
        return self.living_space_footage < other.living_space_footage
{% endhighlight %}

<br><br><br>
> Take Reference from [Python Cookbook](http://shop.oreilly.com/product/0636920027072.do)<br><br>
> CHAPTER 8<br>
> Treat Functions as Objects
