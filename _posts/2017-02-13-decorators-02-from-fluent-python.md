---
author: kimmy
layout: post
title:  "Decorator Part2"
date:   2017-02-13 23:48:06 +08:00
categories: Python
tags:
- Python
- Notes
- Hamburger
- Clock Decorator
---


* content
{:toc}



## Decorators' power
### Replace the decorated function with a different one


```python
def ordinary_function():
    print("I'm an ordinary function.")

ordinary_function()
```

    I'm an ordinary function.



```python
def my_decorator(func):
    def wrapper():
        print("I'm the decorated one.")
    return wrapper
```


```python
@my_decorator
def ordinary_function():
    print("I'm an ordinary function.")

ordinary_function()
```

    I'm the decorated one.


### Run right after the decorated function is defined

As known as import time

## Case study, see previous [implementation](https://zengyu714.github.io/python/2017/02/12/little-about-design-patterns/)


```python
promos = []
def promotion(promo_func):
    promos.append(promo_func)
    return promo_func

@promotion
def fidelity(order):
    """5% discount for customers with 1000 or more fidelity points"""
    return order.total() * .05 if order.customer.fidelity >= 1000 else 0

@promotion
def bulk_item(order):
    """10% discount for each LineItem with 20 or more units"""
    discount = 0
    for item in order.cart:
        if item.quantity >= 20:
            discount += item.total * .1
    return discount

@promotion
def large_order(order):
    """7% discount for orders with 10 or more distinct items"""
    distinct_items = {items.product for item in order.cart}
    if len(distinct_item) >= 10:
        return order.total * .07
    return 0

def best_promo(order):
    """Select best discount available
    """
    return max(promo(order) for promo in promos)
```

### KEYNOTE
---
1. `promotion` decorator returns `promo_func` **unchanged**, after adding it to the `promos` list.
2. Any function decorated by `@promotion` will be added to `promos`.
3. Comparing to

    ```
    promos = [globals()[name] for name in globals()
                  if name.endswith('_promo')
                  and name != 'best_promo']
    ```

    ```
    promos = [func for name, func in
                inspect.getmembers(promotions, inspect.isfunction)]
    ```
  `@promotion`
  + doesn't need to use special names, e.g. `_promo` suffix.
  + is flexible, comment out the decorator if want to disable a promotion.

## Try a simple decorator

clocks every invocation of the decorated function and
prints the elapsed time, the arguments passed, and the result of the call.


```python
import time
def clock(func):
    # accept any number of positional arguments
    def clocked(*args):
        t0 = time.perf_counter()
        result = func(*args)
        elapsed = time.perf_counter() - t0
        name = func.__name__
        arg_str = ', '.join(repr(arg) for arg in args)
        print('[%0.8fs] %s(%s) -> %r ' % (elapsed, name, arg_str, result))
        return result

    # return the inner function to replace the decorated function
    return clocked
```


```python
@clock
def snooze(seconds):
    time.sleep(seconds)

@clock
def factorial(n):
    return 1 if n < 2 else n * factorial(n-1)
```


```python
print('*' * 40, 'Calling snooze(.123)')
snooze(.123)
print('*' * 40, 'Calling factorial(6)')
print('6! =', factorial(6))
```

    **************************************** Calling snooze(.123)
    [0.12311179s] snooze(0.123) -> None
    **************************************** Calling factorial(6)
    [0.00000164s] factorial(1) -> 1
    [0.00009976s] factorial(2) -> 2
    [0.00019090s] factorial(3) -> 6
    [0.00051318s] factorial(4) -> 24
    [0.00059078s] factorial(5) -> 120
    [0.00067083s] factorial(6) -> 720
    6! = 720


### How?


```python
factorial.__name__
```




    'clocked'



`factorial` now actually holds a reference to the `clocked` function.
`factorial(n)` <==> `clocked(n)`

## Advanced clock decorator


```python
import time
import functools

def clock(func):
    @functools.wraps(func)
    def clocked(*args, **kwargs):
        t0 = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - t0
        name = func.__name__
        arg_lst = []
        if args:
            arg_lst.append(', '.join(repr(arg) for arg in args))
        if kwargs:
            pairs = ['%s=%r' % (k, w) for k, w in sorted(kwargs.items())]
            arg_lst.append(', '.join(pairs))
        arg_str = ', '.join(arg_lst)
        print('[%0.8fs] %s(%s) -> %r ' % (elapsed, name, arg_str, result))
        return result
    return clocked
```

## `functools.lru_cache`
named after Least Recently Used, with two optional arguments
`functools.lru_cache(maxsize=128, typed=False)`

### Avoid wasty recursion


```python
@clock
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

fibonacci(6)
```

    [0.00000000s] fibonacci(1) -> 1
    [0.00000000s] fibonacci(0) -> 0
    [0.00100088s] fibonacci(2) -> 1
    [0.00000000s] fibonacci(1) -> 1
    [0.00100088s] fibonacci(3) -> 2
    [0.00000000s] fibonacci(1) -> 1
    [0.00000000s] fibonacci(0) -> 0
    [0.00000000s] fibonacci(2) -> 1
    [0.00100088s] fibonacci(4) -> 3
    [0.00000000s] fibonacci(1) -> 1
    [0.00000000s] fibonacci(0) -> 0
    [0.00000000s] fibonacci(2) -> 1
    [0.00000000s] fibonacci(1) -> 1
    [0.00000000s] fibonacci(3) -> 2
    [0.00100088s] fibonacci(5) -> 5
    [0.00000000s] fibonacci(1) -> 1
    [0.00000000s] fibonacci(0) -> 0
    [0.00000000s] fibonacci(2) -> 1
    [0.00000000s] fibonacci(1) -> 1
    [0.00000000s] fibonacci(3) -> 2
    [0.00000000s] fibonacci(1) -> 1
    [0.00000000s] fibonacci(0) -> 0
    [0.00000000s] fibonacci(2) -> 1
    [0.00000000s] fibonacci(4) -> 3
    [0.00100088s] fibonacci(6) -> 8





    8




```python
import functools

@functools.lru_cache()
@clock
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

fibonacci(6)
```

    [0.00000000s] fibonacci(1) -> 1
    [0.00000000s] fibonacci(0) -> 0
    [0.00000000s] fibonacci(2) -> 1
    [0.00000000s] fibonacci(3) -> 2
    [0.00000000s] fibonacci(4) -> 3
    [0.00000000s] fibonacci(5) -> 5
    [0.00000000s] fibonacci(6) -> 8





    8



### Lastly

`@singledispatch`: better than `if/elif/elif/elif` blocks


## Parameterized decorators


```python
bucket = set()
# Now adding or removing is faster than list

def add_into_bucket(sure = True):
    # takes an optional keyword argument

    def decorator(a_special_item):
        # the actual decorator
        print("processing item(sure = %s)->wrapper(%s)"
             % (sure, a_special_item))
        if sure:
            bucket.add(a_special_item)
        else:
            bucket.discard(a_special_item)

        return a_special_item
        # decorator must return a function

    return decorator
    # then applied to the decorated function


@add_into_bucket()
# if no parameters are passed, add_into_bucket() must still be called as a function
def lovely_item():
    print("★★★★lovely item★★★★ ")

@add_into_bucket(sure = False)
# be invoked as a function, with the desired parameters
def horrible_item():
    print("×××  horrible item  ×××")

def common_item():
    print("———— common item ————")
```

    processing item(sure = True)->wrapper(<function lovely_item at 0x00000251119DA7B8>)
    processing item(sure = False)->wrapper(<function horrible_item at 0x0000025111AB3950>)



```python
bucket
```




    {<function __main__.lovely_item>}




```python
add_into_bucket()(common_item)
```

    processing item(sure = True)->wrapper(<function common_item at 0x0000025111AB3A60>)





    <function __main__.common_item>




```python
bucket
```




    {<function __main__.lovely_item>, <function __main__.common_item>}




```python
add_into_bucket(sure = False)(lovely_item)
```

    processing item(sure = False)->wrapper(<function lovely_item at 0x00000251119DA7B8>)





    <function __main__.lovely_item>




```python
bucket
```




    {<function __main__.common_item>}



## Parameterized clock decorator


```python
import time

DEFAULT_FMT = '[{elapsed:0.8f}s] {name}({args}) -> {result}'
def clock(fmt=DEFAULT_FMT):
    # clock is parameterized decorator factory

    def decorate(func):
        # the actual decorator

        def clocked(*_args):
            # wraps the decorated function
            t0 = time.time()
            _result = func(*_args)       # _result is the actual result of the decorated function
            elapsed = time.time() - t0
            name = func.__name__
            args = ', '.join(repr(arg) for arg in _args)
            result = repr(_result)       # the str representation of _result, for display
            print(fmt.format(**locals()))
            return _result
            # clocked will replace the decorated function,
            # so it should return whatever that function returns
        return clocked
    return decorate
```


```python
@clock()
def snooze(seconds):
    time.sleep(seconds)

# default format
for i in range(3):
    snooze(.123)
```

    [0.12328577s] snooze(0.123) -> None
    [0.12370849s] snooze(0.123) -> None
    [0.12361050s] snooze(0.123) -> None



```python
@clock('{name}: {elapsed}s')
def snooze(seconds):
    time.sleep(seconds)

for i in range(3):
    snooze(.123)
```

    snooze: 0.12315082550048828s
    snooze: 0.12341547012329102s
    snooze: 0.12379145622253418s



```python
@clock('{name}({args}) dt={elapsed:0.3f}s')
def snooze(seconds):
    time.sleep(seconds)

for i in range(3):
    snooze(.123)
```

    snooze(0.123) dt=0.123s
    snooze(0.123) dt=0.124s
    snooze(0.123) dt=0.123s
