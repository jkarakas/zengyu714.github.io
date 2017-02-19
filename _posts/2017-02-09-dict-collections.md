---
author: kimmy
layout: post
title:  "Dict and Collections"
date:   2017-02-09 23:00:18 +08:00
categories: Python
tags:
- Python
- Notes
- Hash
---

* content
{:toc}



# Dict

## Build a `dict` Commonly


{% highlight python %}
a = dict(one=1, two=2, three=3)
b = {'one': 1, 'two': 2, 'three': 3}
c = dict(zip(['one','two','three'],[1,2,3]))
# don't need to type commas and quotes :)
c_plus = dict(zip('one two three'.split(),[1,2,3]))
a == b == c
{% endhighlight %}




    True



## Build with *dict comprehensions*


{% highlight python %}
# produce key:value pairs
colors = [
     ('Black','#000000'),
     ('White','#FFFFFF'),
     ('Red','#FF0000'),
     ('Lime','#00FF00'),
     ('Blue','#0000FF'),
     ('Yellow','#FFFF00'),
     ('Cyan','#00FFFF'),
     ('Magenta','#FF00FF')
]
{% endhighlight %}


{% highlight python %}
color_dict = {name: rgb for name, rgb in colors}
color_dict
{% endhighlight %}




    {'Black': '#000000',
     'Blue': '#0000FF',
     'Cyan': '#00FFFF',
     'Lime': '#00FF00',
     'Magenta': '#FF00FF',
     'Red': '#FF0000',
     'White': '#FFFFFF',
     'Yellow': '#FFFF00'}




{% highlight python %}
# directlt construct `{}`
{name.upper(): rgb for name, rgb in color_dict.items()
                   if len(name) > 4}
{% endhighlight %}




    {'BLACK': '#000000',
     'MAGENTA': '#FF00FF',
     'WHITE': '#FFFFFF',
     'YELLOW': '#FFFF00'}




{% highlight python %}
{name.upper(): rgb for name, rgb in color_dict.iteritems()}
{% endhighlight %}




    {'BLACK': '#000000',
     'BLUE': '#0000FF',
     'CYAN': '#00FFFF',
     'LIME': '#00FF00',
     'MAGENTA': '#FF00FF',
     'RED': '#FF0000',
     'WHITE': '#FFFFFF',
     'YELLOW': '#FFFF00'}



###  KEYNOTE
---
`iter()` returns a real list of 2-tuples ([(key, value), (key, value), ...])
`iteritems()` returns a generator "creates" (key, value) at a time

## `defaultdict`: solving missing keys


{% highlight python %}
import collections
dd = collections.defaultdict(list)
{% endhighlight %}

`default_factory`: Factory for default value called by `__missing__()`

### `__Missing__()`

> Example from Fluent Python Chapter 3


{% highlight python %}
class StrKeyDict0(dict):             # StrKeyDict0 inherits from dict

    def __missing__(self, key):
        if isinstance(key, str):     # check key is **already** a str
            raise KeyError(key)
        return self[str(key)]        # build str from key and look it up

    def get(self, key, default=None):# NOTE 1
        try:
            return self[key]
        except KeyError:             # NOTE 2
            return default

    def __contains__(self, key):     # NOTE 3
        return key in self.keys() or str(key) in self.keys()
{% endhighlight %}


{% highlight python %}
d = StrKeyDict0([(2, 'two'), ('4', 'four')])
print d.get(3)                       # NOTE 4
{% endhighlight %}

    None


### KEYNOTE
---
1. The `get` method delegates to `__getitem__` by using the `self[key]` notation;
    If key is not exist, `__missing__` method would be executed.
2. If a KeyError was raised, `__missing__` already failed, so we return the default.
3. Do not check for the key in the usual Pythonic way, `k in my_dict`, because
    `str(key) in self` would **recursively** call `__contains__`. We avoid this by explicitly
    looking up the `key in self.keys()`.

4. [CAREFULLY] : )<br>
    Must using `d.get(key)` notation, otherwise d[3] directly runs into `__missing__`, and `try...except...` didn't be triggered.

# Collections

## `isinstance()`


{% highlight python %}
import collections
my_dict = {}
print isinstance(my_dict, collections.Mapping)
print isinstance(my_dict, collections.MutableMapping)
{% endhighlight %}

    True
    True



{% highlight python %}
type(my_dict)
{% endhighlight %}




    dict



Using `isinstance` is better than checking `type(arg) == dict`,
because then alternative **mapping types** can be used.
Mapping types in the standard library implement the basic `dict`.In other words, `dict type` ∈ `Mapping type`

## `collections.OrderedDict`
`popitem` method pops the first item by default,
but if called as `my_odict.popitem(last=True)`, it pops the last item added.

## `collections.ChainMap`
Example of simulating Python’s internal lookup chain:


{% highlight python %}
# pip install future --upgrade
import builtins
# collections has no attribute 'ChainMap
pylookup = collections.ChainMap(locals(), globals(), vars(builtins))
{% endhighlight %}

## `collections.Counter`
is very helpful in data science, see [more](https://docs.python.org/3/library/collections.html#collections.Counter "Counter")

## Subclassing `UserDict`

StrKeyDict derived from `UserDict` always converts non-string keys to str: on insertion, update, and lookup.<br>
Compared to which is inherited from `Dict`.


{% highlight python %}
import collections
class StrKeyDict(collections.UserDict):

    def __missing__(self, key):
        if isinstance(key, str):
            raise KeyError(key)
        return self[str(key)]

    # Note 1
    def __contains__(self, key):
        return str(key) in self.data
    # Note 2
    def __setitem__(self, key, item):
        self.data[str(key)] = item
{% endhighlight %}

###  KEYNOTE
---
1. check on `self.data` instead of invoking `self.keys()` as we did in StrKeyDict0<br>
2. `__setitem__` converts any key to a str.
   This method is easier to overwrite when we can delegate to the self.data attribute

# Hashable
+ `__hash__()` method<br>
An object is hashable if it has a hash value which never changes during its lifetime.
+ `__eq__()` method<br>
It can be compared to other objects.

Hashable objects which compare equal must have the same hash value.



{% highlight python %}
absolutely_immutable_tuple = (1,2,(3,4))
hash(immutable_tuple_absolutely)
{% endhighlight %}




    1229485614




{% highlight python %}
partly_immutable_tuple = (1,2,[3,4])
hash(partly_immutable_tuple)
{% endhighlight %}


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-6-3af807c95300> in <module>()
          1 partly_immutable_tuple = (1,2,[3,4])
    ----> 2 hash(partly_immutable_tuple)


    TypeError: unhashable type: 'list'


Convert unhashable type `list` to hashable


{% highlight python %}
forcedly_hashable = (1,2,frozenset([3,4]))
hash(forcedly_hashable)
{% endhighlight %}




    -19024239
