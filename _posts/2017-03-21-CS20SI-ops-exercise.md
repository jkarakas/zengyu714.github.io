---
layout: post
title:  CS20SI Assignment 1.1 Ops Exercises
date:   2017-03-21 22:15:26 +08:00
categories: TensorFlow
tags:
- tf.gather_nd
- tf.less
- tf.cond
- tf.where
---

* content
{:toc}


## Problem 1

### 1a: Create two random 0-d tensors x and y of any distribution.

Create a TensorFlow object that returns $$x + y$$ if $$x > y$$, and $$x - y$$ otherwise.<br>
**Hint**: `tf.cond()`


{% highlight python %}
x = tf.random_uniform([])    # Empty array as shape creates a scalar.
y = tf.random_uniform([])
z = tf.constant(1, dtype=tf.int8)

res = tf.cond(tf.less(x, y), lambda: tf.add(x, y), lambda: tf.subtract(x, y))
{% endhighlight %}


{% highlight python %}
sess = tf.InteractiveSession()
tf.global_variables_initializer()

# Test it.
res.eval()
{% endhighlight %}




    0.53451741



### 1b: Create two 0-d tensors x and y randomly selected from -1 and 1.

Return $$x + y$$ if $$x < y$$, $$x - y$$ if $$x > y$$, $$0$$ otherwise.<br>
**Hint**: `tf.case()`


{% highlight python %}
x = tf.random_uniform(shape=[], minval=-1, maxval=1)
y = tf.random_uniform(shape=[], minval=-1, maxval=1)
f1 = lambda: x + y
f2 = lambda: x - y
res = tf.case({tf.less(x, y): f1, tf.greater(x, y): f2}, default=lambda: tf.constant(0.0))
{% endhighlight %}

    WARNING:tensorflow:case: Provided dictionary of predicate/fn pairs, but exclusive=False.

    Order of conditional tests is not guaranteed.



{% highlight python %}
res.eval()
{% endhighlight %}




    1.1237967



### 1c: Create the tensor x of the value [[0, -2, -1], [0, 1, 2]]

and y as a tensor of zeros with the same shape as x.<br>
Return a boolean tensor that yields Trues if x equals y element-wise.<br>
**Hint**: `tf.equal()`


{% highlight python %}
x = tf.constant([[0, -2, -1], [0, 1, 2]])
y = tf.zeros_like(x)
res = tf.equal(x, y)
{% endhighlight %}


{% highlight python %}
res.eval()
{% endhighlight %}




    array([[ True, False, False],
           [ True, False, False]], dtype=bool)



### 1d: Create the 2-d tensor x.

Get the indices of elements in x whose values are greater than 30.<br>
**Hint**: `tf.where()`<br>

Then extract elements whose values are greater than 30.<br>
**Hint**: `tf.gather_nd()` / `tf.gather()`.


{% highlight python %}
x = tf.random_normal(shape=[5, 4], mean=30, stddev=5)
indices = tf.where(tf.greater(x, 30))
res = tf.gather_nd(x, indices=indices)
{% endhighlight %}


{% highlight python %}
res.eval()
{% endhighlight %}




    array([ 33.24236679,  31.91383743,  30.83126259,  31.27710342,
            32.08654785,  30.50658798,  30.44221306,  30.56854248,
            30.91212654,  32.13285065], dtype=float32)



+ `tf.gather`, its indices must be an integer tensor of any dimension (usually `0-D` or `1-D`).
+ `tf.gather_nd` will still need to turn the column indices into a list of individual element indices <br>
  (equals with `tf.reshape`).

### 1e: Create a diagnoal 2-d tensor of size 6 x 6

with the diagonal values of 1, 2, ..., 6.<br>
**Hint**: `tf.range()` and `tf.diag()`


{% highlight python %}
x = tf.diag(tf.range(1, 7))
{% endhighlight %}


{% highlight python %}
x.eval()
{% endhighlight %}




    array([[1, 0, 0, 0, 0, 0],
           [0, 2, 0, 0, 0, 0],
           [0, 0, 3, 0, 0, 0],
           [0, 0, 0, 4, 0, 0],
           [0, 0, 0, 0, 5, 0],
           [0, 0, 0, 0, 0, 6]])



### 1f: Create a random 2-d tensor of size 10 x 10 from any distribution.

Calculate its determinant.<br>
**Hint**: `tf.matrix_determinant()`


{% highlight python %}
x = tf.random_normal(shape=[10, 10])
res = tf.matrix_determinant(x)
{% endhighlight %}


{% highlight python %}
res.eval()
{% endhighlight %}




    130.65102



### 1g: Create tensor x with value [5, 2, 3, 5, 10, 6, 2, 3, 4, 2, 1, 1, 0, 9].

Return the unique elements in x<br>
**Hint**: `tf.unique()`.


{% highlight python %}
x = [5, 2, 3, 5, 10, 6, 2, 3, 4, 2, 1, 1, 0, 9]
res, idx = tf.unique(x)    # Returns a tuple.
{% endhighlight %}


{% highlight python %}
res.eval()
{% endhighlight %}




    array([ 5,  2,  3, 10,  6,  4,  1,  0,  9])



### 1h: Create two tensors x and y of shape 300 from any normal distribution,

as long as they are from the same distribution.<br>
Return **Huber loss**
$$L_\delta(y, f(x)) =
\begin{cases}
\frac{1}{2}(y-f(x))^2 & ,\ {| y - f(x)|  \le \delta} \\
\delta \ | y - f(x)|  - \frac{1}{2} {\delta}^2 & ,\ otherwise
\end{cases}$$

**Hint**: `tf.less()` and `tf.where()`


{% highlight python %}
x = tf.random_normal([300])
y = tf.random_normal([300])
residual = x - y
condition = tf.less(residual, 0)    # Here delta == 0, similarily.
small_residual_func = 0.5 * tf.square(residual)
large_residual_func = tf.abs(residual)
res = tf.where(condition, small_residual_func, large_residual_func)
{% endhighlight %}

+ `tf.where(condition, x=None, y=None, name=None)` returns the elements, either from `x` or `y`, <br>
    depending on the `condition`.<br>
    If both `x` and `y` are `None`, then this operation returns the coordinates of true elements of `condition`.
