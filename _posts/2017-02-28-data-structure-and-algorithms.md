---
layout: post
title:  Data Structures and Algorithms 101
date:   2017-02-28 12:07:55 +08:00
categories: Python
tags:
- Unpacking
- Generator Expressions
- Dict Comprehensions
- itertools
- lambda
- Counter
- zip
- defaultdict
---

* content
{:toc}



## Unpacking a Sequence into Separate Variables

###  Positional correspondence

{% highlight python %}
# number of variables and structure match the sequence

position = (0, 1)
x, y = position
# x = 0, y = 1
{% endhighlight %}

### Unpacking `str`

{% highlight python %}
# actually works with any iterable object not just tuples or lists,
# also includes strings, files, iterators, and generators.

a, b, c, d, e = 'Hello'
# a = 'H', b = 'e', ..., e = 'o'
{% endhighlight %}

### `_` placeholder to discard certain values.


{% highlight python %}
want_1, want_2, _ , want_3 = ['beautiful', 'execelent', 'tedious', 'wonderdful']
# discard `tedious`
{% endhighlight %}

## Unpacking Elements from Iterables of Arbitrary Length

### `*` star expression


{% highlight python %}
def drop_first_last(grades):
    first, *middle, last = grades
    return average(middle)
{% endhighlight %}

### Unpacking iterables with some known component or pattern


{% highlight python %}
data = [('char', 'a', 'b', 'c'),
        ('digit', '1', '0'),
        ('char', 'd', 'e', 'f')]

def do_char(a, b, c):
    print('char : ', a , b, c)
def do_digit(d1, d2):
    print('digit: ', d1, d2)

for flag, *args in data:
    if flag == 'char':
        do_char(*args)
    if flag == 'digit':
        do_digit(*args)
{% endhighlight %}

    char :  a b c
    digit:  1 0
    char :  d e f


### `*` combines with `_`


{% highlight python %}
want_1,*_ , want_2 = ['beautiful', 'boring', 'tedious', 'wonderdful']
# discard `boring` and `tedious`
{% endhighlight %}

## Keeping the Last N Items

### `collections.deque`<br>
+ `deque(maxlen=N)` creates a fixed-sized `queue`, or get an unbounded queue.<br>
When new items are added and the `queue` is full, the oldest item is automatically removed.
+ Adding or popping items from both sides of a queue has $$O(1)$$ complexity.


{% highlight python %}
from collections import deque
q = deque(maxlen=2)
q.append(1)
q.append(2)
q
{% endhighlight %}




    deque([1, 2])




{% highlight python %}
q.append(3)
q
{% endhighlight %}




    deque([2, 3])



## Finding the Largest or Smallest N Items

### `heapq.nlargest/nsmallest`<br>
+ works by first converting the data into a list where items are ordered as a **heap**.<br>
+ both accept a key parameter.

### `max/min`
+ is faster when we search a single item.

### `sorted(items)[:N]/[-N:]`
+ if `N` is about the same size as the collection itself.


{% highlight python %}
import heapq

nums = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2]
print(heapq.nlargest(3, nums))  # [42, 37, 23]
{% endhighlight %}


{% highlight python %}
print(heapq.nsmallest(3, nums)) # [-4, 1, 2]
{% endhighlight %}

---


{% highlight python %}
portfolio = [
{'name': 'IBM', 'shares': 100, 'price': 91.1},
{'name': 'AAPL', 'shares': 50, 'price': 543.22},
{'name': 'FB', 'shares': 200, 'price': 21.09},
{'name': 'HPQ', 'shares': 35, 'price': 31.75},
{'name': 'YHOO', 'shares': 45, 'price': 16.35},
{'name': 'ACME', 'shares': 75, 'price': 115.65}
]

cheap_3 = heapq.nsmallest(3, portfolio, key = lambda item: item['price'])
expensive_3 = heapq.nlargest(3, portfolio, key = lambda item: item['price'])
{% endhighlight %}

### `heapq.heappop`
+ `heap[0]` is always the smallest item.
+ requires $$O(log N)$$ operations.
+ together with `heapq.heappush` could implement a **priority queue**.

## Mapping Keys to Multiple Values in a Dictionary

### store in another container such as a `list` or `set`

{% highlight python %}
# keep the insertion order
store_in_list = {
    'a': [1, 2, 3],
    'b': [4, 5]
}
# eliminate duplicates and ignore order
store_in_set = {
    'a': {1, 2, 3},
    'b': {4, 5}
}
{% endhighlight %}

### `collections.defaultdict`

{% highlight python %}
from collections import defaultdict

dl = defaultdict(list)
dl['a'].append(1)
dl['a'].append(2)
dl['b'].append(3)

ds = default(set)
ds['a'].append(7)
ds['a'].append(8)
ds['b'].append(9)
{% endhighlight %}

### manually construct a multivalued dictionary

{% highlight python %}
d = {}
for key, value in pairs:
    if key not in d:
        d[key] = []
    d[key].append(value)

#---------------------------
# compare to defaultdict
#---------------------------

d = defaultdict(list)
for key, value in pairs:
    d[key].append(value)
{% endhighlight %}

## Keeping Dictionaries in Order

### `collections.OrderedDict`<br>
+ preserves the original insertion order of data when iterating.
+ internally maintains a doubly linked list, so is more than **twice** as large as a normal dictionary.
+ is very useful when the mapping is for later serializing or ecoding to a different format.


{% highlight python %}
from collections import OrderedDict

od = OrderedDict()
od['a'] = 1
od['d'] = 4
od['c'] = 3
od['b'] = 2

for key in od:
    print(key, od[key])
{% endhighlight %}

    a 1
    d 4
    c 3
    b 2


---


{% highlight python %}
import json
json.dumps(od)
{% endhighlight %}




    '{"a": 1, "d": 4, "c": 3, "b": 2}'



## Calculating with Dictionaries

### `zip` inverts the keys and values


{% highlight python %}
prices = {
'ACME': 45.23,
'AAPL': 612.78,
'IBM': 205.55,
'HPQ': 37.20,
'FB': 10.75
}
min_price = min(zip(prices.values(), prices.keys()))
# min_price is (10.75, 'FB')
max_price = max(zip(prices.values(), prices.keys()))
# max_price is (612.78, 'AAPL')
prices_sorted = sorted(zip(prices.values(), prices.keys()))
# [(10.75, 'FB'),
#  (37.2, 'HPQ'),
#  (45.23, 'ACME'),
#  (205.55, 'IBM'),
#  (612.78, 'AAPL')]
{% endhighlight %}


{% highlight python %}
prices_and_names = zip(prices.values(), prices.keys())
print(min(prices_and_names)) # √
print(max(prices_and_names)) # zip() creates an iterator that can only be consumed once
{% endhighlight %}

    (10.75, 'FB')



    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    <ipython-input-15-bbedaa8285ea> in <module>()
          1 prices_and_names = zip(prices.values(), prices.keys())
          2 print(min(prices_and_names)) # OK
    ----> 3 print(max(prices_and_names)) # ValueError: max() arg is an empty sequence


    ValueError: max() arg is an empty sequence


### `min/max` with `key`


{% highlight python %}
# common data reductions only process the keys, not the values.

min(prices) # Returns 'AAPL'
max(prices) # Returns 'IBM'
{% endhighlight %}


{% highlight python %}
# `k` here is exactly the name like `IBM`, `FB`,
# thus the indexing works, returns stock price used for comparison

min(prices, key=lambda k: prices[k]) # Returns 'FB'
max(prices, key=lambda k: prices[k]) # Returns 'AAPL'
{% endhighlight %}


{% highlight python %}
# getting the min value needs another lookup step

min_value = prices[min(prices, key=lambda k: prices[k])]
{% endhighlight %}

## Finding Commonalities in Two Dictionaries

### `d.keys()`
+ returns a keys-view object that exposes the keys.
+ supports common `set` operations such as unions, intersections, and differences.

### `d.items()`
+ returns an items-view object consisting of `(key, value)` pairs.
+ supports similar set operations.


{% highlight python %}
a = {
    'x' : 1,
    'y' : 2,
    'z' : 3
}
b = {
    'w' : 10,
    'x' : 11,
    'y' : 2
}
{% endhighlight %}


{% highlight python %}
# Find keys in common
a.keys() & b.keys()     # {'x', 'y'}

# Find different keys
a.keys() - b.keys()     # {'z'}

# Find (key, value) pairs in common
a.items() & b.items()   # { ('y', 2) }
{% endhighlight %}

## Determining the Most Frequently Occurring Items in a Sequence

###  `Counter.most_common`


{% highlight python %}
words = [
'look', 'into', 'my', 'eyes', 'look', 'into', 'my', 'eyes',
'the', 'eyes', 'the', 'eyes', 'the', 'eyes', 'not', 'around', 'the',
'eyes', "don't", 'look', 'around', 'the', 'eyes', 'look', 'into',
'my', 'eyes', "you're", 'under'
]

from collections import Counter
words_count = Counter(words)
top_three = words_count.most_common(3)
print(top_three)
{% endhighlight %}

    [('eyes', 8), ('the', 5), ('look', 4)]



{% highlight python %}
# a Counter is a dictionary that maps the items to the number of occurrences

words_count['look']    # 4
words_count['my']      # 3
{% endhighlight %}

###  `Counter.update`


{% highlight python %}
# `update` method to increase the count

more_words = ['why','are','you','not','looking','in','my','eyes']
words_count.update(more_words)
{% endhighlight %}

### `Counter` instances using mathmatical operations


{% highlight python %}
a = Counter(words)
a
{% endhighlight %}




    Counter({'around': 2,
             "don't": 1,
             'eyes': 8,
             'into': 3,
             'look': 4,
             'my': 3,
             'not': 1,
             'the': 5,
             'under': 1,
             "you're": 1})




{% highlight python %}
b = Counter(more_words)
b
{% endhighlight %}




    Counter({'are': 1,
             'eyes': 1,
             'in': 1,
             'looking': 1,
             'my': 1,
             'not': 1,
             'why': 1,
             'you': 1})




{% highlight python %}
# combine counts

c = a + b
c
{% endhighlight %}




    Counter({'are': 1,
             'around': 2,
             "don't": 1,
             'eyes': 9,
             'in': 1,
             'into': 3,
             'look': 4,
             'looking': 1,
             'my': 4,
             'not': 2,
             'the': 5,
             'under': 1,
             'why': 1,
             'you': 1,
             "you're": 1})




{% highlight python %}
# subtract counts

d = a - b
d
{% endhighlight %}




    Counter({'around': 2,
             "don't": 1,
             'eyes': 7,
             'into': 3,
             'look': 4,
             'my': 2,
             'the': 5,
             'under': 1,
             "you're": 1})



## Sorting a List of Dictionaries by a Common Key

### `operator.itemgetter`


{% highlight python %}
rows = [
{'fname': 'Brian', 'lname': 'Jones', 'uid': 1003},
{'fname': 'David', 'lname': 'Beazley', 'uid': 1002},
{'fname': 'John', 'lname': 'Cleese', 'uid': 1001},
{'fname': 'Big', 'lname': 'Jones', 'uid': 1004}
]

from operator import  itemgetter

rows_by_frame = sorted(rows, key=itemgetter('fname'))
rows_by_uid = sorted(rows, key=itemgetter('uid'))
{% endhighlight %}

+ accept nultiple keys


{% highlight python %}
rows_by_lfname = sorted(rows, key=itemgetter('lname','fname'))
{% endhighlight %}

+ applies to `min` and `max`


{% highlight python %}
min(rows, key=itemgetter('uid'))
{% endhighlight %}




    {'fname': 'John', 'lname': 'Cleese', 'uid': 1001}




{% highlight python %}
max(rows, key=itemgetter('uid'))
{% endhighlight %}




    {'fname': 'Big', 'lname': 'Jones', 'uid': 1004}



### `lambda` expression


{% highlight python %}
rows_by_lfname = sorted(rows, key=lambda r: (r['lname'], r['fname']))
# notice this lambda returns a tuple
{% endhighlight %}

## Filtering Sequence Elements

### list comprehensions


{% highlight python %}
mylist = [1, 4, -5, 10, -7, 2, 3, -1]
[x for x in mylist if x > 0]          # [1, 4, 10, 2, 3]
[x for x in mylist if x < 0]          # [-5, -7, -1]
{% endhighlight %}


{% highlight python %}
# replacing values
clip_neg = [x if x > 0 else 0 for x in mylist]   # [1, 4, 0, 10, 0, 2, 3, 0]

# transform values
make_square = [pow(x, 2) for x in mylist]        # [1, 16, 25, 100, 49, 4, 9, 1]
{% endhighlight %}

### generator expressions


{% highlight python %}
res = (x for x in mylist if x > 0)
res   # <generator object <genexpr> at 0x000001AD0D75E258>
for i in res:
    print(i)
{% endhighlight %}

    1
    4
    10
    2
    3


### `filter`


{% highlight python %}
values = ['1', '2', '-3', '-', '4', 'N/A', '5']

def is_int(val):
    try:
        x = int(val)
        return True
    except ValueError:
        return False

# filter() creates an iterator, use list to gshow values
int_val = list(filter(is_int, values))
print(int_val)
{% endhighlight %}

    ['1', '2', '-3', '4', '5']


### `itertools.compress`
+ picks out the items corresponding to `True` values.
+ normally returns an iterator, need to use `list()` to turn the results into a list if desired.

## Extracting a Subset of a Dictionary

### dictionary comprehensions


{% highlight python %}
prices = {
    'ACME': 45.23,
    'AAPL': 612.78,
    'IBM': 205.55,
    'HPQ': 37.20,
    'FB': 10.75
}

# select according value
price_over_200 = {key:value for key, value in prices.items() if value > 200}

# select according key
tech_names = { 'AAPL', 'IBM', 'HPQ', 'MSFT' }
tech_dict = {key:value for key, value in prices.items() if key in tech_names}
{% endhighlight %}

### pass `tuples` and to the `dict()` function


{% highlight python %}
price_over_200 = dict((key,value) for key, value in prices.items() if value > 200)
{% endhighlight %}

### tricky way


{% highlight python %}
tech_names = { 'AAPL', 'IBM', 'HPQ', 'MSFT' }
tech_dict = { key:prices[key] for key in prices.keys() & tech_names }
{% endhighlight %}

## Transforming and Reducing Data at the Same Time

### generator expressions


{% highlight python %}
# calculate the sum of squares
nums = [1, 2, 3, 4, 5]
print(sum(x * x for x in nums))
{% endhighlight %}

    55



{% highlight python %}
# determine if any .py files exist in a directory
import os
files = os.listdir('.')
if any(name.endswith(".py") for name in files):
    print("Here it is!")
else:
    print("Nop!")
{% endhighlight %}

    Nop!



{% highlight python %}
# output a tuple as CSV
s = ('ACME', 50, 123.45)
print(','.join(str(x) for x in s))
{% endhighlight %}

    ACME,50,123.45



{% highlight python %}
# data reduction across fields of a data structure
portfolio = [
    {'name':'GOOG', 'shares': 50},
    {'name':'YHOO', 'shares': 75},
    {'name':'AOL', 'shares': 20},
    {'name':'SCOX', 'shares': 65}
]

min_shares1 = min(s['shares'] for s in portfolio)
min_shares2 = min(portfolio, key = lambda s: s['shares'])
{% endhighlight %}

> Take Reference from [Python Cookbook](http://shop.oreilly.com/product/0636920027072.do)<br><br>
> CHAPTER 1<br>
> Data Structures and Algorithms
