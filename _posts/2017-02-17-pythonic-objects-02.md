---
author: kimmy
layout: post
title:  "Pythonic Object Attributes Part2"
date:   2017-02-17 23:30:19 +08:00
categories: Python
tags:
- Python
- Notes
- zip
- reduce
---


* content
{:toc}



## `reprlib` : produce limited-length representations


```python
def __repr__(self):
    components = reprlib.repr(self._components)      # 1
    components = components[components.find('['):-1] # 2
    return 'Vector({}).format(components)'
```

1. `generates, e.g., `array('d', [0.0, 1.0, 2.0, 3.0, 4.0, ...])`
2. remove prefix `array('d'` and trailing `)`
3. calling repr() on an object should never raise an exception

## `__len__` and `__getitem__`: constitute the **sequence protocol**


```python
class Vector:
    # many lines omitted
    # ...

    def __len__(self):
        return len(self._components)

    def __getitem__(self, index):
        return self._components[index]
```

### Problem
a slice of a `Vector` was not a `Vector` instance but a `array`


```python
v7 = Vector(range(7))
v7[1:4]
```




    array('d', [1.0, 2.0, 3.0])



### How Slicing Works


```python
class MySeq:
    def __getitem__(self, index):
        return index

s = MySeq()
```


```python
s[1]
```




    1




```python
s[1:4]
```




    slice(1, 4, None)




```python
s[1:4:2]
```




    slice(1, 4, 2)




```python
s[1:4:2,3]    # 1
```




    (slice(1, 4, 2), 3)




```python
s[1:4:2,3:5]  # 2
```




    (slice(1, 4, 2), slice(3, 5, None))



### Notice
---
1. The presence of **commas** inside the `[]` means `__getitem__` receives a tuple.
2. The tuple may even hold several **slice objects**.

### Solution


```python
import numbers
def __len__(self):
    return len(self._components)

def __getitem__(self, index):
    cls = type(self)
    if isinstance(index, slice):              # 1
        return cls(self._components[index])
    elif isinstance(index, numbers.Integral): # 2
        return self._components[index]
    else:
        msg = '{cls.__name__} indices must be integers'
        raise TypeError(msg.format(cls=cls))
```

### KEYNOTE
---
1. If the `index` argument is a `slice`, invoke the **class constructor** to
   build another `Vector` instance from a slice of the `_components` array.
2. If the `index` is an `int` or some other kind of integer,
   return the specific item from `_components`.
3. SKILLS to LEARN
   + `isinstance`: the test against `numbers.Integral`—an Abstract Base Class
   + ```msg = '{cls.__name__} indices must be integers'
     raise TypeError(msg.format(cls=cls))```


```python
v7 = Vector(range(7))
```


```python
v7[0]
```




    0.0




```python
v7[2:6]
```




    Vector([2.0, 3.0, 4.0, 5.0])




```python
v7[-1:]
```




    Vector([6.0])



NOTE
A slice of `len == 1` also creates a `Vector`.


```python
v7[1,2]
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-75-82fccfd8982a> in <module>()
    ----> 1 v7[1,2]


    <ipython-input-68-31a7ce988c85> in __getitem__(self, index)
         24         else:
         25             msg = '{cls.__name__} indices must be integers'
    ---> 26             raise TypeError(msg.format(cls=cls))
         27
         28     def __repr__(self):


    TypeError: Vector indices must be integers


## `__getattr__`: is invoked by the interpreter when attribute lookup fails


```python
shortcut_names = 'xyzt'

def __getattr__(self, name):
    cls = type(self)
    if len(name) == 1:
        pos = cls.shortcut_names.find(name)
        if 0 <= pos < len(self._components):          # 1
            return self._components[pos]
    msg = '{.__name__!r} object has no attribute {!r}'# 2
    raise AttributeError(msg.format(cls, name))
```


```python
def __setattr__(self, name, value):
    cls = type(self)
    if len(name) == 1:
        if name in cls.shortcut_names:
            error = 'readonly attribute {attr_name!r}'
        elif name.islower():
            error = "can't set attributes 'a' to 'z' in {cls_name!r}"
        else:
            error = ''
        if error:
            msg = error.format(cls_name=cls.__name__, attr_name=name)
            raise AttributeError(msg)   # 3
    super().__setattr__(name, value)    # 4
```

### KEYNOTE
---
1. The less-than signs could be **successive** in an inequation.
2. `{.__name__!r}`, NO EXTRA WHITESPACE.
3. If there is a nonblank error message, `raise AttributeError(msg)`.
4.  `super()` function provides a way to access methods of superclasses dynamically.
   Then call `__setattr__` on superclass for standard behavior.

> Very often when you implement `__getattr__`, you need to code `__setattr__` as well,
to avoid inconsistent behavior in your objects

## `__hash__`: apply the `^` operator to the hashes of every component

###  Calculating the accumulated xor


```python
# Solution 1
n = 0
for i in range(1, 6):
    n ^= i
n
```




    1




```python
# Solution 2
import functools
functools.reduce(lambda a, b: a^b, range(6))
```




    1




```python
# Solution 3
import operator
functools.reduce(operator.xor, range(6))
```




    1



`operator` provides the functionality of **all** Python infix operators in function form,
lessening the need for lambda.

### Awesome `zip`
makes it easy to iterate in parallel over two or more iterables by
returning tuples that you can unpack into variables, one for each item in the parallel inputs.

+ `zip` returns a generator that produces tuples on demand


```python
zip(range(3), 'ABC')
```




    <zip at 0x210e63cef88>



+ usually iterate over the generator, here for display


```python
list(zip(range(3), 'ABC'))
```




    [(0, 'A'), (1, 'B'), (2, 'C')]



+ `zip` will stop at the shortest operand.


```python
list(zip(range(3), 'ABC', [0.0, 1.1, 2.2, 3.3]))
```




    [(0, 'A', 0.0), (1, 'B', 1.1), (2, 'C', 2.2)]



+ `enumerate`: get the **index** conveniently


```python
seasons = ['Spring', 'Summer', 'Fall', 'Winter']
list(enumerate(seasons))
```




    [(0, 'Spring'), (1, 'Summer'), (2, 'Fall'), (3, 'Winter')]



+ As to `Vector`


```python
def __eq__(self, other):
    if len(self) != len(other):
        return False
    for a, b in zip(self, other):
        if a != b:
            return False
    return True
```


```python
# using zip and all
def __eq__(self, other):
    return len(self) == len(other) and all(a == b for a, b in zip(self, other))
```

### Details in `Vector` Class


```python
import functools
import operator

class Vector:
    # ...

    def __eq__(self, other):
        return len(self) == len(other) and all(a == b for a, b in zip(self, other))

    def __hash__(self):
        hashes = (hash(x) for x in self._components)
        return functools.reduce(operator.xor, hashes, 0)

    # ...

# -------------------------------------------------------
#####  use map makes the mapping step more visible  #####
#--------------------------------------------------------
#
#     def __hash__(self):
#         hashes = map(hash, self._components)
#         return functools.reduce(operator.xor, hashes)
#
# -------------------------------------------------------
```

___
It's a good practice to provide the third argument, `reduce(function, iterable, initializer)`.
`initializer` is the first argument in the reducing loop if the sequence is **empty**.
