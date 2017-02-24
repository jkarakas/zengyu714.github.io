---
author: kimmy
layout: post
title:  "Iterables, Iterators, and Generators"
date:   2017-02-24 21:08:12 +08:00
categories: Python
tags:
- Python
- Notes
- iterator
- generator
- fibonacci
---


* content
{:toc}



Every generator is an iterator: generators fully implement the iterator interface
+ an iterator retrieves items from a collection.
+ a generator can produce items “out of thin air”.

## `Sentence` Example

### 1. Implement the Sequence Protocol


{% highlight python %}
import re
import reprlib

PROG = re.compile('\w+')

class Sentence:

    def __init__(self, text):
        self.text = text
        self.words = PROG.findall(text)

    def __getitem__(self, index):
        return self.words[index]

    def __len__(self):
        return len(self.words)

    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text)
{% endhighlight %}

+ `re.findall(pattern, string, flags=0)` returns all non-overlapping matches of pattern in string, as a **list** of strings.
+ `reprlib.repr` generates **abbreviated** string representations of data structures that can be very large.<br>
    By default, `reprlib.repr` limits the generated string to 30 characters.
+ `__getitem__` makes it `iterable`.


{% highlight python %}
s = Sentence('Join them; it only takes a minute.')
s
{% endhighlight %}




    Sentence('Join them; i...kes a minute.')




{% highlight python %}
for word in s:
    print(word)
{% endhighlight %}

    Join
    them
    it
    only
    takes
    a
    minute



{% highlight python %}
list(s)
{% endhighlight %}




    ['Join', 'them', 'it', 'only', 'takes', 'a', 'minute']



---

### `iterable` Versus `iterator`
see [source](http://stackoverflow.com/questions/9884132/what-exactly-are-pythons-iterator-iterable-and-iteration-protocols).
+ An `iterable ` is an object that has an `__iter__` method that instantiates a new `iterator` every time, <br>
or which defines a  `__getitem__` method that can take sequential indexes starting from zero (and raises an `IndexError` when the indexes are no longer valid). <br>
So an `iterable ` is an object that you can get an `iterator` from. <br>
+ `iterator` is an object with a`__next__` method that returns individual items, and an `__iter__` method that returns self.<br>
  `isinstance(x, abc.Iterator)`: the best way to check if an object `x` is an `iterator`.


{% highlight python %}
s = Sentence("hello and goodbye")

# s is an ITERABLE
# s is a Sentence object that is immutable
# s has no state
# s has a __getitem__() method
{% endhighlight %}


{% highlight python %}
it = iter(s)
it

# it is an ITERATOR
# it has state ( starts by pointing at the "hello"
# it has a next() method and an __iter__() method
{% endhighlight %}




    <iterator at 0x1abf359d278>




{% highlight python %}
# behave similar as *(iter++)
# next(it) fetches the next word

next(it)
{% endhighlight %}




    'hello'




{% highlight python %}
next(it)
{% endhighlight %}




    'and'




{% highlight python %}
next(it)
{% endhighlight %}




    'goodbye'




{% highlight python %}
# no words any more
# raises a StopIteration exception

next(it)
{% endhighlight %}


    ---------------------------------------------------------------------------

    StopIteration                             Traceback (most recent call last)

    <ipython-input-23-2cdb14c0d4d6> in <module>()
    ----> 1 next(it)


    StopIteration:



{% highlight python %}
# an iterator becomes useless once exhausted

list(it)
{% endhighlight %}




    []




{% highlight python %}
# generate a new iterator
# to go over the sentence again

list(iter(s))
{% endhighlight %}




    ['hello', 'and', 'goodbye']



### 2: A Classic Iterator


{% highlight python %}
import re
import reprlib

RE_WORD = re.compile('\w+')

class Sentence:

    def __init__(self, text):
        self.text = text
        self.words = RE_WORD.findall(text)

    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text)

    def __iter__(self):                       # 1
        return SentenceIterator(self.words)   # 2

class SentenceIterator:

    def __init__(self, words):
        self.words = words                    # 3
        self.index = 0

    def __next__(self):                       # 4
        try:
            word = self.words[self.index]
        except IndexError:
            raise StopIteration()
        self.index += 1
        return word

    def __iter__(self):
        return self
{% endhighlight %}

1. `Sentence` class is iterable because it implements `__iter__`.
2. `__iter__` fulfills the iterable protocol by instantiating and returning an iterator.
3. `SentenceIterator` holds a reference to the list of words.
4. An `iterable` should never act as an iterator over itself. <br>
   In other words, `iterables` must implement `__iter__`, but not `__next__`.

---

### How a Generator Function Works

Any Python function that has the `yield` keyword in its body is a generator function, <br>
which replaces the `return` of a function to provide a generator object to its caller without destroying local variables.


{% highlight python %}
def gen_123():
    # the occurence of `yield` indicates it's a generator function
    # usually the body of a generator function has a loop, but not necessarily
    yield 1
    yield 2
    yield 3
{% endhighlight %}


{% highlight python %}
# a function object
gen_123
{% endhighlight %}




    <function __main__.gen_123>




{% highlight python %}
# a generator object when invoked
gen_123()
{% endhighlight %}




    <generator object gen_123 at 0x000001ABF363D4C0>




{% highlight python %}
for i in gen_123():
    print(i)
{% endhighlight %}

    1
    2
    3



{% highlight python %}
g = gen_123()
{% endhighlight %}


{% highlight python %}
# fetch the next item produced by **yield**
next(g)
{% endhighlight %}




    1




{% highlight python %}
next(g)
{% endhighlight %}




    2




{% highlight python %}
next(g)
{% endhighlight %}




    3




{% highlight python %}
next(g)
{% endhighlight %}


    ---------------------------------------------------------------------------

    StopIteration                             Traceback (most recent call last)

    <ipython-input-39-5f315c5de15b> in <module>()
    ----> 1 next(g)


    StopIteration:


Makes the interaction between a `for` loop and the body of the function more explicit.


{% highlight python %}
def gen_AB():
    print('start')
    yield 'A' #
    print('continue')
    yield 'B' #
    print('end.')
{% endhighlight %}


{% highlight python %}
for c in gen_AB():
    print('--->', c)
{% endhighlight %}

    start
    ---> A
    continue
    ---> B
    end.


To iterate, the `for` machinery does the equivalent of `g = iter(gen_AB())` to get a generator object,<br>
and then `next(g)` at each iteration.


{% highlight python %}
g = gen_AB()
next(g)
{% endhighlight %}

    start
    [out] 'A'




{% highlight python %}
next(g)
{% endhighlight %}

    continue
    [out] 'B'




{% highlight python %}
next(g)
{% endhighlight %}

    end.
    [out] ---------------------------------------------------------------------

    StopIteration                             Traceback (most recent call last)

    <ipython-input-59-5f315c5de15b> in <module>()
    ----> 1 next(g)


    StopIteration:


### 3: Using A Generator Function


{% highlight python %}
import re
import reprlib

RE_WORD = re.compile('\w+')
class Sentence:
    def __init__(self, text):
        self.text = text
        self.words = RE_WORD.findall(text)

    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text)

    def __iter__(self):
        for word in self.words:     # 1
            yield word
        return
{% endhighlight %}

+ `__iter__` in section 2 called the `SentenceIterator` constructor to build an iterator and return it.<br>
+ `# 1` above is in fact a generator object, built automatically when the `__iter__` method is called.<br>
    Because `__iter__` here is a generator function.

---

### When to Use Generator Expressions

Generator expressions <br>
+ are syntactic sugar.
+ they can always be replaced by generator functions, but sometimes are more convenient.

Generator functions<br>
+ could have name for reuse.
+ are more readability if the generator expression spans more than a couple of lines.


{% highlight python %}
# list comprehension
# eagerly iterates over the items yiedlded by
# the generator object produced calling gen_AB(): `A` and `B`

res1 = [x*3 for x in gen_AB()]
{% endhighlight %}

    start
    continue
    end.



{% highlight python %}
# This for loop iterates over the res1 **list**
# produced by the list comprehension.

for i in res1:
    print('-->', i)
{% endhighlight %}

    --> AAA
    --> BBB



{% highlight python %}
# generator expression
# res2 is a generator object
# enclosed in plain parentheses ()

res2 = (x*3 for x in gen_AB())
res2
{% endhighlight %}




    <generator object <genexpr> at 0x000001ABF363DC50>




{% highlight python %}
# only when the for loop iterates over res2, the body of gen_AB actually executes
# Each iteration of the for loop implicitly calls next(res2), advancing gen_AB to the next yield.

for i in res2:
    print('-->', i)
{% endhighlight %}

    start
    --> AAA
    continue
    --> BBB
    end.


### 4: A Lazy Implementation


{% highlight python %}
import re
import reprlib

RE_WORD = re.compile('\w+')

class Sentence:

    def __init__(self, text):
        self.text = text
        # no need to have a words list

    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text)

    #--------------------------------------------------------
    # def __iter__(self):
    #     for match in RE_WORD.finditer(self.text):
    #         yield match.group()
    #--------------------------------------------------------


    def __iter__(self):
        return (match.group() for match in RE_WORD.finditer(self.text))
{% endhighlight %}

+ `finditer` builds an iterator over the matches of `RE_WORD` on `self.text`, yielding `MatchObject` instances.
+ `__iter__` method is not a generator function (it has no `yield`), <br>
    but uses a generator expression `()` to build a generator and then returns it. <br>
+ The end result is the same: the caller of `__iter__` gets a generator object.

## A Closer Look at the `iter` Function

+ `iter(x)`: iterates over an object `x`.
+ `iter(func, sentinel)`: creates an iterator from a regular function or any callable object.<br>
    Sentinel is a marker value which, when returned by the callable, causes the iterator to raise `StopIteration`.


### Example 1: roll a six-sided die until a 1 is rolled


{% highlight python %}
import random
def dice_with_6_sides():
    return random.randint(1, 6)
stop_when_1_occurs = iter(dice_with_6_sides, 1)
{% endhighlight %}


{% highlight python %}
stop_when_1_occurs
{% endhighlight %}




    <callable_iterator at 0x1d8728b6198>




{% highlight python %}
for roll in stop_when_1_occurs:
    print(roll)
{% endhighlight %}

    5
    2
    5


### Example 2: read lines until a blank line is found or reach the end


{% highlight python %}
with open('my_data.txt') as fp:
    for line in iter(fp.readlien, ''):
        process_line(line)
{% endhighlight %}

## Pythonic `fibonacci` Generator


{% highlight python %}
def fibonacci():
    prev, cur = 0, 1
    while True:
        yield cur
        prev, cur = cur, prev + cur
{% endhighlight %}


{% highlight python %}
import itertools
list(itertools.islice(fibonacci(), 7))
{% endhighlight %}




    [1, 1, 2, 3, 5, 8, 13]




> Take Reference from Fluent Python Chapter 14
