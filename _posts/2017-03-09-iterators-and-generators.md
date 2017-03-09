---
layout: post
title:  Iterators and Generators
date:   2017-03-09 15:06:30 +08:00
categories: Python
tags:
- heapq.merge
- chain
- format
- random
- datetime
---

* content
{:toc}




## Implementing the Iterator Protocol

Python’s iterator protocol requires `__iter__()` to return a special iterator object that<br>
implements a `__next__()` operation and uses a `StopIteration` exception to signal completion.


{% highlight python %}
class Node:
    def __init__(self, value):
        self._value = value
        self._children = []

    def __repr__(self):
        return 'Node({!r})'.format(self._value)

    def add_child(self, node):
        self._children.append(node)

    def __iter__(self):
        return iter(self._children)

    def depth_first(self):
        yield self
        for c in self:
            yield from c.depth_first()
{% endhighlight %}


{% highlight python %}
root = Node(0)
child1 = Node(1)
child2 = Node(2)
root.add_child(child1)
root.add_child(child2)
child1.add_child(Node(3))
child1.add_child(Node(4))
child2.add_child(Node(5))
{% endhighlight %}


{% highlight python %}
for ch in root.depth_first():
    print(ch)
{% endhighlight %}

    Node(0)
    Node(1)
    Node(3)
    Node(4)
    Node(2)
    Node(5)


+ `depth_first()`<br>
    first yields itself and then iterates over each child yielding the items produced by the child's `depth_first()` method(using `yield from`).

## Taking a Slice of an Iterator

### `itertools.islice()`
+ is perfectly suited for taking slices of iterators and generators.
+ works by consuming and discarding all of the items up to the starting slice index.


{% highlight python %}
# Normal slicing operator doesn’t work.
def count(n):
    while True:
        yield n
        n += 1

c = count(0)
c[10:20]
{% endhighlight %}


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-13-d2d4a627221c> in <module>()
          6
          7 c = count(0)
    ----> 8 c[10:20]


    TypeError: 'generator' object is not subscriptable



{% highlight python %}
# Now using islice()
import itertools
for x in itertools.islice(c, 10, 15):
    print(x)
{% endhighlight %}

    10
    11
    12
    13
    14


## Skipping the First Part of an Iterable

Discarding the first part of an iterable is also slightly different than simply filtering all of it.

### `itertools.dropwhile()`
+ needs a function and an iterable.
+ The returned iterator discards the first items in the sequence as long as the supplied function returns `True`.

### `itertools.islice()`
+ requires for the exact **index** of items that you want to skip.


{% highlight python %}
from itertools import islice
items = ['a', 'b', 'c', 1, 2, 3, 4]
for x in islice(items, 3, None):
    print(x)
{% endhighlight %}

    1
    2
    3
    4


`None` indicates everything **beyond** the first three items as opposed to only the first three items<br>
(e.g. a slice of `[3:]` as opposed to a slice of `[:3]`).

## Iterating Over the Index-Value Pairs of a Sequence

### `enumerate()`


{% highlight python %}
my_list = ['a', 'b', 'c']
for i, v in enumerate(my_list):
    print(i, v)
{% endhighlight %}

    0 a
    1 b
    2 c


+ Pass in a `start` argument to count from 1 instead of 0.


{% highlight python %}
for i, v in enumerate(my_list, 1):
    print(i, v)
{% endhighlight %}

    1 a
    2 b
    3 c


+ Especially useful for tracking line numbers in files to use a line number in an error message.


{% highlight python %}
def parse_data(file_name):
    with open(filename, 'rt') as f:
        for lineno, line in enumerate(f, 1):
            fields = line.split()
            try:
                count = int(fields[1])
                # ...
            except Value as e:
                print('Line {}: Parse error: {}'.format(lineno, e))
{% endhighlight %}

## Iterating Over Multiple Sequences Simultaneously

### `zip`


{% highlight python %}
xs = list('abcdefg')
ys = list(range(7))
for x, y in zip(xs, ys):
    print(x, y)
{% endhighlight %}

    a 0
    b 1
    c 2
    d 3
    e 4
    f 5
    g 6


+ Iteration stops whenever one of the input sequences is exhausted.


{% highlight python %}
xs = list('abc')
ys = list(range(7))
for x, y in zip(xs, ys):
    print(x, y)
{% endhighlight %}

    a 0
    b 1
    c 2


+ Produce a dictionary.


{% highlight python %}
headers = ['name', 'age', 'height']
values = ['kimmy', '20', '230']
# pair the values together to make a dictionary
my_dict = dict(zip(headers, values))
{% endhighlight %}

+ `zip` creates an iterator as a result. Use `list()` function if necessary.


{% highlight python %}
zip(xs, ys)
{% endhighlight %}




    <zip at 0x1d92c992908>




{% highlight python %}
list(zip(xs, ys))
{% endhighlight %}




    [('a', 0), ('b', 1), ('c', 2)]



### `itertools.zip_longest()`


{% highlight python %}
from itertools import zip_longest
for i in zip_longest(xs, ys):
    print(i)
{% endhighlight %}

    ('a', 0)
    ('b', 1)
    ('c', 2)
    (None, 3)
    (None, 4)
    (None, 5)
    (None, 6)



{% highlight python %}
for i in zip_longest(xs, ys, fillvalue='*'):
    print(i)
{% endhighlight %}

    ('a', 0)
    ('b', 1)
    ('c', 2)
    ('*', 3)
    ('*', 4)
    ('*', 5)
    ('*', 6)


## Iterating on Items in Separate Containers

### `itertools.chain()`


{% highlight python %}
from itertools import chain
a = ['1', '2', '3', '4']
b = ['a', 'b', 'c']
for x in chain(a, b):
    print(x)
{% endhighlight %}

    1
    2
    3
    4
    a
    b
    c


+ In most case, `chain()` is to perform certain operations on all of the items as once but the items are pooled into different working set.


{% highlight python %}
# Various working sets of items
activate_items = set()
inactivate_items = set()

# iterate over all items
for item in chain(activate_items, inactivate_items):
    # Process item
    # ...
    pass
{% endhighlight %}

+ `chain` is more efficient than simple `+` operation, which creates an entirely new sequence and additionally requires `a` and `b` to be the same type.


{% highlight python %}
# Inefficient
for x in a + b:
    # ...

# Better
for x in chain(a, b):
    # ...
{% endhighlight %}

## Flattening a Nested Sequence

### `yield from`


{% highlight python %}
from collections import Iterable

# a recursive generator function
def flatten(items, ignore_types=(str, bytes)):
    for x in items:
        if isinstance(x, Iterable) and not isinstance(x, ignore_types):
            # emit all of its values as a kind of subroutine
            yield from (flatten(x))
        else:
            # prevent strings and bytes from being interpreted as iterables
            # and expanded as individual characters.
            yield x
{% endhighlight %}


{% highlight python %}
items = [1, 2, [3, 4, [5, 6], 7], 8]
for x in flatten(items):
    print(x)
{% endhighlight %}

    1
    2
    3
    4
    5
    6
    7
    8



{% highlight python %}
items = ['hello,',['the', ['new'], 'world']]
for x in flatten(items):
    print(x)
{% endhighlight %}

    hello,
    the
    new
    world


+ `yield from` statement is useful if you ever want to write generators that call other generators as `subroutines`.


{% highlight python %}
# use extra `for` loop
def flatten_loop(items, ignore_types=(str, bytes)):
    for x in items:
        if isinstance(x, Iterable) and not isinstance(x, ignore_types):
            for i in flatten(x):
                yield i
        else:
            yield x
{% endhighlight %}

+ `yield from` has a more important role in advanced programs involving coroutines and generator-based concurrency.

## Iterating in Sorted Order Over Merged Sorted Iterables

### `heapq.merge()`


{% highlight python %}
import heapq
a = [1, 3, 5, 7]
b = [2, 4, 6, 8]
for c in heapq.merge(a, b):
    print(c)
{% endhighlight %}

    1
    2
    3
    4
    5
    6
    7
    8


+ Merge two **sorted** files.


{% highlight python %}
import heapq

with open('sorted_file_1', 'rt') as f1, \
     open('sorted_file_2', 'rt') as f2, \
     open('merged_file', 'wt') as outf:
    for line in heapq.merge(f1, f2):
        outf.write(line)
{% endhighlight %}



<br><br><br>
> Take Reference from [Python Cookbook](http://shop.oreilly.com/product/0636920027072.do)<br><br>
> CHAPTER 4<br>
> Iterators and Generators
