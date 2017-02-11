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


```python
from collections import namedtuple
Student = namedtuple('Student','name age grade')
olivia = Student('olivia','20',('A+','Excellent'))
```

### Access the fields by name or position


```python
olivia[1]
```




    '20'




```python
olivia.age
```




    '20'



### `_fields` class attribute


```python
Student._fields
```




    ('name', 'age', 'grade')



### `_make(iterable)` class method
instantiate a named tuple from an iterable


```python
Performance = namedtuple('Performance','res comment')
sophie_res = ('sophie',21,Performance('A','Grade'))
sophie = Student._make(sophie_res)
print sophie
# student jedi doesn't use class Performance
jedi_res = ('jedi',90,('B','Good'))
jedi = Student._make(jedi_res)
print jedi
```

    Student(name='sophie', age=21, grade=Performance(res='A', comment='Grade'))
    Student(name='jedi', age=90, grade=('B', 'Good'))


### `_asdict()` instance method
can be used to produce a nice display of city data.


```python
sophie._asdict()
```




    OrderedDict([('name', 'sophie'),
                 ('age', 21),
                 ('grade', Performance(res='A', comment='Grade'))])




```python
for k, v in sophie._asdict().items():
    print k + ':', v
```

    name: sophie
    age: 21
    grade: Performance(res='A', comment='Grade')



```python
type(sophie._asdict()['grade'])
```




    __main__.Performance




```python
for k, v in jedi._asdict().items():
    print k + ':', v
```

    name: jedi
    age: 90
    grade: ('B', 'Good')



```python
type(jedi._asdict()['grade'])
```




    tuple



## KEYNOTES
---
1. `collections.namedtuple(...,verbose = True)` could see the details
2. build classes of objects that are just bundles of attributes with no custom methods
3. the use of `typename`

 ```python
Point=namedtuple('whatsmypurpose',['x','y']) # (verbose=True) tells the answer
p_temp = Point(3,4)
print p_temp
```

    whatsmypurpose(x=3, y=4)
    <class '__main__.whatsmypurpose'>




<br>
> [Fluent Python](http://shop.oreilly.com/product/0636920032519.do "Fluent Python"), Chapter 2
