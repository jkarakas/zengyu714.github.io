---
author: kimmy
layout: post
title:  "Treat Functions as Objects"
date:   2017-02-12 16:22:42 +08:00
categories: Python
tags:
- Python
- Notes
---

* content
{:toc}



## Consider a Function as An **object**

### Example


{% highlight python %}
def factorial(n):
    '''return n!'''
    return 1 if n < 2 else n*factorial(n-1)
{% endhighlight %}


{% highlight python %}
factorial(5)
{% endhighlight %}




    120




{% highlight python %}
factorial.__class__
{% endhighlight %}




    function




{% highlight python %}
print(type(factorial))
{% endhighlight %}

    <class 'function'>



{% highlight python %}
factorial.__doc__
{% endhighlight %}




    'return n!'




{% highlight python %}
help(factorial)
{% endhighlight %}

    Help on function factorial in module __main__:

    factorial(n)
        return n!




{% highlight python %}
map(factorial,range(11))
{% endhighlight %}




    <map at 0x2515a22a358>




{% highlight python %}
list(map(factorial,range(11)))
{% endhighlight %}




    [1, 1, 2, 6, 24, 120, 720, 5040, 40320, 362880, 3628800]



### KEYNOTE
---
1. `factorial.__class__` , `factorial.__doc__` are attributes of function objects.
2. `factorial` is an instance of the `function` class

## Higher-Order Functions: nested functions

### `sorted` and its optional `key` —— *one-argument* function


{% highlight python %}
mess = list('Higher-Order Functions: nested functions'.split())
print(sorted(mess, key = len))
{% endhighlight %}

    ['nested', 'functions', 'Functions:', 'Higher-Order']



{% highlight python %}
def reverse(word):    # one argument
    return word[::-1]
{% endhighlight %}


{% highlight python %}
sorted(mess)
{% endhighlight %}




    ['Functions:', 'Higher-Order', 'functions', 'nested']




{% highlight python %}
sorted(mess, key = reverse)
{% endhighlight %}




    ['Functions:', 'nested', 'Higher-Order', 'functions']



### Replace `map` and `filter`

with generator expressions and list comprehensions


{% highlight python %}
list(map(factorial,range(7)))
[factorial(x) for x in range(7)]
{% endhighlight %}




    [1, 1, 2, 6, 24, 120, 720]




{% highlight python %}
list(map(factorial,filter(lambda x: x & 0x1, range(7))))
[factorial(x) for x in range(7) if x % 2]
{% endhighlight %}




    [1, 6, 120]



### Use `sum` rather than `reduce`

Apply some operation with previous results in a sequence, until reducing a sequence of values to a single value.<br>
Others reducing functions like `all(iterabe)` , `any(iterable)`

## Anonymous Functions

The body part of lambda functions should be pure expressions.<br>
Most used in the argument of higher-order functions


{% highlight python %}
sorted(mess, key = lambda x : x[::-1])
{% endhighlight %}




    ['Functions:', 'nested', 'Higher-Order', 'functions']



## User-Defined Callable Types
> Following example from Fluent Python chapter 5<br>
A randomPick class does one thing: picks items from a shuffled list


{% highlight python %}
import random

class randomPick:

    def __init__(self,item):
        self._item = list(item)    # 1
        random.shuffle(self._item)

    def pick(self):
        try:
            return self._item.pop()
        except IndexError:
            raise LookupError('pick from empty list')

    def __call__(self):            # 2
        return self.pick()
{% endhighlight %}


{% highlight python %}
test = randomPick(range(7))
test.pick()
{% endhighlight %}




    2




{% highlight python %}
test()         # same as test.pick()
{% endhighlight %}




    6




{% highlight python %}
callable(test) # 3
{% endhighlight %}




    True



### KEYNOTE
---
1. `__init__` accepts any iterable; building a local copy prevents unexpected side
    effects on any list passed as an argument
2. — `randomPick()` is an alias of `randomPick.pick()`.<br>
  — Implementing `__call__` is an easy way to create function-like objects that have
    some **internal state** that must be kept across invocations.

3. `callable()` built-in to identify whether a callable object.

## Function-specific Attributes
`set(function) - set(user-defined object)`


{% highlight python %}
print(dir(factorial))
{% endhighlight %}

    ['__annotations__', '__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__get__', '__getattribute__', '__globals__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__kwdefaults__', '__le__', '__lt__', '__module__', '__name__', '__ne__', '__new__', '__qualname__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__']



{% highlight python %}
class C: pass
obj = C()
def func(): pass
print(sorted(set(dir(func)) - set(dir(obj))))
{% endhighlight %}

    ['__annotations__', '__call__', '__closure__', '__code__', '__defaults__', '__get__', '__globals__', '__kwdefaults__', '__name__', '__qualname__']


### KETNOTE
---
1. `dir()`<br>
    Without arguments, return the list of names in the **current local scope**.<br>
    With an argument, attempt to return a list of valid attributes for that object.
2. `pass` can be used when a statement is **required syntactically** but the program requires no action.<br>
    Also can be used as a place-holder to help think at a more abstract level.<br>
    Here is creating minimal classes/functions.
3. Making clever use of **set difference**.

## Functional Programming: `operator` and `functools` Package


{% highlight python %}
from functools import reduce
{% endhighlight %}

`reduce` applies a function of **two arguments** cumulatively to the items of a sequence,<br>
    from left to right, so as to reduce the sequence to a single value.
### `lambda`


{% highlight python %}
def fact(n):
    return reduce(lambda a, b: a * b, range(1, n+1))
{% endhighlight %}

### `mul`


{% highlight python %}
from operator import mul

def fact(n):
    return reduce(mul, range(1, n+1))
{% endhighlight %}

### `itemgetter`: sorting a list of tuples by the value of one field


{% highlight python %}
metro_data = [
('Tokyo', 'JP', 36.933, (35.689722, 139.691667)),
('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)),
('Mexico City', 'MX', 20.142, (19.433333, -99.133333)),
('New York-Newark', 'US', 20.104, (40.808611, -74.020386)),
('Sao Paulo', 'BR', 19.649, (-23.547778, -46.635833)),
]
{% endhighlight %}


{% highlight python %}
from operator import itemgetter
for city in sorted(metro_data, key = itemgetter(1)):
    print(city)
{% endhighlight %}

    ('Sao Paulo', 'BR', 19.649, (-23.547778, -46.635833))
    ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889))
    ('Tokyo', 'JP', 36.933, (35.689722, 139.691667))
    ('Mexico City', 'MX', 20.142, (19.433333, -99.133333))
    ('New York-Newark', 'US', 20.104, (40.808611, -74.020386))



{% highlight python %}
# multiple index arguments will return tuples with extracted values

selected_column = itemgetter(1, 0)
for city in metro_data:
    print(selected_column(city))
{% endhighlight %}

    ('JP', 'Tokyo')
    ('IN', 'Delhi NCR')
    ('MX', 'Mexico City')
    ('US', 'New York-Newark')
    ('BR', 'Sao Paulo')


### `attrgetter`: creating functions to extract object attributes by name


{% highlight python %}
student_data = [
    ('Olivia','23',('90','Excellent')),
    ('Jedi','207',('40','Not good')),
    ('Sophie','20',('85','Great'))
]

from collections import namedtuple
Performance = namedtuple('Performance', 'score comment')
Student = namedtuple('Student','name age grade')
{% endhighlight %}


{% highlight python %}
student_obj = [Student(name, age, Performance(score,comment))
                   for name, age, (score,comment) in student_data]
{% endhighlight %}

NOTE:
1. **nested** tuple unpacking to extract (score, comment)
2. build the `Performance(score,comment)` for the `grade` attribute of `Student`


{% highlight python %}
student_obj[0]
{% endhighlight %}




    Student(name='Olivia', age='23', grade=Performance(score='90', comment='Excellent'))




{% highlight python %}
student_obj[0].grade.comment
{% endhighlight %}




    'Excellent'




{% highlight python %}
from operator import attrgetter
name_grade = attrgetter('name','grade.comment')
for one in sorted(student_obj, key = attrgetter('name')):
    print(name_grade(one))
{% endhighlight %}

    ('Jedi', 'Not good')
    ('Olivia', 'Excellent')
    ('Sophie', 'Great')


### `methodcaller`: calling a method by name of the object given as argument


{% highlight python %}
from operator import methodcaller
s = 'hello'
make_upper = methodcaller('upper')
make_upper(s)
{% endhighlight %}




    'HELLO'



`methodcaller` can also do a partial application to freeze some arguments,
like the `functools.partial` function does.


{% highlight python %}
replace_e = methodcaller('replace', 'e', '-')
{% endhighlight %}


{% highlight python %}
replace_e(s)
{% endhighlight %}




    'h-llo'



### By the way, how to `print` all objects in a **list** elegently ?


{% highlight python %}
special_print = print(*list(s), sep = '-')
{% endhighlight %}

    h-e-l-l-o


### `functools.partial`: freezing arguments


{% highlight python %}
from operator import pow
from functools import partial

pow_7 = partial(pow, 7)
pow_7(2)
{% endhighlight %}




    49




{% highlight python %}
print([pow_7(x) for x in range(6)])
{% endhighlight %}

    [1, 7, 49, 343, 2401, 16807]


### What else in `operator` ?


{% highlight python %}
import operator
print([name for name in dir(operator) if not name.startswith('_')])
{% endhighlight %}

    ['abs', 'add', 'and_', 'attrgetter', 'concat', 'contains', 'countOf', 'delitem', 'eq', 'floordiv', 'ge', 'getitem', 'gt', 'iadd', 'iand', 'iconcat', 'ifloordiv', 'ilshift', 'imatmul', 'imod', 'imul', 'index', 'indexOf', 'inv', 'invert', 'ior', 'ipow', 'irshift', 'is_', 'is_not', 'isub', 'itemgetter', 'itruediv', 'ixor', 'le', 'length_hint', 'lshift', 'lt', 'matmul', 'methodcaller', 'mod', 'mul', 'ne', 'neg', 'not_', 'or_', 'pos', 'pow', 'rshift', 'setitem', 'sub', 'truediv', 'truth', 'xor']
