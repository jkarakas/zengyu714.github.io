---
author: kimmy
layout: post
title:  "Getting Started With TensorFlow"
date:   2017-02-18 22:26:21 +08:00
categories: TensorFlow
tags:
- Python
- Notes
- TensorFlow
- CNN
---

* content
{:toc}



## [Fundamental](https://www.tensorflow.org/get_started/get_started)

1. **TensorFlow Core** is the lowest level API, while provides most flexible control.
2. A few of high-level TensorFlow APIs, whose method names contain `contrib`, are still in development and unstable.
3. `tensor` consists of a set of primitive values shaped into an array of any number of dimensions.
   Tensor's **rank** is its number of dimensions.


##  **Computational Graph**
- is a series of TensorFlow operations arranged into a graph of nodes.
+ can be parameterized to accept external inputs, AKA **placeholders**, which is a promise to provide a value
later.
+ accepts **variables** to add trainable parameters, and they are constructed with a type and initial value.

**Node**: BUILD
   - takes zero or more tensors as inputs and produces a tensor as an output.
   - also includes **operations**.

**Session**: RUN
   - connects to the highly efficient C++ backend for doing intensive computation.
   - is usually launched after creating a graph.


{% highlight python %}
import tensorflow as tf
{% endhighlight %}

### Constant Node


{% highlight python %}
node1 = tf.constant(3.0, tf.float32)
node2 = tf.constant(4.0)             # tf.float32 implicitly
print(node1, node2)
{% endhighlight %}

    Tensor("Const:0", shape=(), dtype=float32) Tensor("Const_1:0", shape=(), dtype=float32)


### Run a Session


{% highlight python %}
sess = tf.Session()
print(sess.run([node1, node2]))
{% endhighlight %}

    [3.0, 4.0]


### Combine with Operations


{% highlight python %}
node3 = tf.add(node1, node2)         # add two constant nodes and produce a new graph
print("node3: ", node3)
print("sess.run(node3): ", sess.run(node3))
{% endhighlight %}

    node3:  Tensor("Add_1:0", shape=(), dtype=float32)
    sess.run(node3):  7.0


### Placeholder


{% highlight python %}
a = tf.placeholder(tf.float32)
b = tf.placeholder(tf.float32)
adder_node = a + b                  # + provides a shortcut for tf.add
{% endhighlight %}


{% highlight python %}
print(sess.run(adder_node, {a: 3, b: 4.5}))
print(sess.run(adder_node, {a: [1, 3], b: [2, 4]}))
{% endhighlight %}

    7.5
    [ 3.  7.]



{% highlight python %}
add_and_triple = adder_node * 3
print(sess.run(add_and_triple, {a: 3, b: 4.5}))
{% endhighlight %}

    22.5


### Variables

+ The value of `tf.constant` can never change, and constants are initialized previously.
+ `tf.Variable` are not initialized, but need to call explicitly

{% highlight python %}
init = tf.global_variables_initializer()
sess.run(init)

{% endhighlight %}


{% highlight python %}
W = tf.Variable([.3], tf.float32)
b = tf.Variable([-.3], tf.float32)
x = tf.placeholder(tf.float32)
linear_mode = W * x + b
{% endhighlight %}


{% highlight python %}
init = tf.global_variables_initializer()
sess.run(init)
{% endhighlight %}


{% highlight python %}
print(sess.run(linear_mode, {x:[1,2,3,4]}))
{% endhighlight %}

    [ 0.          0.30000001  0.60000002  0.90000004]


### Caculate Loss
A standard loss model for linear regression
+ `linear_model - y` creates a vector where each element is the corresponding example's error delta
+ `tf.square` to square that error
+ `tf.reduce_sum` to sum all the squared errors


{% highlight python %}
y = tf.placeholder(tf.float32)
squared_delta = tf.square(linear_mode - y)
loss = tf.reduce_sum(squared_delta)
print(sess.run(loss, {x:[1,2,3,4], y:[0,-1,-2,-3]}))
{% endhighlight %}

    23.66


### Change `W` and `b` Manually


{% highlight python %}
fixW = tf.assign(W, [-1.])
fixb = tf.assign(b, [1.])
sess.run([fixW, fixb])
print(sess.run(loss,{x:[1,2,3,4], y:[0,-1,-2,-3]}))
{% endhighlight %}

    0.0


### `tf.train` API

`optimizer`: slowly changes each variable in order to minimize the loss function.
`tf.gradients`: can automatically produce derivatives according to the description of a model.


{% highlight python %}
optimizer = tf.train.GradientDescentOptimizer(0.01)
train = optimizer.minimize(loss)

sess.run(init)  # reset values to incorrect defaults
for i in range(1000):
    sess.run(train, {x:[1,2,3,4], y:[0,-1,-2,-3]})
print(sess.run([W,b]))
{% endhighlight %}

    [array([-0.9999969], dtype=float32), array([ 0.99999082], dtype=float32)]


## Complete Codes


{% highlight python %}
# Model parameters
W = tf.Variable([.3], tf.float32)
b = tf.Variable([-.3], tf.float32)
# Model input and output
x = tf.placeholder(tf.float32)
linear_model = W * x + b
y = tf.placeholder(tf.float32)
# loss: sum of the squares
loss = tf.reduce_sum(tf.square(linear_model - y))
# optimizer
optimizer = tf.train.GradientDescentOptimizer(0.01)
train = optimizer.minimize(loss)
# training data
x_train = [1,2,3,4]
y_train = [0,-1,-2,-3]
# training loop
init = tf.global_variables_initializer()
sess.run(init)  # reset values to wrong
for i in range(1000):
    sess.run(train, {x:x_train, y:y_train})
#evaluate training accuracy
curr_W, curr_b, curr_loss = sess.run([W, b, loss], {x:x_train, y: y_train})
print("W: %s b: %s loss: %s"%(curr_W, curr_b, curr_loss))
{% endhighlight %}

    W: [-0.9999969] b: [ 0.99999082] loss: 5.69997e-11


## `tf.contrib.learn`
+ a high-level TensorFlow library that simplifies the mechanics of machine learning.
+ defines many common models.


### Declare List of Features
There are many other types of columns that are more complicated and useful.


{% highlight python %}
features = [tf.contrib.layers.real_valued_column("",dimension=1)]
{% endhighlight %}

### Predefined Types
An `estimator` is the front end to invoke training(fitting) and evaluation(inference).
Other neural network classifiers and regressors are like linear regression, logistic regression, linear classification, logistic classification.


{% highlight python %}
estimator = tf.contrib.learn.LinearRegressor(feature_columns=features)
{% endhighlight %}

### Read and Set Up Data Sets

The various readers provided by `tf.learn` are better at batching and
feeding the data in a computationally efficient way compared to this simple approach like using `numpy`.


{% highlight python %}
import numpy as np
dataSet = tf.contrib.learn.datasets.base.Dataset(
data = np.array([[1],[2],[3],[4]]),
target = np.array([[0],[-1],[-2],[-3]]))
{% endhighlight %}

### `fit`
We can invoke 1000 training steps by invoking the `fit` method and passing the training data set.


{% highlight python %}
estimator.fit(x=dataSet.data, y=dataSet.target, steps=1000)
{% endhighlight %}

### `evaluate`
how well our model did.
In a real example, we would want to use a separate **validation** and **testing** data set to avoid overfitting.


{% highlight python %}
estimator.evaluate(x=dataSet.data, y=dataSet.target)
{% endhighlight %}

## A Custom Model

To define a custom model that works with `tf.contrib.learn`, we provide `Estimator` a **function model** that tells `tf.contrib.learn` how a model should evaluate predictions, training steps, and loss.


{% highlight python %}
import numpy as np
import tensorflow as tf
# Declare list of features, here only have one real-values feature
features = [tf.contrib.layers.real_valued_column("", dimension=1)]

def model(features, labels, mode, params):
    with(tf.device("/cpu:0")):
        # Build a linear model and predict values
        W = tf.get_variable("W", [1], dtype=tf.float64)
        b = tf.get_variable("b", [1], dtype=tf.float64)
        y = W*features[:,0] + b
        # Loss sub-graph
        loss = tf.reduce_sum(tf.square(y - labels))
        # Training sub-graph
        global_step = tf.train.get_global_step()
        optimizer = tf.train.GradientDescentOptimizer(0.01)
        train = tf.group(optimizer.minimize(loss),
                        tf.assign_add(global_step, 1))
        # ModelFnOps connects subgraphs we built to the
        # approprite functionality
        return tf.contrib.learn.ModelFnOps(
        mode=mode, predictions=y,
        loss=loss,
        train_op=train)

estimator = tf.contrib.learn.Estimator(model_fn=model)
# Define the data set
dataSet = tf.contrib.learn.datasets.base.Dataset(
data=np.array([[1.],[2.],[3.],[4.]]),
target=np.array([[0.],[-1.],[-2.],[-3.]]))
# Train
estimator.fit(x=dataSet.data, y=dataSet.target, steps=1000)
# Evaluate
print(estimator.evaluate(x=dataSet.data, y=dataSet.target, steps=10))

{% endhighlight %}
