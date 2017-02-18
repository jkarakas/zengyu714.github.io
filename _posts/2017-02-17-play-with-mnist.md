---
author: kimmy
layout: post
title:  "Play With MNIST and Softmax"
date:   2017-02-17 23:34:21 +08:00
categories: TensorFlow
tags:
- Python
- Notes
- TensorFlow
- MNIST
---


* content
{:toc}



<br>
<br>
MNIST is a tutorial level for further moving on.

And one day, we will find the way to build an elaborate world in Deep Learning.
<br>
<br>


## REVIEW
---
### Data Split
The MNIST data is split into three parts
+ `mnist.train`
+ `mnist.test`
+ `mnist.validation`

### x
+ Every image is `28 x 28`, we can flatten the 2-d array into `728 x 1`.
  Make sure to be consistent between images.
+ `mnist.train.images` is a **tensor**(an n-dimensional array) with a shape of `[55000, 784]`.
  Each entry in the tensor is a pixel **regularized** between 0 and 1.

### y
+ Corresponding label is a number between 0 and 9.
  Adopting `one-hot` encoding, e.g., 2 would be [0,0,1,0,0,0,0,0,0,0].
+ Consequently, `mnist.train.labels` is a `[55000, 10]` array of floats.


{% highlight python %}
# windows is based on python3.5, where __future__ module is mandatory
# from __future__ import absolute_import
# from __future__ import division
# from __future__ import print_function


import argparse
import sys

from tensorflow.examples.tutorials.mnist import input_data

import tensorflow as tf

FLAGS = None

def main(_):
    # Import data
    mnist = input_data.read_data_sets(FLAGS.data_dir, one_hot=True)

    # Create the model
    x = tf.placeholder(tf.float32, [None, 784])  # 1
    W = tf.Variable(tf.zeros([784, 10]))         # 2
    b = tf.Variable(tf.zeros([10]))
    y = tf.matmul(x, W) + b                      # 3

    # Define loss and optimizer
    y_ = tf.placeholder(tf.float32, [None, 10])                                   # 1

    # use tf.nn.softmax_cross_entropy_with_logits on the
    # raw outputs of 'y', and then average across the batch
    cross_entropy = tf.reduce_mean(
    tf.nn.softmax_cross_entropy_with_logits(labels=y_, logits=y))
    train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)   # 2

    sess = tf.InteractiveSession()                                                # 3
    tf.global_variables_initializer().run()                                       # 4

    # Train
    for _ in range(1000):                                                         # 5
        batch_xs, batch_ys = mnist.train.next_batch(100)
        sess.run(train_step, feed_dict={x: batch_xs, y: batch_ys})

    # Test trained model
    correct_prediction = tf.equal(tf.argmax(y, 1), tf.argmax(y_, 1))    # 1
    accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))  # 2
    print(sess.run(accuracy, feed_dict={x:mnist.test.images,            # 3
                                       y_:mnist.test.labels}))

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--data_dir", type=str,
                        default='./input_data',
                        help="Directory for storing input data")
    FLAGS, unparsed = parser.parse_known_args()
    tf.app.run(main=main, argv=[sys.argv[0]] + unparsed)
{% endhighlight %}

## Regression
---
1.  `x = tf.placeholder(tf.float32, [None, 784])`
   `None` means a dimension can be any length, that is the input can be any number of MNIST images.
2.  `W = tf.Variable(tf.zeros([784, 10]))`
   `b = tf.Variable(tf.zeros([10]))`
   Here initialize both `W` and `b` as tensors full of zeros.
   `Variable` is a modifiable tensor, or is the representation of network parameters.
3.  `y = tf.nn.softmax(tf.matmul(x, W) + b)`
   Notice mutiply `x` by `W`, think about matrix multiplication rules.


## Training
---
1.  $-\sum y' log(y)\quad$
   could be implemented by `-tf.reduce_sum(y_ * tf.log(y), reduction_indices=[1])`
   `tf.reduce_sum` adds the elements in the **second** dimension of y, due to `reduction_indices=[1]`
   However, `tf.nn.softmax_cross_entropy_with_logits` are more numerically stable function.

2.  `train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)`
   using the gradient descent algorithm with a learning rate of 0.5.
   See more [optimization algorithms](https://www.tensorflow.org/api_guides/python/train#optimizers).

3.  `sess = tf.InteractiveSession()` launches the model in an `InteractiveSession`.

4.  `tf.global_variables_initializer().run()` creates an operation to initialize the variables.

5.  run the training step 1000 times

    {% highlight python %}
    for _ in range(1000):
        batch_xs, batch_ys = mnist.train.next_batch(100)
        sess.run(train_step, feed_dict={x: batch_xs, y_: batch_ys})
    {% endhighlight %}
    + `mnist.train.next_batch(100)` get a **batch** of one hundred *random* data points.
    + In each `train_step`, we use batches to replace the `placeholders`


## Evaluating
---
1.  `tf.argmax` gives the **index** of the highest entry in a tensor along a specific axis.
   `tf.equal` checkes whether the prediction matches the truth, outputs a list of **booleans**.
2.  `tf.cast` to floating point numbers, e.g.,
    [True, True, True, False] $\to$ [1, 1, 1, 0] $\to$ 0.75
3.  `sess.run(accuracy, feed_dict={x: mnist.test.images, y_: mnist.test.labels})`
   caculates the accuracy on test data.
