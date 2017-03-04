---
layout: post
title:  Strings and Text Part2
date:   2017-03-04 19:48:06 +08:00
categories: Python
tags:
- join
- textwrap
- Aligning
- strip
---

* content
{:toc}




## Stripping Unwanted Characters from Strings

### `str.strip()` / `str.lstrip()` / `str.rstrip()`
+ strip characters from the beginning or end/ left/ right side of a string.


{% highlight python %}
# strip whitespace
s = '     hello \n \t'
s.strip()
{% endhighlight %}




    'hello'




{% highlight python %}
s.lstrip()
{% endhighlight %}




    'hello \n \t'




{% highlight python %}
s.rstrip()
{% endhighlight %}




    '     hello'



+ strip **whitespace** by default, but other characters like **quotations** can be given.


{% highlight python %}
# strip characters
sc = '***hello***!!!'
sc.lstrip('*')
{% endhighlight %}




    'hello***!!!'




{% highlight python %}
sc.strip('*!')
{% endhighlight %}




    'hello'



+ does not apply to any text in the **middle** of a string.<br>
  `replace()` or a regular expression substitution can be used.


{% highlight python %}
s = 'hello?   world???   '
s.strip()
{% endhighlight %}




    'hello?   world???'



## Sanitizing and Cleaning Up Text

### `str.upper()` / `str.lower()`
+ converts text to a standard case.

### `str.replace()` or `re.sub()`
+ focuses on removing or changing very specific character sequences.

### `unicodedata.normalize()`
+ normalizes text.

### `str.translate()`


{% highlight python %}
s = 'pýtĥöñ\fis\tawesome\r\n'
s
{% endhighlight %}




    'pýtĥöñ\x0cis\tawesome\r\n'




{% highlight python %}
remap = {
    ord('\t') : ' ',
    ord('\f') : ' ',
    ord('\r') : None   # Deleted
}
a = s.translate(remap)
a
{% endhighlight %}




    'pýtĥöñ is awesome\n'



+ `str.replace()` is often the fastest approach.


{% highlight python %}
def clean_spaces(s):
    s = s.replace('\r', '')  # None
    s = s.replace('\t', ' ')
    s = s.replace('\f', ' ')
    return s
{% endhighlight %}


{% highlight python %}
print(clean_spaces(s))
{% endhighlight %}

    pýtĥöñ is awesome



## Aligning Text Strings

### `ljust()` / `rjust()` / `center()`


{% highlight python %}
s = 'Hello'
s.ljust(20)
{% endhighlight %}




    'Hello               '




{% highlight python %}
s.rjust(20)
{% endhighlight %}




    '               Hello'




{% highlight python %}
s.center(20)
{% endhighlight %}




    '       Hello        '



+ accpet an optional fill character as well.


{% highlight python %}
s.ljust(20, '~')
{% endhighlight %}




    'Hello~~~~~~~~~~~~~~~'




{% highlight python %}
s.center(20, '*')
{% endhighlight %}




    '*******Hello********'



### `format()`
+ could use `<`, `>`, or `^` characters along with a desired width.


{% highlight python %}
format(s, '<20')
{% endhighlight %}




    'Hello               '




{% highlight python %}
format(s, '>20')
{% endhighlight %}




    '               Hello'




{% highlight python %}
format(s, '^20')
{% endhighlight %}




    '       Hello        '



+ specifies a fill character before the alignment character.


{% highlight python %}
format(s, '~<20')
{% endhighlight %}




    'Hello~~~~~~~~~~~~~~~'




{% highlight python %}
format(s, '*>20')
{% endhighlight %}




    '***************Hello'




{% highlight python %}
format(s, '=^20')
{% endhighlight %}




    '=======Hello========'



+ formats **multiple** values.


{% highlight python %}
'{:~>10s} {:>10s}'.format('Hello', 'world')
{% endhighlight %}




    '~~~~~Hello      world'



+ works with any value like numbers.


{% highlight python %}
x = 1.2345
format(x, '>10')
{% endhighlight %}




    '    1.2345'




{% highlight python %}
format(x, '^10.2f')
{% endhighlight %}




    '   1.23   '



## Combining and Concatenating Strings

### `join()`
+ could specify delimiter.


{% highlight python %}
parts = ['How', 'are', 'you', 'today?']
' '.join(parts)
{% endhighlight %}




    'How are you today?'




{% highlight python %}
','.join(parts)
{% endhighlight %}




    'How,are,you,today?'




{% highlight python %}
''.join(parts)  # len('') == 0
{% endhighlight %}




    'Howareyoutoday?'



### `+` operator
+ is grossly inefficient due to the memory copies and garbage collection that occurs.


{% highlight python %}
a = 'How are you'
b = 'today?'
a + ' ' + b
{% endhighlight %}




    'How are you today?'



### conversion and concatenation at the same time


{% highlight python %}
data = ['TODAY', 2017, 3, 4]
'-'.join(str(c) for c in data )
{% endhighlight %}




    'TODAY-2017-3-4'



## Interpolating Variables in Strings

### `format()`


{% highlight python %}
s = '{name} is {age} years old.'
s.format(name='kimmy', age=21)
{% endhighlight %}




    'kimmy is 21 years old.'



### hide the variable substitution process


{% highlight python %}
import sys

class safesub(dict):
    def __missing_(self, key):
        return '{' + key + '}'

def sub(text):
    return text.format_map(safesub(sys._getframe(1).f_locals))
name = 'kimmy'
age = 21
print(sub('Hello, {name}'))
{% endhighlight %}

    Hello, kimmy


+ `sys._getframe()` returns a frame object from the call **stack**<br>
    The default for depth is zero, returning the frame at the top of the call stack. see [more](https://docs.python.org/3/library/sys.html#sys._getframe).
+ `f_locals` is a dictionary that is a copy of the local variables in the calling function.

## Reformatting Text to a Fixed Number of Columns

### `textwarp.fill()`
+ to reformat text for output.


{% highlight python %}
s = "Visualizing the Mandelbrot set doesn't have \
anything to do with machine learning, \
but it makes for a fun example of how one \
can use TensorFlow for general mathematics."

import textwrap
print(textwrap.fill(s, 70))
{% endhighlight %}

    Visualizing the Mandelbrot set doesn't have anything to do with
    machine learning, but it makes for a fun example of how one can use
    TensorFlow for general mathematics.



{% highlight python %}
print(textwrap.fill(s, 40))
{% endhighlight %}

    Visualizing the Mandelbrot set doesn't
    have anything to do with machine
    learning, but it makes for a fun example
    of how one can use TensorFlow for
    general mathematics.



{% highlight python %}
print(textwrap.fill(s, 40, initial_indent='    '))
{% endhighlight %}

        Visualizing the Mandelbrot set
    doesn't have anything to do with machine
    learning, but it makes for a fun example
    of how one can use TensorFlow for
    general mathematics.



{% highlight python %}
print(textwrap.fill(s, 40, subsequent_indent='    '))
{% endhighlight %}

    Visualizing the Mandelbrot set doesn't
        have anything to do with machine
        learning, but it makes for a fun
        example of how one can use
        TensorFlow for general mathematics.


### `os.get_terminal_size()`
+ gets size to fit the `print` output in the terminal.


{% highlight python %}
import os
os.get_terminal_size().columns
{% endhighlight %}




    100

    
<br><br><br>
> Take Reference from [Python Cookbook](http://shop.oreilly.com/product/0636920027072.do)<br><br>
> CHAPTER 2<br>
> Strings and Text
