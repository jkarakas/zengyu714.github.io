---
author: kimmy
layout: post
title:  "Python Data Model"
date:   2017-02-08 22:15:36 +08:00
categories: Python
tags:
- Python
- Notes
---

* content
{:toc}


## How to Behave as **sequences**
+  `__getitem__`
+  `__len__`

{% highlight python %}
import collections
# construct a class quickly
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
{% endhighlight %}

#### KEYNOTE
---
1. `collections.namedtuple()` creates tuple with **name** in each position
2. `list('JQKA')` and say, `'J Q K A'.split()` generate `list` beautifully

## Gain **iteration** and **index**

### Sample

{% highlight python %}
from random import choice
choice(deck)
# [out]:
# Card(rank='10', suit='diamonds')
{% endhighlight %}

### Index:
{% highlight python %}
deck[:3]
# [out]:
# [Card(rank='2', suit='spades'),
#  Card(rank='3', suit='spades'),
#  Card(rank='4', suit='spades')]
#----------------------------------------
# seq = L[start:stop]
deck[12:13]
# [out]:
# [Card(rank='A', suit='spades')]
#----------------------------------------
# seq = L[start:stop:step]
deck[12::13]
# [out]:
# [Card(rank='A', suit='spades'),
#  Card(rank='A', suit='diamonds'),
#  Card(rank='A', suit='clubs'),
#  Card(rank='A', suit='hearts')]
{% endhighlight %}

### Sort
{% highlight python %}
suit_values = dict(spades = 3, hearts = 2, diamonds = 1, clubs = 0)

def spades_high(card):
    rank_value = FrenchDeck.ranks.index(card.rank)
    return rank_value * len(suit_values) + suit_values[card.suit]
{% endhighlight %}

#### KEYNOTE
---
1. Assign `str` numerical values to make sort feasible.<br>
2. `a_list.index(a)` is similar to `index = find(A == a)` in Matlab.<br>
3.  Structure in FrenchDeck is basically a 2-D array and 'rank' represents 'rows' while 'suit' represents 'columns', so index here is `r * len(c) + c`.

## Emulating **numeric** Types

{% highlight python %}
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
{% endhighlight %}

#### KEYNOTE
---
1. `hypot(x,y)` returns $$\sqrt{x^2 + y^2}$$
2. `return bool(self.x or self.y)` better than `return bool(abs(self))`

## About **str** and **repr**
Difference between `__str__` and `__repr__` in Python, see [more](http://stackoverflow.com/questions/1436703/difference-between-str-and-repr-in-python "stackoverflow")

1. `__repr__` goal is to be unambiguous, understanding the object, placeholder `%r`
2. `__str__` goal is to be readable, optional implemention, placeholder `%s`
3. `__repr__` overrides `__str__`

## Summary
Implementing special methods like `__repr__`, `__getitem__`,<br>
could be more **generalizable** among available packages other than reinventing wheels.
