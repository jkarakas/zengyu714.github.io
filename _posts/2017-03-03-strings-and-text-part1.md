---
layout: post
title:  Strings and Text Part1
date:   2017-03-03 20:08:44 +08:00
categories: Python
tags:
- re.match
- fnmatch

---

* content
{:toc}



## Splitting Strings on Any of Multiple Delimiters

### `re.split()`
+ gets the delimiter when a specified pattern like `;` `@` is matched.

### `str.split()`
+ does not allow for **multiple** delimiters or considering possible **whilespace** around delimiters.


{% highlight python %}
line = 'a; b, c d@ e* f$ g'
import re
re.split(r'[;,\s@*$]\s*', line)
{% endhighlight %}




    ['a', 'b', 'c', 'd', 'e', 'f', 'g']



## Matching Text at the Start or End of a String

### `str.startswith()` / `str.endswith()`
+ could accept a **tuple** of possible patterns to check against multiple choices.


{% highlight python %}
import os
filenames = os.listdir('.')
filenames
{% endhighlight %}




    ['.ipynb_checkpoints',
     'Data Structures and Algorithms 101.ipynb',
     'Strings and Text.ipynb']



+ **tuple** is required.


{% highlight python %}
choices = ['.ipynb','.h']
[name for name in filenames if name.endswith(choices)]
{% endhighlight %}


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-5-c948be51c5bb> in <module>()
          1 choices = ['.ipynb','.h']
    ----> 2 [name for name in filenames if name.endswith(choices)]


    <ipython-input-5-c948be51c5bb> in <listcomp>(.0)
          1 choices = ['.ipynb','.h']
    ----> 2 [name for name in filenames if name.endswith(choices)]


    TypeError: endswith first arg must be str or a tuple of str, not list



{% highlight python %}
choices = ['.ipynb','.h']
[name for name in filenames if name.endswith(tuple(choices))]
{% endhighlight %}




    ['Data Structures and Algorithms 101.ipynb', 'Strings and Text.ipynb']




{% highlight python %}
any(name.endswith('.ipynb') for name in filenames)
{% endhighlight %}




    True



+ look elegant when combined with other operations.


{% highlight python %}
if any(name.endswith(('.h','.py')) for name in os.listdir(dirname)):
    do_something()
{% endhighlight %}

### slice


{% highlight python %}
filename = 'foo.txt'
filename[-4:] == '.txt'
{% endhighlight %}




    True



### regular expressions


{% highlight python %}
import re
url = 'https://www.google.com.hk/webhp?hl=zh-CN'
re.match('http:|https|ftp:', url)
{% endhighlight %}




    <_sre.SRE_Match object; span=(0, 5), match='https'>



## Matching Strings Using Shell Wildcard Patterns

### `fnmatch()`
+ matches text using the same wildcard patterns as are commonly used in Unix shells <br>
  (e.g., `*.py`, `Dat[0-9]*.csv`, etc.).


{% highlight python %}
from fnmatch import fnmatch, fnmatchcase
fnmatch('foo.txt', '*.txt')
{% endhighlight %}




    True



{% highlight python %}
fnmatch('foo.txt','?oo.txt')
{% endhighlight %}




    True



{% highlight python %}
fnmatch('Dat45.csv', 'Dat[0-9]*')
{% endhighlight %}




    True



+ matching result varies based on operating system.


{% highlight python %}
# Windows
fnmatch('foo.txt', '*.TXT')
{% endhighlight %}




    True



### `fnmatchcase()`
+ matches exactly based on the lower/uppercase which is independent on the OS.


{% highlight python %}
fnmatchcase('foo.txt', '*.TXT')
{% endhighlight %}




    False



## Matching and Searching for Text Patterns

### `str.find()` / `str.endswith()` / `str.startswith()`


{% highlight python %}
# # Search for the location of the first occurrence
s = 'ni hao wa, ni jin tian zhen hao kan'
s.find('ni')
{% endhighlight %}




    0



### `re.match()`
+ `\d+` means match one or more digits.
+ only checks the beginning of a string.


{% highlight python %}
# How time flies!
day1 = '2017-03-03 '
day2 = 'Mar 03, 2017'

import re
for day in [day1, day2]:
    if re.match(r'\d+-\d+-\d+', day):
        print(day, ' : matches the pattern!')
    else:
        print(day, ': no!')
{% endhighlight %}

    2017-03-03   : matches the pattern!
    Mar 03, 2017 : no!


+ precompile the regular expression pattern into a pattern object first if used a lot.


{% highlight python %}
date_pattern = re.compile(r'\d+-\d+-\d+')
for day in [day1, day2]:
    if date_pattern.match(day):
        print(day, ' : matches the pattern!')
    else:
        print(day, ': no!')
{% endhighlight %}

    2017-03-03   : matches the pattern!
    Mar 03, 2017 : no!


### capture groups
+ by enclosing parts of the pattern in parentheses.
+ simplify subsequent processing, because the contents of each group can be extracted individually.


{% highlight python %}
date_pattern = re.compile(r'(\d+)-(\d+)-(\d+)')
agg = date_pattern.match('2017-03-03')
agg
{% endhighlight %}




    <_sre.SRE_Match object; span=(0, 10), match='2017-03-03'>




{% highlight python %}
# Extract the contents of each group
agg.group(0)
{% endhighlight %}




    '2017-03-03'




{% highlight python %}
agg.group(1)
{% endhighlight %}




    '2017'




{% highlight python %}
agg.group(2)
{% endhighlight %}




    '03'




{% highlight python %}
agg.group(3)
{% endhighlight %}




    '03'




{% highlight python %}
agg.groups()
{% endhighlight %}




    ('2017', '03', '03')




{% highlight python %}
year, month, day = agg.groups()
{% endhighlight %}

### `re.findall()`
+ searches the text and finds all matches, returning them as a `list`.


{% highlight python %}
time_stamp = 'Today is 2017-03-03. Reading "Python CookBook" starts from 2017-02-27.'
date_pattern.findall(time_stamp)
{% endhighlight %}




    ['2017-03-03', '2017-02-27']



+ notice results split into **tuples** when cooperates with groups.


{% highlight python %}
time_stamp = 'Today is 2017-03-03. Reading "Python CookBook" starts from 2017-02-27.'
date_pattern = re.compile(r'(\d+)-(\d+)-(\d+)')

date_pattern.findall(time_stamp)
{% endhighlight %}




    [('2017', '03', '03'), ('2017', '02', '27')]




{% highlight python %}
for year, month, day in date_pattern.findall(time_stamp):
    print('{}/{}/{}'.format(year, month, day))
{% endhighlight %}

    2017/03/03
    2017/02/27


### `re.finditer()`
+ find matches iteratively


{% highlight python %}
for agg in date_pattern.finditer(time_stamp):
    print(agg.groups())
{% endhighlight %}

    ('2017', '03', '03')
    ('2017', '02', '27')


## Searching and Replacing Text

### `str.replace()`


{% highlight python %}
s = 'ni hao wa, ni jin tian zhen hao kan'
s.replace('ni', 'wo')
{% endhighlight %}




    'wo hao wa, wo jin tian zhen hao kan'



### `re.sub()`
+ backslashed digits such as `\3` refer to capture group numbers in the pattern.


{% highlight python %}
time_stamp = 'Today is 2017-03-03. Reading "Python CookBook" starts from 2017-02-27.'
import re
re.sub(r'(\d+)-(\d+)-(\d+)', r'\2/\3/\1', time_stamp)
{% endhighlight %}




    'Today is 03/03/2017. Reading "Python CookBook" starts from 02/27/2017.'



## Searching and Replacing Case-Insensitive Text

### `re.IGNORECASE` flag


{% highlight python %}
text = 'UPPER PYTHON, lower python, Mixed Python'
re.findall('python', text, flags=re.IGNORECASE)
{% endhighlight %}




    ['PYTHON', 'python', 'Python']




{% highlight python %}
re.sub('python', 'anaconda', text, flags=re.IGNORECASE)
{% endhighlight %}




    'UPPER anaconda, lower anaconda, Mixed anaconda'



## Others

### Normalizing [Unicode](https://nedbatchelder.com/text/unipain.html) Text to a Standard Representation
`unicodedata.normalize()`

### Writing a Regular Expression for Multiline Patterns
`re.DOTALL` flag

<br><br><br>
> Take Reference from [Python Cookbook](http://shop.oreilly.com/product/0636920027072.do)<br><br>
> CHAPTER 2<br>
> Strings and Text
