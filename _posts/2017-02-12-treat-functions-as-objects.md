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


```python
def factorial(n):
    '''return n!'''
    return 1 if n < 2 else n*factorial(n-1)
```


```python
factorial(5)
```




    120




```python
factorial.__class__
```




    function




```python
print(type(factorial))
```

    <class 'function'>



```python
factorial.__doc__
```




    'return n!'




```python
help(factorial)
```

    Help on function factorial in module __main__:

    factorial(n)
        return n!




```python
map(factorial,range(11))
```




    <map at 0x2515a22a358>




```python
list(map(factorial,range(11)))
```




    [1, 1, 2, 6, 24, 120, 720, 5040, 40320, 362880, 3628800]



### KEYNOTE
---
1. `factorial.__class__` , `factorial.__doc__` are attributes of function objects.
2. `factorial` is an instance of the `function` class

## Higher-Order Functions: nested functions

### `sorted` and its optional `key` —— *one-argument* function


```python
mess = list('Higher-Order Functions: nested functions'.split())
print(sorted(mess, key = len))
```

    ['nested', 'functions', 'Functions:', 'Higher-Order']



```python
def reverse(word):    # one argument
    return word[::-1]
```


```python
sorted(mess)
```




    ['Functions:', 'Higher-Order', 'functions', 'nested']




```python
sorted(mess, key = reverse)
```




    ['Functions:', 'nested', 'Higher-Order', 'functions']



### Replace `map` and `filter`

with generator expressions and list comprehensions


```python
list(map(factorial,range(7)))
[factorial(x) for x in range(7)]
```




    [1, 1, 2, 6, 24, 120, 720]




```python
list(map(factorial,filter(lambda x: x & 0x1, range(7))))
[factorial(x) for x in range(7) if x % 2]
```




    [1, 6, 120]



### Use `sum` rather than `reduce`

Apply some operation with previous results in a sequence, until reducing a sequence of values to a single
value.
Others reducing functions like `any(iterabe)` , `any(iterable)`

## Anonymous Functions

The body part of lambda functions should be pure expressions.
Most used in the argument of higher-order functions


```python
sorted(mess, key = lambda x : x[::-1])
```




    ['Functions:', 'nested', 'Higher-Order', 'functions']



## User-Defined Callable Types
> Following example from Fluent Python chapter 5
A randomPick class does one thing: picks items from a shuffled list


```python
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
```


```python
test = randomPick(range(7))
test.pick()
```




    2




```python
test()         # same as test.pick()
```




    6




```python
callable(test) # 3
```




    True



### KEYNOTE
---
1. `__init__` accepts any iterable; building a local copy prevents unexpected side
    effects on any list passed as an argument
2. — `randomPick()` is an alias of `randomPick.pick()`.
  — Implementing `__call__` is an easy way to create function-like objects that have
    some **internal state** that must be kept across invocations.

3. `callable()` built-in to identify whether a callable object.

## Function-specific Attributes
`set(function) - set(user-defined object)`


```python
print(dir(factorial))
```

    ['__annotations__', '__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__get__', '__getattribute__', '__globals__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__kwdefaults__', '__le__', '__lt__', '__module__', '__name__', '__ne__', '__new__', '__qualname__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__']



```python
class C: pass
obj = C()
def func(): pass
print(sorted(set(dir(func)) - set(dir(obj))))
```

    ['__annotations__', '__call__', '__closure__', '__code__', '__defaults__', '__get__', '__globals__', '__kwdefaults__', '__name__', '__qualname__']


### KETNOTE
---
1. `dir()`
    Without arguments, return the list of names in the **current local scope**.
    With an argument, attempt to return a list of valid attributes for that object.
2. `pass` can be used when a statement is **required syntactically** but the program requires no action.
    Also can be used as a place-holder to help think at a more abstract level.
    Here is creating minimal classes/functions.
3. Making clever use of **set difference**.

## Functional Programming: `operator` and `functools` Package


```python
from functools import reduce
```

`reduce` applies a function of **two arguments** cumulatively to the items of a sequence,
    from left to right, so as to reduce the sequence to a single value.
### `lambda`


```python
def fact(n):
    return reduce(lambda a, b: a * b, range(1, n+1))
```

### `mul`


```python
from operator import mul

def fact(n):
    return reduce(mul, range(1, n+1))
```

### `itemgetter`: sorting a list of tuples by the value of one field


```python
metro_data = [
('Tokyo', 'JP', 36.933, (35.689722, 139.691667)),
('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)),
('Mexico City', 'MX', 20.142, (19.433333, -99.133333)),
('New York-Newark', 'US', 20.104, (40.808611, -74.020386)),
('Sao Paulo', 'BR', 19.649, (-23.547778, -46.635833)),
]
```


```python
from operator import itemgetter
for city in sorted(metro_data, key = itemgetter(1)):
    print(city)
```

    ('Sao Paulo', 'BR', 19.649, (-23.547778, -46.635833))
    ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889))
    ('Tokyo', 'JP', 36.933, (35.689722, 139.691667))
    ('Mexico City', 'MX', 20.142, (19.433333, -99.133333))
    ('New York-Newark', 'US', 20.104, (40.808611, -74.020386))



```python
# multiple index arguments will return tuples with extracted values

selected_column = itemgetter(1, 0)
for city in metro_data:
    print(selected_column(city))
```

    ('JP', 'Tokyo')
    ('IN', 'Delhi NCR')
    ('MX', 'Mexico City')
    ('US', 'New York-Newark')
    ('BR', 'Sao Paulo')


### `attrgetter`: creating functions to extract object attributes by name


```python
student_data = [
    ('Olivia','23',('90','Excellent')),
    ('Jedi','207',('40','Not good')),
    ('Sophie','20',('85','Great'))
]

from collections import namedtuple
Performance = namedtuple('Performance', 'score comment')
Student = namedtuple('Student','name age grade')
```


```python
student_obj = [Student(name, age, Performance(score,comment))
                   for name, age, (score,comment) in student_data]
```

NOTE:
1. **nested** tuple unpacking to extract (score, comment)
2. build the `Performance(score,comment)` for the `grade` attribute of `Student`


```python
student_obj[0]
```




    Student(name='Olivia', age='23', grade=Performance(score='90', comment='Excellent'))




```python
student_obj[0].grade.comment
```




    'Excellent'




```python
from operator import attrgetter
name_grade = attrgetter('name','grade.comment')
for one in sorted(student_obj, key = attrgetter('name')):
    print(name_grade(one))
```

    ('Jedi', 'Not good')
    ('Olivia', 'Excellent')
    ('Sophie', 'Great')


### `methodcaller`: calling a method by name of the object given as argument


```python
from operator import methodcaller
s = 'hello'
make_upper = methodcaller('upper')
make_upper(s)
```




    'HELLO'



`methodcaller` can also do a partial application to freeze some arguments,
like the `functools.partial` function does.


```python
replace_e = methodcaller('replace', 'e', '-')
```


```python
replace_e(s)
```




    'h-llo'



### By the way, how to `print` all objects in a list elegently ?


```python
special_print = print(*list(s), sep = '-')
```

    h-e-l-l-o


### `functools.partial`: freezing arguments


```python
from operator import pow
from functools import partial

pow_7 = partial(pow, 7)
pow_7(2)
```




    49




```python
print([pow_7(x) for x in range(6)])
```

    [1, 7, 49, 343, 2401, 16807]


### What else in `operator` ?


```python
import operator
print([name for name in dir(operator) if not name.startswith('_')])
```

    ['abs', 'add', 'and_', 'attrgetter', 'concat', 'contains', 'countOf', 'delitem', 'eq', 'floordiv', 'ge', 'getitem', 'gt', 'iadd', 'iand', 'iconcat', 'ifloordiv', 'ilshift', 'imatmul', 'imod', 'imul', 'index', 'indexOf', 'inv', 'invert', 'ior', 'ipow', 'irshift', 'is_', 'is_not', 'isub', 'itemgetter', 'itruediv', 'ixor', 'le', 'length_hint', 'lshift', 'lt', 'matmul', 'methodcaller', 'mod', 'mul', 'ne', 'neg', 'not_', 'or_', 'pos', 'pow', 'rshift', 'setitem', 'sub', 'truediv', 'truth', 'xor']
