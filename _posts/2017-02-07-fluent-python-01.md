---
author: kimmy
layout: post
title:  "Python Data Model"
date:   2017-02-07 18:57:36 +08:00
categories: Python
tags:
- Python
- Notes
---

* content
{:toc}


## Example 1  

### Add special methods to behave as *sequences*
+  `__getitem__`
+  `__len__`

```python  
import collections
# construct a simple class
Card = collections.namedtuple('Card',['rank','suit'])

class FrenchDeck:
  # elegent creation
  ranks = [str(n) for n in range(2,11)] + list('JQKA')
  suits = 'spades diamonds clubs hearts'.split()

  def __init__(self):
    self._cards = [Card(rank,suit) for rank in self.ranks
                                   for suit in self.suits]

  def __len__(self):
    return len(self._cards)

  def __getitem__(self,pos):
    return self._cards[pos]
# construct an object
deck = FrenchDeck()

```  

+ `collections.namedtuple()` creates tuple with **name** in each position. e.g.  

```Python
from collections import namedtuple
Point = namedtuple('Point',['x','y'])
p = Point(5,12)
```
  > `collections.namedtuple()`  can be used to build classes
  of objects that are just bundles of attributes with no custom methods, like a database record.

  Just like the `struct` in C with public access and the named fields but quiet easy-using and concise.  

+ `list('JQKA')` and say, `'J Q K A'.split()` generate `list` beautifully.

### Then gain *iteration* and *index* to play with  

#### sample  
```Python
from random import choice
choice(deck)
# [out]:
# Card(rank='10', suit='diamonds')
```
#### index:

```Python
deck[:3]
# [out]:
# [Card(rank='2', suit='spades'),
#  Card(rank='3', suit='spades'),
#  Card(rank='4', suit='spades')]

deck[12:13]
# [out]:
# [Card(rank='A', suit='spades')]

deck[12::13]
# [out]:
# [Card(rank='A', suit='spades'),
#  Card(rank='A', suit='diamonds'),
#  Card(rank='A', suit='clubs'),
#  Card(rank='A', suit='hearts')]
```  

**Supplement**: Accessing Lists  

+ L[i:j] returns a new list, containing the objects between i and j   

```python
n = len(L)
item = L[index]
seq = L[start:stop]
```  
+ Lists also support slice *steps*:  
  seq = L[start:stop:step]  

```python
seq = L[::2] # get every other item, starting with the first
seq = L[1::2] # get every other item, starting with the second
```    

#### sort   
```python  
suit_values = dict(spades = 3, hearts = 2, diamonds = 1, clubs = 0)  

def spades_high(card):
    rank_value = FrenchDeck.ranks.index(card.rank)
    return rank_value * len(suit_values) + suit_values[card.suit]
```   
+ Assign `str` numerical values to achieve support  
+ `one_list.index(a)` is similar to `index = find(A == a)` in Matlab
+ Structure in FrenchDeck is basically a 2-D array and 'rank' represents 'rows' while 'suit' represents 'columns', so index here is `r * len(c) + c`  

### Conclusion
> By implementing the special
methods \__len\__ and \__getitem\__, FrenchDeck behaves like a standard Python
sequence, allowing it to benefit from core language features (e.g., iteration and slicing)  


## Example 2    

### Emulating Numeric Types  

```python
from math import hypot

class Vector:
    def __init__(self,x = 0,y = 0):
        self.x = x
        self.y = y

    def __repr__(self):
        return 'Vector(%r,%r)'%(self.x,self.y)

    def __abs__(self):
        return hypot(self.x,self.y)

    def __bool__(self):
        return bool(self.x or self.y)

    def __add__(self,other):
        return Vector(self.x+other.x,self.y+other.y)

    def __mul__(self,scalar):
        return Vector(self.x*scalar,self.y*scalar)
```
+ `hypot(x,y)` returns sqrt(x^2 + y^2)
+ `return bool(self.x or self.y)` better than `return bool(abs(self))`  
### \__str__ and \__repr__
Difference between \__str\__ and \__repr\__ in Python, see [more](http://stackoverflow.com/questions/1436703/difference-between-str-and-repr-in-python "stackoverflow")  
 + \__repr\__ goal is to be unambiguous, understanding the object, placeholder `%r`
 + \__str\__ goal is to be readable, optional implemention, placeholder `%s`
 + \__repr\__ overrides \__str\__

## Summary  
> By implementing special methods, your objects can behave like the built-in types, enabling
the expressive coding style the community considers Pythonic

Be more **generalizable** among available packages other than reinventing wheels.
