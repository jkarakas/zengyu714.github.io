---
author: kimmy
layout: post
title:  "Little about Design Patterns"
date:   2017-02-12 19:48:17 +08:00
categories: Python
tags:
- Python
- Notes
- Hello
---


* content
{:toc}



## Case Study

### Function-Oriented Strategy


{% highlight python %}
from collections import namedtuple

Customer = namedtuple('Customer','name fidelity')

class LineItem:

    def __init__(self, product, quantity, price):
        self.product = product
        self.quantity = quantity
        self.price = price

    def total(self):
        return self.price * self.quantity
{% endhighlight %}


{% highlight python %}
class Order:
    # the Context

    def __init__(self, customer, cart, promotion = None):
        self.customer = customer
        self.cart = list(cart)
        self.promotion = promotion

    def total(self):
        if not hasattr(self, '__total'):
            self.__total = sum(item.total() for item in self.cart)
        return self.__total

    def due(self):
        if self.promotion == None:
            discount = 0
        else:
            discount = self.promotion(self)
        return self.total() - discount

    def __repr__(self):
        fmt = '<Order total: {:.2f} due: {:.2f}>'
        return fmt.format(self.total(), self.due())

{% endhighlight %}


{% highlight python %}
def fidelity_promo(order):
    """5% discount for customers with 1000 or more fidelity points"""
    return order.total()* .05 if order.customer.fidelity >= 1000 else 0

def bulk_item_promo(order):
    """10% discount for each LineItem with 20 or more units"""
    discount = 0
    for item in order.cart:
        if item.quantity >= 20:
            discount += item.total() * .1
    return discount

def large_order_promo(order):
    """7% discount for orders with 10 or more distinct items"""
    discount = 0
    distinct_items = {item.product for item in order.cart}
    if len(distinct_items) >= 10:
        return order.total()* .07
    return 0
{% endhighlight %}


{% highlight python %}
joe = Customer('John Doe', 0)
ann = Customer('Ann Smith', 1100)
cart = [
    LineItem('banana', 4, .5),
    LineItem('apple', 10, 1.5),
    LineItem('watermellon', 5, 5.0)
]
{% endhighlight %}


{% highlight python %}
Order(joe, cart, fidelity_promo)
{% endhighlight %}




    <Order total: 42.00 due: 42.00>




{% highlight python %}
Order(ann, cart, fidelity_promo)
{% endhighlight %}




    <Order total: 42.00 due: 39.90>




{% highlight python %}
banana_cart = [LineItem('banana', 30, .5),
               LineItem('apple', 10, 1.5)]
Order(joe, banana_cart, bulk_item_promo)
{% endhighlight %}




    <Order total: 30.00 due: 28.50>




{% highlight python %}
long_order = [LineItem(str(item_code), 1, 1.0)
                    for item_code in range(10)]
Order(joe, long_order, large_order_promo)
{% endhighlight %}




    <Order total: 10.00 due: 9.30>



### Simple Approach

best_promo finds the maximum discount iterating over a list of functions


{% highlight python %}
# list of the strategies implemented as functions
promos = [fidelity_promo, bulk_item_promo, large_order_promo]

def best_promo(order):
    """Select best discount available
    """
    return max(promo(order) for promo in promos)
{% endhighlight %}


{% highlight python %}
Order(joe, long_order, best_promo)
{% endhighlight %}




    <Order total: 10.00 due: 9.30>




{% highlight python %}
Order(joe, banana_cart, best_promo)
{% endhighlight %}




    <Order total: 30.00 due: 28.50>




{% highlight python %}
Order(ann, cart, best_promo)
{% endhighlight %}




    <Order total: 42.00 due: 39.90>



### `globals()` to help find other available `*_promo` functions automatically


{% highlight python %}
promos = [globals()[name] for name in globals()
                        if name.endswith('_promo')
                        and name != 'best_promo']
{% endhighlight %}

No changes inside `best_promo`.


{% highlight python %}
def best_promo(order):
    """Select best discount available
    """
    return max(promo(order) for promo in promos)
{% endhighlight %}

### Introspection of a separate module `promotion`


{% highlight python %}
promos = [func for name, func in
            inspect.getmembers(promotions, inspect.isfunction)]
{% endhighlight %}

## Command

Each instance of MacroCommand has an internal list of commands


{% highlight python %}
class MacroCommand:
    """A command that executes a list of commands"""
    def __init__(self, commands):
        self.commands = list(commands) # 1
    def __call__(self):
        for command in self.commands:  # 2
            command()
{% endhighlight %}

### KEYNOTE
---
1. Building a list from the commands arguments ensures that it is **iterable** and keeps
    a local copy of the command references in each MacroCommand instance.
2. When an instance of MacroCommand is invoked, each command in `self.commands` is called in sequence.
