---
author: kimmy
layout: post
title:  "Named Tuples"
date:   2017-02-08 23:30:06 +08:00
categories: Python
tags:
- Python
- Notes
---

* content
{:toc}


## Example: define named tuple about a student


{% highlight python %}
from collections import namedtuple
Student = namedtuple('Student','name age grade')
olivia = Student('olivia','20',('A+','Excellent'))
{% endhighlight %}

### Access the fields by name or position


{% highlight python %}
olivia[1]
{% endhighlight %}




    '20'




{% highlight python %}
olivia.age
{% endhighlight %}




    '20'



### `_fields` class attribute


{% highlight python %}
Student._fields
{% endhighlight %}




    ('name', 'age', 'grade')



### `_make(iterable)` class method
instantiate a named tuple from an iterable


{% highlight python %}
Performance = namedtuple('Performance','res comment')
sophie_res = ('sophie',21,Performance('A','Grade'))
sophie = Student._make(sophie_res)
print sophie
# student jedi doesn't use class Performance
jedi_res = ('jedi',90,('B','Good'))
jedi = Student._make(jedi_res)
print jedi
{% endhighlight %}

    Student(name='sophie', age=21, grade=Performance(res='A', comment='Grade'))
    Student(name='jedi', age=90, grade=('B', 'Good'))


### `_asdict()` instance method
can be used to produce a nice display of city data.


{% highlight python %}
sophie._asdict()
{% endhighlight %}




    OrderedDict([('name', 'sophie'),
                 ('age', 21),
                 ('grade', Performance(res='A', comment='Grade'))])




{% highlight python %}
for k, v in sophie._asdict().items():
    print k + ':', v
{% endhighlight %}

    name: sophie
    age: 21
    grade: Performance(res='A', comment='Grade')



{% highlight python %}
type(sophie._asdict()['grade'])
{% endhighlight %}




    __main__.Performance




{% highlight python %}
for k, v in jedi._asdict().items():
    print k + ':', v
{% endhighlight %}

    name: jedi
    age: 90
    grade: ('B', 'Good')



{% highlight python %}
type(jedi._asdict()['grade'])
{% endhighlight %}




    tuple



## KEYNOTES
---
1. `collections.namedtuple(...,verbose = True)` could see the details
2. build classes of objects that are just bundles of attributes with no custom methods
3. the use of `typename`

 {% highlight python %}
Point=namedtuple('whatsmypurpose',['x','y']) # (verbose=True) tells the answer
p_temp = Point(3,4)
print p_temp
{% endhighlight %}

    whatsmypurpose(x=3, y=4)
    <class '__main__.whatsmypurpose'>




<br>
> [Fluent Python](http://shop.oreilly.com/product/0636920032519.do "Fluent Python"), Chapter 2
