---
author: kimmy
layout: post
title:  "Play with MNIST and CNN"
date:   2017-02-18 22:22:24 +08:00
categories: TensorFlow
tags:
- Python
- Notes
- TensorFlow
- CNN
---

* content
{:toc}


## Setup

### Load MNIST Data


{% highlight python %}
from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets('MNIST_data', one_hot=True)
{% endhighlight %}

    Extracting MNIST_data\train-images-idx3-ubyte.gz
    Extracting MNIST_data\train-labels-idx1-ubyte.gz
    Extracting MNIST_data\t10k-images-idx3-ubyte.gz
    Extracting MNIST_data\t10k-labels-idx1-ubyte.gz


### Start TensorFlow InteravtiveSession
+ `InteractiveSession` allows to interleave operations build a computation graph and ones run the graph.
+ If not use `InteractiveSession`, then you should build the entire computation graph before starting a session and launching the graph.


{% highlight python %}
import tensorflow as tf
sess = tf.InteractiveSession()
{% endhighlight %}

## Softmax Regression Model


{% highlight python %}
x = tf.placeholder(tf.float32, shape=[None, 784])
y_ = tf.placeholder(tf.float32, shape=[None, 10])
{% endhighlight %}

+ `None` indicates the first dimension, corresponding to the batch size, can be of **any** size.
+ `shape` argument is optional, but could check bugs from inconsistent tensor shapes


{% highlight python %}
W = tf.Variable(tf.zeros([784, 10]))
b = tf.Variable(tf.zeros([10]))
{% endhighlight %}


{% highlight python %}
sess.run(tf.global_variables_initializer())
{% endhighlight %}

+ `Variable` must be initialized through `sess.run()` before they can be used within that session.
+ This step assigns the initial values specified before (here tensors full of zeros) to each `Variable`.


{% highlight python %}
y = tf.matmul(x, W) + b
cross_entropy = tf.reduce_mean(
    tf.nn.softmax_cross_entropy_with_logits(labels=y_, logits=y))
{% endhighlight %}


{% highlight python %}
train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)
{% endhighlight %}


{% highlight python %}
for _ in range(1000):
    batch = mnist.train.next_batch(100)
    train_step.run(feed_dict={x: batch[0], y_:batch[1]})
{% endhighlight %}


{% highlight python %}
correct_prediction = tf.equal(tf.argmax(y, 1), tf.argmax(y_, 1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
print(accuracy.eval(feed_dict={x: mnist.test.images, y_:mnist.test.labels}))
{% endhighlight %}

    0.9178


## CNN Model

### Weight Initialization


{% highlight python %}
def weight_variable(shape):
    initial = tf.truncated_normal(shape, stddev=0.1)
    return tf.Variable(initial)

def bias_variable(shape):
    initial = tf.constant(0.1, shape=shape)
    return tf.Variable(initial)
{% endhighlight %}

+ Initialize weights with a small amount of noise for **symmetry breaking**, and to **prevent 0 gradients**.
+ ReLU neurons suggest a **slightly positive** initial bias to avoid dead neurons.
+ Create two handy functions in case do it repeatedly.

### Convolution and Pooling


{% highlight python %}
def conv2d(x, W):
    return tf.nn.conv2d(x, W, strides=[1, 1, 1, 1], padding='SAME')

def max_pool_2x2(x):
    return tf.nn.max_pool(x, ksize=[1, 2, 2, 1],
                         strides=[1, 2, 2, 1], padding='SAME')
{% endhighlight %}

+ `strides`: A list of ints. 1-D of length 4. The stride of the sliding window for **each dimension** of input, say, "NHWC".
+ `ksize`:  A list of ints that has length >= 4. The size of the window for each dimension of the input tensor.

### First Convolutional Layer


{% highlight python %}
W_conv1 = weight_variable([5, 5, 1, 32])
b_conv1 = bias_variable([32])
{% endhighlight %}

+ The convolution will compute 32 features for each 5x5 patch.
+ Its weight tensor has a shape of [5, 5, 1, 32], which means [filter_height, filter_width, in_channels, out_channels].


{% highlight python %}
x_image = tf.reshape(x, [-1, 28, 28, 1])
h_conv1 = tf.nn.relu(conv2d(x_image, W_conv1) + b_conv1)
h_pool1 = max_pool_2x2(h_conv1)
{% endhighlight %}

+ First reshape x to a 4d tensor with shape of [batch, in_height, in_width, in_channels].
+ conv --> relu --> pool
+ `max_pool_2x2` reduces the image size to 14 x 14.

### Second Convolutional Layer


{% highlight python %}
W_conv2 = weight_variable([5, 5, 32, 64])
b_conv2 = bias_variable([64])

h_conv2 = tf.nn.relu(conv2d(h_pool1, W_conv2) + b_conv2)
h_pool2 = max_pool_2x2(h_conv2)
{% endhighlight %}

+ Stack several layers of this type to build a deep network.
+ max_pool_2x2 reduces the image size to 7 x 7

### Densely Connected Layer


{% highlight python %}
W_fc1 = weight_variable([7 * 7 * 64, 1024])
b_fc1 = bias_variable([1024])

h_pool2_flat = tf.reshape(h_pool2, [-1, 7 * 7 * 64])
h_fc1 = tf.nn.relu(tf.matmul(h_pool2_flat, W_fc1) + b_fc1)
{% endhighlight %}

+ Add a fully-connected layer with 1024 neurons to allow processing on the entire image.
+ Reshape the tensor from the pooling layer into **a batch of vectors**.

### Dropout


{% highlight python %}
keep_prob = tf.placeholder(tf.float32)
h_fc1_drop = tf.nn.dropout(h_fc1, keep_prob)
{% endhighlight %}

+ Apply dropout before the readout layer to reduce overfitting.
+ Use the `placehold` to save `keep_prob` allows to turn dropout on during training, and turn it off during testing.

### Readout Layer


{% highlight python %}
W_fc2 = weight_variable([1024, 10])
b_fc2 = bias_variable([10])

y_conv = tf.matmul(h_fc1_drop, W_fc2) + b_fc2
{% endhighlight %}

### Train and Evaluate the Model


{% highlight python %}
cross_entropy = tf.reduce_mean(
    tf.nn.softmax_cross_entropy_with_logits(labels=y_, logits=y_conv))

train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)
correct_prediction = tf.equal(tf.argmax(y_conv,1), tf.argmax(y_,1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))

sess.run(tf.global_variables_initializer())

for i in range(5000):
    batch = mnist.train.next_batch(500)
    if i%500 == 0:
        train_accuracy = accuracy.eval(feed_dict={
            x:batch[0], y_:batch[1], keep_prob: 1.0
        })
        print("step %d, training accuracy %.4f"%(i, train_accuracy))
    train_step.run(feed_dict={x: batch[0], y_:batch[1], keep_prob: 0.5})

print("test accuracy %.4f"%accuracy.eval(feed_dict={
    x: mnist.test.images, y_: mnist.test.labels, keep_prob: 1.0}))

{% endhighlight %}
