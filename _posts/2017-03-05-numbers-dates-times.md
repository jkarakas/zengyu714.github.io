---
layout: post
title:  Numbers, Dates, and Times
date:   2017-03-05 19:07:22 +08:00
categories: Python
tags:
- str.format
- Decimal
- format
- random
- datetime
---

* content
{:toc}



## Rounding Numerical Values

### `round(value, ndigits)`


{% highlight python %}
round(1.234, 1)
{% endhighlight %}




    1.2




{% highlight python %}
round(1.2345, 3)
{% endhighlight %}




    1.234



+ The behavior of `round` is to round to the nearest **even** digit.


{% highlight python %}
round(1.5)
{% endhighlight %}




    2




{% highlight python %}
round(2.5)
{% endhighlight %}




    2



+ when `ndigits` is negative, `round` will keep place for tens, hundreds, thousands and so on.


{% highlight python %}
round(1234567, -2)
{% endhighlight %}




    1234600




{% highlight python %}
round(1234567, -5)
{% endhighlight %}




    1200000



### `str.format()`
+ **outputs** a numerical value with a certain number of decimal places.


{% highlight python %}
x = 1.234567
format(x, '.2f')
{% endhighlight %}




    '1.23'




{% highlight python %}
format(x, '.4f')
{% endhighlight %}




    '1.2346'




{% highlight python %}
'value is {:.3f}'.format(x)
{% endhighlight %}




    'value is 1.235'



## Performing Accurate Decimal Calculations

+ Floating-point numbers can't be represented accurately on base-10 decimals.
+ The performance of native floats is significantly faster, especially if you're performaing a large number of calculations.


{% highlight python %}
a = 4.2
b = 2.1
a + b
{% endhighlight %}




    6.300000000000001




{% highlight python %}
a + b == 6.3
{% endhighlight %}




    False



### `decimal.Decimal()`
+ is especially useful when accessing **financial** data.


{% highlight python %}
from decimal import Decimal
a = Decimal('4.2')
b = Decimal('2.1')
a + b
{% endhighlight %}




    Decimal('6.3')




{% highlight python %}
print(a + b)
{% endhighlight %}

    6.3



{% highlight python %}
(a + b) == Decimal('6.3')
{% endhighlight %}




    True



### `decimal.localcontext()`

+ allows you to control different aspects of calculations, including number of digits and rounding,<br>
    by creating a local context and change its settings.


{% highlight python %}
from decimal import localcontext
a = Decimal('1.3')
b = Decimal('1.7')
print(a / b)
{% endhighlight %}

    0.7647058823529411764705882353



{% highlight python %}
with localcontext() as ctx:
    ctx.prec = 3
    print(a / b)
{% endhighlight %}

    0.765



{% highlight python %}
with localcontext() as ctx:
    ctx.prec = 50
    print(a / b)
{% endhighlight %}

    0.76470588235294117647058823529411764705882352941176


### `math.fsum()`
+ solves subtractive cancellation and adding large and small numbers together.


{% highlight python %}
nums = [1.23e+18, 1, -1.23e+18]
sum(nums)  # Notice 1 has disappeared
{% endhighlight %}




    0.0




{% highlight python %}
import math
math.fsum(nums)
{% endhighlight %}




    1.0



## Formatting Numbers for Output

### `format()`


{% highlight python %}
x = 1234.56789
{% endhighlight %}


{% highlight python %}
# Two decimal places of accuracy
format(x, '.2f')
{% endhighlight %}




    '1234.57'




{% highlight python %}
# Right justified in 10 chars, one-digit accuracy
format(x, '>10.1f')
{% endhighlight %}




    '    1234.6'




{% highlight python %}
# Left justified
format(x, '<10.1f')
{% endhighlight %}




    '1234.6    '




{% highlight python %}
# Centered
format(x, '^10.1f')
{% endhighlight %}




    '  1234.6  '




{% highlight python %}
# Inclusion of thousands seperator
format(x, ',')
{% endhighlight %}




    '1,234.56789'




{% highlight python %}
# Together with one-digit accuracy
format(x, ',.2f')
{% endhighlight %}




    '1,234.57'



+ Use `e` / `E` to represent exponential notation.


{% highlight python %}
format(x, 'e')
{% endhighlight %}




    '1.234568e+03'




{% highlight python %}
format(x, '.3e')
{% endhighlight %}




    '1.235e+03'



+ The same format codes are also used in the `.format()` method of strings


{% highlight python %}
'The value is {:0,.2f}'.format(x)
{% endhighlight %}




    'The value is 1,234.57'



+ `str.translate()` could swap separator characters.


{% highlight python %}
swap_separator = {ord('.'): ',', ord(','): '.'}
format(x, ',').translate(swap_separator)
{% endhighlight %}




    '1.234,56789'



## Working with Binary, Octal, and Hexadecimal Integers

### `bin()` / `oct()` / `hex()`


{% highlight python %}
x = 1234
{% endhighlight %}


{% highlight python %}
bin(x)
{% endhighlight %}




    '0b10011010010'




{% highlight python %}
oct(x)
{% endhighlight %}




    '0o2322'




{% highlight python %}
hex(x)
{% endhighlight %}




    '0x4d2'



### `format()`
+ eliminates the `0b`, `0o`, `0x` prefixes.


{% highlight python %}
format(x, 'b')
{% endhighlight %}




    '10011010010'




{% highlight python %}
format(x, 'o')
{% endhighlight %}




    '2322'




{% highlight python %}
format(x, 'x')
{% endhighlight %}




    '4d2'



### `int()`
+ converts integer strings in different bases.


{% highlight python %}
int('4d2', 16)
{% endhighlight %}




    1234




{% highlight python %}
int(bin(100), 2)
{% endhighlight %}




    100



### `0o`
specifies octal values, slightly different than many othr languages.


{% highlight python %}
import os
os.chmod('script.py', 0755)
{% endhighlight %}


      File "<ipython-input-60-f4751dc2ca20>", line 2
        os.chmod('script.py', 0755)
                                 ^
    SyntaxError: invalid token




{% highlight python %}
os.chmod('scripy.py', 0o755) # √
{% endhighlight %}

### Packing and Unpacking Large Integers from Bytes

### `int.from_bytes()`
+ is to interpret the bytes as an integer, and specify the byte ordering.


{% highlight python %}
data = b'\x00\x124V\x00x\x90\xab\x00\xcd\xef\x01\x00#\x004'
int.from_bytes(data, 'little')
{% endhighlight %}




    69120565665751139577663547927094891008




{% highlight python %}
int.from_bytes(data, 'big')
{% endhighlight %}




    94522842520747284487117727783387188



### `int.to_bytes()`
+ converts a large integer value back into a byte string, and specify the number of bytes.


{% highlight python %}
x = 94522842520747284487117727783387188
x.to_bytes(16, 'big')
{% endhighlight %}




    b'\x00\x124V\x00x\x90\xab\x00\xcd\xef\x01\x00#\x004'




{% highlight python %}
x.to_bytes(16, 'little')
{% endhighlight %}




    b'4\x00#\x00\x01\xef\xcd\x00\xab\x90x\x00V4\x12\x00'



### `little` / `big`


{% highlight python %}
x = 0x01020304
x.to_bytes(4, 'big')
{% endhighlight %}




    b'\x01\x02\x03\x04'




{% highlight python %}
x.to_bytes(4, 'little')
{% endhighlight %}




    b'\x04\x03\x02\x01'



## Performing Complex-Valued Math

### `complex(real, imag)` / `j` suffix


{% highlight python %}
a = complex(2, 4)  # (2+4j)
b = 3 - 5j         # (3-5j)
{% endhighlight %}


{% highlight python %}
a.real  # 2.0
a.imag  # 4.0
a.conjugate()
{% endhighlight %}




    (2-4j)



### usual mathematical operators


{% highlight python %}
a + b  # (5-1j)
a * b  # (26+2j)
abs(a)
{% endhighlight %}




    4.47213595499958



### `cmath`
+ computes additional complex-valued functions, such as sines, cosines, or square roots.


{% highlight python %}
import cmath
cmath.sin(a)
{% endhighlight %}




    (24.83130584894638-11.356612711218174j)




{% highlight python %}
cmath.exp(a)
{% endhighlight %}




    (-4.829809383269385-5.5920560936409816j)



### Discussion
+ `numpy` can deal with complex values straightforward.
+ Standard mathematical functions like in module `math` does not produce complex values by default.

## Calculating with Fractions

### `fractions`
+ alleviates the need for a user to manually make conversions to decimals or floats.


{% highlight python %}
from fractions import Fraction
a = Fraction(5, 4)
b = Fraction(7, 16)
print(a + b)
{% endhighlight %}

    27/16



{% highlight python %}
print(a * b)
{% endhighlight %}

    35/64


+ gets numerator/denominator


{% highlight python %}
c = a * b
c.numerator
{% endhighlight %}




    35




{% highlight python %}
c.denominator
{% endhighlight %}




    64



+ converts to a float


{% highlight python %}
float(c)
{% endhighlight %}




    0.546875



+ Converting a float to a fraction.


{% highlight python %}
x = 1.25
y = Fraction(*x.as_integer_ratio())
y
{% endhighlight %}




    Fraction(5, 4)




{% highlight python %}
x.as_integer_ratio()
{% endhighlight %}




    (5, 4)



## Performing Matrix and Linear Algebra Calculations

### `np.matrix`
+ follows linear algebra rules for computation.


{% highlight python %}
m = np.matrix([[1,-2,3],[0,4,5],[7,8,-9]])
m
{% endhighlight %}




    matrix([[ 1, -2,  3],
            [ 0,  4,  5],
            [ 7,  8, -9]])




{% highlight python %}
m.T
{% endhighlight %}




    matrix([[ 1,  0,  7],
            [-2,  4,  8],
            [ 3,  5, -9]])




{% highlight python %}
v = np.matrix([[2],[3],[4]])
m * v
{% endhighlight %}




    matrix([[ 8],
            [32],
            [ 2]])



### `np.linalg`


{% highlight python %}
# Determinant
np.linalg.det(m)
{% endhighlight %}




    -229.99999999999983




{% highlight python %}
# Eigenvalues
np.linalg.eigvals(m)
{% endhighlight %}




    array([-13.11474312,   2.75956154,   6.35518158])




{% highlight python %}
# Solve for x in mx = v
x = np.linalg.solve(m, v)
x
{% endhighlight %}




    matrix([[ 0.96521739],
            [ 0.17391304],
            [ 0.46086957]])



## Picking Things at Random

### `random.choice()`


{% highlight python %}
import random
values = [1, 2, 3, 4, 5, 6]
random.choice(values)
{% endhighlight %}




    2




{% highlight python %}
random.choice(values)
{% endhighlight %}




    5



### `random.sample()`
+ the selected items are removed from further sampling.


{% highlight python %}
random.sample(values, 2)
{% endhighlight %}




    [1, 6]




{% highlight python %}
random.sample(values, 3)
{% endhighlight %}




    [3, 2, 5]



### `random.shuffle()`


{% highlight python %}
random.shuffle(values)
values
{% endhighlight %}




    [2, 3, 4, 1, 6, 5]



### `random.randint()`


{% highlight python %}
random.randint(0, 10)
{% endhighlight %}




    6



### `random.random()`
+ produces **uniform** floating-point values in the range 0 to 1.


{% highlight python %}
random.random()
{% endhighlight %}




    0.15573546310453557



## Converting Days to Seconds, and Other Basic Time Conversions

### `datetime.timedelta`
+ represents an interval of time.


{% highlight python %}
from datetime import timedelta
a = timedelta(days=2, hours=6)
b = timedelta(hours=7.5)
c = a + b
c.days
{% endhighlight %}




    2




{% highlight python %}
c.seconds
{% endhighlight %}




    48600




{% highlight python %}
c.seconds / 3600
{% endhighlight %}




    13.5




{% highlight python %}
c.total_seconds() / 3600
{% endhighlight %}




    61.5



### `datetime.datetime`
+ represents specific dates and times.


{% highlight python %}
from datetime import datetime
a = datetime(2017, 3, 5)
print(a + timedelta(days=10))
{% endhighlight %}

    2017-03-15 00:00:00



{% highlight python %}
b = datetime(2017, 6, 1)
how_close_the_TOEFL = b - a
how_close_the_TOEFL.days
{% endhighlight %}




    88



### `datetime.today()`


{% highlight python %}
now = datetime.today()
print(now)
{% endhighlight %}

    2017-03-05 18:07:02.377567



{% highlight python %}
# SCARED ！！
print(now + timedelta(days=88))
{% endhighlight %}

    2017-06-01 18:07:02.377567


### `dateutil.relativedelta()`
+ fills in some gaps pertaining to the handling of months (and their differing number of days).


{% highlight python %}
from dateutil.relativedelta import relativedelta
a = datetime(2017, 3, 5)
a + relativedelta(months=3)
{% endhighlight %}




    datetime.datetime(2017, 6, 5, 0, 0)




{% highlight python %}
b = datetime(2017, 12, 31)
time_flies = b - a
time_flies
{% endhighlight %}




    datetime.timedelta(301)




{% highlight python %}
time_flies_relative = relativedelta(b, a)
time_flies_relative
{% endhighlight %}




    relativedelta(months=+9, days=+26)




{% highlight python %}
time_flies_relative.months
{% endhighlight %}




    9




{% highlight python %}
time_flies_relative.days
{% endhighlight %}




    26



### Determining Last Saturday’s Date


{% highlight python %}
from datetime import datetime
from dateutil.relativedelta import relativedelta
from dateutil.rrule import *
{% endhighlight %}


{% highlight python %}
now = datetime.now()
print(now)
{% endhighlight %}

    2017-03-05 18:20:46.449900



{% highlight python %}
# Next Saturday
print(now + relativedelta(weekday=SA))
{% endhighlight %}

    2017-03-11 18:20:46.449900



{% highlight python %}
# Last Saturday
print(now + relativedelta(weekday=SA(-1)))
{% endhighlight %}

    2017-03-04 18:20:46.449900


## Converting Strings into Datetimes

### `datetime.strptime()`


{% highlight python %}
from datetime import datetime
text = '2017-03-05'
y = datetime.strptime(text, '%Y-%m-%d')
z = datetime.now()
diff = z - y
diff
{% endhighlight %}




    datetime.timedelta(0, 68039, 227190)



## Others

### `math.isnan()` / `math.isinf()`
+ is used to test for NaN value.

### `Numpy`


{% highlight python %}
import numpy as np
a = np.array(range(12)).reshape(3, 4)
a
{% endhighlight %}




    array([[ 0,  1,  2,  3],
           [ 4,  5,  6,  7],
           [ 8,  9, 10, 11]])




{% highlight python %}
# Broadcast a row vector across an operation on all rows
a + [100, 101, 102, 103]
{% endhighlight %}




    array([[100, 102, 104, 106],
           [104, 106, 108, 110],
           [108, 110, 112, 114]])




{% highlight python %}
# Conditional assignment on an array
np.where(a < 10, a, 10)
{% endhighlight %}




    array([[ 0,  1,  2,  3],
           [ 4,  5,  6,  7],
           [ 8,  9, 10, 10]])



### `range()`


{% highlight python %}
def date_range(start, stop, step):
    while start < stop:
        yield start
        start += step

for d in date_range(datetime(2017, 3, 5), datetime(2017, 3, 6),
                    timedelta(hours=6)):
    print(d)
{% endhighlight %}

    2017-03-05 00:00:00
    2017-03-05 06:00:00
    2017-03-05 12:00:00
    2017-03-05 18:00:00



<br><br><br>
> Take Reference from [Python Cookbook](http://shop.oreilly.com/product/0636920027072.do)<br><br>
> CHAPTER 3<br>
> Numbers, Dates, and Times
