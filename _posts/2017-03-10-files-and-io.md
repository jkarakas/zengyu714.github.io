---
layout: post
title:  Files and I/O
date:   2017-03-10 13:23:19 +08:00
categories: Python
tags:
- os.path
- iter()
- print
---

* content
{:toc}



## Reading and Writing Text Data

### `open(filename, rt)`

Mode `rt` to read a text file, `wt` to write a new file and `at` to append an existing file.


{% highlight python %}
# Read the entire file as a single string
with open('somefile.txt', 'rt') as f:
    data = f.read()

# Iterate over the lines of the file
with open('somefile.txt', 'rt') as f:
    for line in f:
        # process the line
        # ...
{% endhighlight %}

### `sys.getdefaultencoding()`

Files are read/written using the system default text encoding.


{% highlight python %}
import sys
sys.getdefaultencoding()
{% endhighlight %}




    'utf-8'



### Discussion

+ Read with disabled newline translation<br>
    {% highlight python %}
    with open('somefile.txt', 'rt', newline='') as f:
        ...
    {% endhighlight %}
+ If get error like `UnicodeDecodeError: 'ascii' codec can't decode byte 0xc3 in position`, <br>
    use different encoding mode (e.g., reading data as `UTF-8` instead of `Latin-1` or whatever it needs to be).<br>
  Furthermore, supply an optional `errors` argument to `open()`:
    {% highlight python %}
    with open('somefile.txt', 'rt', encoding='ascii', errors='replace') as f:
        ...
    {% endhighlight %}

## Printing to a File

### `print(filename, file=f)`
+ redirects the output of the `print()` function to a file `f`.
+ Make sure that the file is opened in text mode.


{% highlight python %}
with open('somefile.txt', 'wt') as f:
    print('Hello World!', file=f)
{% endhighlight %}

## Printing with a Different Separator or Line Ending

### `print(data1, data2, data3, sep=__, end=__)`

`end` argument can also suppress the output of newlines.


{% highlight python %}
print(1, 2, 'a', 'b')
{% endhighlight %}

    1 2 a b



{% highlight python %}
print(1, 2, 'a', 'b', sep=',')
{% endhighlight %}

    1,2,a,b



{% highlight python %}
print(1, 2, 'a', 'b', sep=',', end='~~')
{% endhighlight %}

    1,2,a,b~~


{% highlight python %}
for i in range(3):
    print(i)
{% endhighlight %}

    0
    1
    2



{% highlight python %}
for i in range(3):
    print(i, end=' ')
{% endhighlight %}

    0 1 2

### `str.join()`

only works with `string`.


{% highlight python %}
print(','.join(['1', '2', 'a', 'b']))
{% endhighlight %}

    1,2,a,b



{% highlight python %}
mix = (1, 2, 'a', 'b')
print(','.join(mix))
{% endhighlight %}


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-16-63d32dce6f9c> in <module>()
          1 mix = (1, 2, 'a', 'b')
    ----> 2 print(','.join(mix))


    TypeError: sequence item 0: expected str instance, int found



{% highlight python %}
print(','.join(str(x) for x in mix))
{% endhighlight %}

    1,2,a,b



{% highlight python %}
# Notice the unpacking mark
print(*mix, sep=',')
{% endhighlight %}

    1,2,a,b


## Writing to a File That Doesn’t Already Exist

### `open(filename, 'xt')`
prevents from overwriting an existing file accidentally.


{% highlight python %}
with open('somefile', 'wt') as f:
    f.write('Hello\n')
with open('somefile', 'xt') as f:
    f.write('Hello\n')
{% endhighlight %}


    ---------------------------------------------------------------------------

    FileExistsError                           Traceback (most recent call last)

    <ipython-input-21-f70366d3d1a0> in <module>()
          1 with open('somefile', 'wt') as f:
          2     f.write('Hello\n')
    ----> 3 with open('somefile', 'xt') as f:
          4     f.write('Hello\n')


    FileExistsError: [Errno 17] File exists: 'somefile'


## Performing I/O Operations on a String

### `io.StringIO()`


{% highlight python %}
import io
s = io.StringIO()
s.write('Hello World\n')
print('Try to write data to the file-like object', file=s)
s.getvalue()
{% endhighlight %}




    'Hello World\nTry to write data to the file-like object\n'




{% highlight python %}
# Wrap a file interface around an existing string
s = io.StringIO('Hello\nWorld\n')
s.read(7)
{% endhighlight %}




    'Hello\nW'




{% highlight python %}
s.read()
{% endhighlight %}




    'orld\n'



### `io.BytesIO()`


{% highlight python %}
s = io.BytesIO()
s.write(b'hello')
s.getvalue()
{% endhighlight %}




    b'hello'



## Reading and Writing Compressed Datafiles

### `gzip.open()` / `bz2.open()`

## Iterating Over Fixed-Sized Records

### `iter()` and `functools.partial()`


{% highlight python %}
from functools import partial

RECORD_SIZE = 32

with open('somefile.data', 'rb') as f:
    records = iter(partial(f.read, RECORD_SIZE), b'')
    for r in records:
        ...
{% endhighlight %}

+ The `records` object in this example is an `iterable` that will produce fixed-sized chunks until the end of the file is reached.
+ `iter()` function is that it can create an iterator if you pass it a callable and a **sentinel** value (e.g. `b''`).

## Memory Mapping Binary Files

into a mutable byte array, possibly for random access to its contents or to make in-place modifications.


{% highlight python %}
import os
import mmap

# access argument: mmap.ACCESS_READ / mmap.ACCESS_COPY
def memory_map(filename, access=mmap.ACCESS_WRITE):
    size = os.path.getsize(filename)
    f = os.open(filename, os.O_RDWR)
    return mmap.mmap(f, size, access=access)
{% endhighlight %}

### How to create a file and expand it to a desired size ?


{% highlight python %}
size = 100000
with open('example.bin', 'wb') as f:
    f.seek(size-1)
    f.write(b'\x00')
{% endhighlight %}

### `memory_map()`


{% highlight python %}
m = memory_map('example.bin')
len(m)
{% endhighlight %}




    100000




{% highlight python %}
m[0:10]
{% endhighlight %}




    b'Hello, wor'




{% highlight python %}
m[0]
{% endhighlight %}




    72




{% highlight python %}
# Reassign a slice
m[:12] = b'Hello, world'
m.close()
{% endhighlight %}


{% highlight python %}
# Verify the changes
with open('example.bin', 'rb') as f:
    print(f.read(12))
{% endhighlight %}

    b'Hello, world'


### Use `mmap` object as a context manager


{% highlight python %}
with memory_map('example.bin') as m:
    print(len(m))
    print(m[0:12])
{% endhighlight %}

    100000
    b'Hello, world'


## Manipulating Pathnames

### `os.path`
+ is portable and knows about differences between Unix and Windows which can **reliably** deal with filenames such as `Data/data.csv` and `Data\data.csv`.
+ saves precious time.


{% highlight python %}
import os
path = "D:/Documents/Jupyter Notebook/Python Cookbook/somefile"
{% endhighlight %}


{% highlight python %}
# Get the last component of the path
os.path.basename(path)
{% endhighlight %}




    'somefile'




{% highlight python %}
# Get the directory name
os.path.dirname(path)
{% endhighlight %}




    'D:/Documents/Jupyter Notebook/Python Cookbook'




{% highlight python %}
# Join path components together
os.path.join('tmp', 'data', os.path.basename(path))
{% endhighlight %}




    'tmp\\data\\somefile'




{% highlight python %}
# Expand the user's home directory
path = '~/Python Cookbook/somefile'
os.path.expanduser(path)
{% endhighlight %}




    'C:\\Users\\lenovo/Python Cookbook/somefile'




{% highlight python %}
# Split the file extension
path = '~/Python Cookbook/somefile.bin'
os.path.splitext(path)
{% endhighlight %}




    ('~/Python Cookbook/somefile', '.bin')



## Testing for the Existence of a File

### `os.path.exists()`
tests whether or not a file or directory exists.


{% highlight python %}
import os
os.path.exists('D:/Documents/Jupyter Notebook/Python Cookbook')
{% endhighlight %}




    True




{% highlight python %}
os.path.exists('D:/Documents/Jupyter Notebook/Python Boxbook')
{% endhighlight %}




    False



### `os.path.getsize()` / `os.path.getmtime`


{% highlight python %}
os.path.getsize('D:/Documents/Jupyter Notebook/Python Cookbook/somefile')
{% endhighlight %}




    7




{% highlight python %}
# last modification time
os.path.getmtime('D:/Documents/Jupyter Notebook/Python Cookbook/somefile')
{% endhighlight %}




    1489109929.3288977




{% highlight python %}
# created time
os.path.getctime('D:/Documents/Jupyter Notebook/Python Cookbook/somefile')
{% endhighlight %}




    1489109929.325891




{% highlight python %}
import time
# last access time
time.ctime(os.path.getatime('D:/Documents/Jupyter Notebook/Python Cookbook/somefile'))
{% endhighlight %}




    'Fri Mar 10 09:38:49 2017'



### Others


{% highlight python %}
# Is a regular file
os.path.isfile()

# Is a directory
os.path.isdir()

# Is a symbolic link
os.path.islink()

# Get the file linked to
os.path.realpath()
{% endhighlight %}

## Getting a Directory Listing

### `os.listdir()`


{% highlight python %}
import os
names = os.listdir('somedir')

# Get all regular files
names = [name for name in os.listdir('somedir')
        if os.path.isfile(os.path.join('somedir', name))]

# Get all dirs
dirnames = [name for name in os.listdir('somedir')
           if os.path.isdir(os.path.join('somedir', name))]

# Get absolute path
absnames = [os.path.join('somedir', name) for name in os.listdir('somepath')]
{% endhighlight %}

## Serializing Python Objects

### `pickle.dump()` / `pickle.dumps()`


{% highlight python %}
import pickle

data = ... # Some Python object
f = open('somefile', 'wb')
pickle.dump(data, f)

# dump an object to a string
s = pickle.dumps(data)
{% endhighlight %}

### `pickle.load()` / `pickle.loads()`


{% highlight python %}
# Restore from a file
f = open('somefile', 'rb')
data = pickle.load(f)

# Restore from a string
data = pickle.loads(s)
{% endhighlight %}


<br><br><br>
> Take Reference from [Python Cookbook](http://shop.oreilly.com/product/0636920027072.do)<br><br>
> CHAPTER 5<br>
> Files and I/O
