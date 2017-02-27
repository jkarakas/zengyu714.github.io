---
author: kimmy
layout: post
title:  "Itertools and Arithmetic Progression Generator"
date:   2017-02-24 21:22:31 +08:00
categories: Python
tags:
- Python
- Notes
- iterator
- itertools
---


* content
{:toc}



## `ArithmeticProgression` Class


{% highlight python %}
class ArithmeticProgression:

    def __init__(self, begin, step, end=None):
        self.begin = begin
        self.step = step
        self.end = end                                  # None: "infinite" series

    def __iter__(self):
        result=type(self.begin + self.step)(self.begin) # cast type
        forever = self.end is None
        index = 0

        while forever or result < self.end:
            yield result
            index += 1                                   # use index to reduce the cumulative
            result = self.begin + self.step * index      # effect of errors when working with floats
{% endhighlight %}

## `arithmetic_progression_generator` Function


{% highlight python %}
def arithmetic_progression_generator(begin, step, end=None):
    result = type(begin + step)(begin)
    forever = end is None
    index = 0
    while forever or result < end:
        yield result
        index += 1
        result = bagin + index * step
{% endhighlight %}

## Arithmetic Progression with `itertools`


{% highlight python %}
import itertools
{% endhighlight %}

### `itertools.count` never stops


{% highlight python %}
gen = itertools.count(1, 0.5)
{% endhighlight %}


{% highlight python %}
next(gen)
{% endhighlight %}




    1




{% highlight python %}
next(gen)
{% endhighlight %}




    1.5




{% highlight python %}
next(gen)
{% endhighlight %}




    2.0



### `itertools.takewhile` stops when a given predicate evaluates to `False`

and produces a generator that consumes another generator.


{% highlight python %}
gen = itertools.takewhile(lambda n: n < 3, itertools.count(1, .5))
list(gen)
{% endhighlight %}




    [1, 1.5, 2.0, 2.5]



### `short_arithmetic_progression_generator` Function


{% highlight python %}
import itertools
def short_arithmetic_progression_generator(begin, step, end=None):
    first = type(begin + step)(begin)
    ap_gen = itertools.count(first, step)
    if end is not None:
        ap_gen = itertools.takewhile(lambda n: n < end, ap_gen)
    return ap_gen
{% endhighlight %}

NOTE:<br>
`short_arithmetic_progression_generator` is not a generator function because it has no `yield` keyword.<br>
But it returns a generator, so it operates as a generator factory, just as a generator function does.

## More about `itertools`

### `itertools.cycle`


{% highlight python %}
from itertools import cycle
infinite_letter = cycle(list('abc'))
{% endhighlight %}


{% highlight python %}
next(infinite_letter)
{% endhighlight %}




    'a'




{% highlight python %}
next(infinite_letter)
{% endhighlight %}




    'b'




{% highlight python %}
next(infinite_letter)
{% endhighlight %}




    'c'




{% highlight python %}
next(infinite_letter)
{% endhighlight %}




    'a'



### `itertools.islice`


{% highlight python %}
from itertools import islice
infinite_letter = cycle(list('abc'))
finite_letter = islice(infinite_letter, 1, 7)
list(finite_letter)
{% endhighlight %}




    ['b', 'c', 'a', 'b', 'c', 'a']



---


{% highlight python %}
import itertools
{% endhighlight %}

### `itertools.filterfalse`


{% highlight python %}
list(itertools.filterfalse(lambda c: c.lower() in 'aeiou', 'generator'))
{% endhighlight %}




    ['g', 'n', 'r', 't', 'r']



### `itertools.dropwhile`


{% highlight python %}
list(itertools.dropwhile(lambda c: c.lower() in 'aeiou', 'AaBbCc'))
{% endhighlight %}




    ['B', 'b', 'C', 'c']



### `itertools.takewhile`


{% highlight python %}
list(itertools.takewhile(lambda c: c.lower() in 'aeiou', 'AaBbCc'))
{% endhighlight %}




    ['A', 'a']



### `itertools.accumulate`
yields accumulated sums;<br>
if `func` is provided, yields the result of applying it to the first pair of items, then to the first result and next item, etc.


{% highlight python %}
sample = [5, 4, 2, 8, 7, 6, 3, 0, 9, 1]
{% endhighlight %}


{% highlight python %}
list(itertools.accumulate(sample))
{% endhighlight %}




    [5, 9, 11, 19, 26, 32, 35, 35, 44, 45]




{% highlight python %}
list(itertools.accumulate(sample, min))
{% endhighlight %}




    [5, 4, 2, 2, 2, 2, 2, 0, 0, 0]




{% highlight python %}
list(itertools.accumulate(sample, max))
{% endhighlight %}




    [5, 5, 5, 8, 8, 8, 8, 8, 9, 9]




{% highlight python %}
import operator
list(itertools.accumulate(sample, operator.mul))
{% endhighlight %}




    [5, 20, 40, 320, 2240, 13440, 40320, 0, 0, 0]




{% highlight python %}
# factorials from 1! to 10!
list(itertools.accumulate(range(1, 11), operator.mul))
{% endhighlight %}




    [1, 2, 6, 24, 120, 720, 5040, 40320, 362880, 3628800]



### `itertools.product`
is a lazy way of computing Cartesian products.


{% highlight python %}
list(itertools.product('ABC', range(2)))
{% endhighlight %}




    [('A', 0), ('A', 1), ('B', 0), ('B', 1), ('C', 0), ('C', 1)]




{% highlight python %}
list(itertools.product('ABC'))
{% endhighlight %}




    [('A',), ('B',), ('C',)]



The `repeat=N` keyword argument tells product to consume each input iterable `N` times.


{% highlight python %}
list(itertools.product('ABC', repeat=2))
{% endhighlight %}




    [('A', 'A'),
     ('A', 'B'),
     ('A', 'C'),
     ('B', 'A'),
     ('B', 'B'),
     ('B', 'C'),
     ('C', 'A'),
     ('C', 'B'),
     ('C', 'C')]




{% highlight python %}
list(itertools.product(range(2), repeat=3))
{% endhighlight %}




    [(0, 0, 0),
     (0, 0, 1),
     (0, 1, 0),
     (0, 1, 1),
     (1, 0, 0),
     (1, 0, 1),
     (1, 1, 0),
     (1, 1, 1)]



### combinatoric generators


{% highlight python %}
list(itertools.combinations('ABC', 2))
{% endhighlight %}




    [('A', 'B'), ('A', 'C'), ('B', 'C')]




{% highlight python %}
list(itertools.combinations_with_replacement('ABC', 2))
{% endhighlight %}




    [('A', 'A'), ('A', 'B'), ('A', 'C'), ('B', 'B'), ('B', 'C'), ('C', 'C')]




{% highlight python %}
list(itertools.permutations('ABC', 2))
{% endhighlight %}




    [('A', 'B'), ('A', 'C'), ('B', 'A'), ('B', 'C'), ('C', 'A'), ('C', 'B')]




{% highlight python %}
list(itertools.product('ABC', repeat=2))
{% endhighlight %}




    [('A', 'A'),
     ('A', 'B'),
     ('A', 'C'),
     ('B', 'A'),
     ('B', 'B'),
     ('B', 'C'),
     ('C', 'A'),
     ('C', 'B'),
     ('C', 'C')]



### `itertools.groupby`


{% highlight python %}
animals = ['duck', 'eagle', 'rat', 'giraffe', 'bear',
            'bat', 'dolphin', 'shark', 'lion']
{% endhighlight %}


{% highlight python %}
# To use `groupby`, the input should be sorted; here the words are sorted by length.
animals.sort(key=len)
animals
{% endhighlight %}




    ['rat', 'bat', 'duck', 'bear', 'lion', 'eagle', 'shark', 'giraffe', 'dolphin']




{% highlight python %}
for length, group in itertools.groupby(animals, len):
    print(length, '--> ', list(group))
{% endhighlight %}

    3 -->  ['rat', 'bat']
    4 -->  ['duck', 'bear', 'lion']
    5 -->  ['eagle', 'shark']
    7 -->  ['giraffe', 'dolphin']



{% highlight python %}
for length, group in itertools.groupby(reversed(animals), len):
    print(length, '--> ', list(group))
{% endhighlight %}

    7 -->  ['dolphin', 'giraffe']
    5 -->  ['shark', 'eagle']
    4 -->  ['lion', 'bear', 'duck']
    3 -->  ['bat', 'rat']


## Interesting

{% highlight python %}
all([])
{% endhighlight %}




    True


{% highlight python %}
any([])
{% endhighlight %}




    False


> Take Reference from Fluent Python Chapter 14
