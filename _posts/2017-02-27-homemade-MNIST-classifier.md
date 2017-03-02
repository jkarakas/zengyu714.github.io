---
author: kimmy
layout: post
title:  "Homemade MNIST Classifier"
date:   2017-02-27 16:36:55 +08:00
categories: DeepLearning
tags:
- DeepLearning
- Notes
- Backpropagation

---


* content
{:toc}




## Split MNIST Data
+ 50000 training
+ 10000 validation
+ 10000 test

## Load Data
see full [source](https://github.com/zengyu714/neural-networks-and-deep-learning-py3/blob/master/mnist_loader.py)


{% highlight python %}
import mnist_loader
training_data, validation_data, test_data = mnist_loader.load_data()
{% endhighlight %}

Note the `one_hot` function, if `i` is the true label, then `i == true_label` would be `True`, thus equals 1.


{% highlight python %}
def one_hot(true_label):
    return np.array([int(i == true_label) for i in range(10)])
{% endhighlight %}

## Construct Network
see full [source](https://github.com/zengyu714/neural-networks-and-deep-learning-py3/blob/master/network.py)


{% highlight python %}
class Network(object):

    def __init__(self, shape):                                                        # 1
        self.num_layers = len(shape)
        self.shape = shape
        self.biases = [np.random.randn(y, 1) for y in shape[1:]]                      # 2
        self.weights = [np.random.randn(y, x) for x, y in zip(shape[:-1], shape[1:])] # 3

{% endhighlight %}

1. Initialization, for example, `net = Network((784, 30, 10))`.
2. Input layer has no bias, therefore `self.biases` starts from the second layer.
3. $$w_{jk}$$ is the weight for the connection between the $$k^{th}$$ layer and $$j^{th}$$ layer, in a perspective of the **later** layer $$j^{th}$$.<br>
    So as the reason for the implement of `np.random.randn(y, x)`.

## KEYNOTE
---

### 1. `zip` function


{% highlight python %}
x = [1, 2, 3]
y = [4, 5, 6]
zipped = zip(x, y)
list(zipped)
{% endhighlight %}


{% highlight python %}
list(zipped)
{% endhighlight %}

`zip()` is a generator, it produces the values just **once**.


{% highlight python %}
zipped_to_save = list(zip(x, y))
zipped_list = zipped_to_save[:]
zipped_list
{% endhighlight %}

If we want to reuse the zipped object, we could save it in a `list`.

### 2. Reload module when debug

However, the instances instantiated before will not change.<br>

#### Module `importlib`


{% highlight python %}
import foo
# ...
# something changed in func
# ...
import importlib
importlib.reload(foo)
{% endhighlight %}

#### IPython notebook


{% highlight python %}
# Reload all modules (except those excluded by %aimport)
# automatically now.
%load_ext autoreload

# Reload all modules (except those excluded by %aimport)
# every time before executing the Python code typed.
%autoreload 2
{% endhighlight %}


{% highlight python %}
from foo import some_function
some_function()
{% endhighlight %}




    42




{% highlight python %}
# open foo.py in an editor and change some_function to return 43
some_function()
{% endhighlight %}




    47



### 3. Expand the shape of an `array`


{% highlight python %}
array_1_dim = np.array([1, 2, 3, 4, 5])
array_2_dim = np.array([[1, 2, 3, 4, 5]])
print("array_1_dim: ", array_1_dim.shape)
print("array_2_dim: ", array_2_dim.shape)
{% endhighlight %}

    array_1_dim:  (5,)
    array_2_dim:  (1, 5)


`array_1_dim` is definitely one dimensional, because the square bracket is only one.<br>

#### Solution 1
`numpy.expand_dims(a, axis)`


{% highlight python %}
import numpy as np
temp = np.expand_dims(array_1_dim, axis = 1)
temp.shape
{% endhighlight %}




    (5, 1)



#### Solution 2
`array.shape`


{% highlight python %}
# notice this way expand dimension in-place
# be CAREFUL to use it continuously
array_1_dim.shape += (1, )  # '+' means tuple concatenate
array_1_dim.shape
{% endhighlight %}




    (5, 1)



#### Solution 3
`array.reshape`


{% highlight python %}
temp = np.reshape(array_1_dim,(5, 1))
temp.shape
{% endhighlight %}




    (5, 1)



#### Solution 4
`num_elements = np.prod(array.shape)`


{% highlight python %}
array_1_dim.shape = (np.prod(array_1_dim.shape), 1)
array_1_dim.shape
{% endhighlight %}




    (5, 1)
