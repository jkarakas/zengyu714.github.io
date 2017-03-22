---
layout: post
title:  CS20SI Assignment 1.2 Logistic Regression
date:   2017-03-22 09:59:58 +08:00
categories: TensorFlow
tags:
- tf.random_normal
- tf.matmul

---

* content
{:toc}



## Task 1: Improving the accuracy of our logistic regression on MNIST

### Implement


{% highlight python %}
"""
Starter code for logistic regression model to solve OCR task
with MNIST in TensorFlow
MNIST dataset: yann.lecun.com/exdb/mnist/
"""

import tensorflow as tf
import numpy as np
from tensorflow.examples.tutorials.mnist import input_data
import time
{% endhighlight %}


{% highlight python %}
# Define paramaters for the model
learning_rate = 1
batch_size = 128
n_epochs = 100
{% endhighlight %}


{% highlight python %}
# Step 1: Read in data
# using TF Learn's built in function to load MNIST data to the folder data/mnist

mnist = input_data.read_data_sets('/home/zengyu/tmp/MNIST', one_hot=True)
{% endhighlight %}


{% highlight python %}
# Step 2: create placeholders for features and labels
# each image in the MNIST data is of shape 28*28 = 784
# therefore, each image is represented with a 1x784 tensor
# there are 10 classes for each image, corresponding to digits 0 - 9.

x = tf.placeholder(shape=[None, 28*28], dtype=tf.float32)
y = tf.placeholder(shape=[None, 10], dtype=tf.uint8)
{% endhighlight %}


{% highlight python %}
# Step 3: create weights and bias
# weights and biases are initialized to 0
# shape of w depends on the dimension of X and Y so that Y = X * w + b
# shape of b depends on Y

weight_1 = tf.Variable(tf.random_normal(shape=[28*28, 500], stddev=0.01))
bias_1 = tf.Variable(tf.zeros([500]))

weight_2 = tf.Variable(tf.random_normal(shape=[500, 500], stddev=0.01))
bias_2 = tf.Variable(tf.zeros([500]))

weight_3 = tf.Variable(tf.random_normal(shape=[500, 10], stddev=0.01))
bias_3 = tf.Variable(tf.zeros([10]))
{% endhighlight %}


{% highlight python %}
# Step 4: build model
# the model that returns the logits.
# this logits will be later passed through softmax layer
# to get the probability distribution of possible label of the image
# DO NOT DO SOFTMAX HERE

hidden_1 = tf.nn.relu(tf.matmul(x, weight_1) + bias_1)
hidden_2 = tf.nn.relu(tf.matmul(hidden_1, weight_2) + bias_2)
y_out = tf.matmul(hidden_2, weight_3) + bias_3
{% endhighlight %}


{% highlight python %}
# Step 5: define loss function
# use cross entropy loss of the real labels with the softmax of logits
# use the method:
# tf.nn.softmax_cross_entropy_with_logits(logits, Y)
# then use tf.reduce_mean to get the mean loss of the batch

loss = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(logits=y_out, labels=y))
{% endhighlight %}


{% highlight python %}
# Step 6: define training op
# using gradient descent to minimize loss

train_op = tf.train.GradientDescentOptimizer(learning_rate).minimize(loss)

sess = tf.InteractiveSession()
tf.global_variables_initializer().run()

start_time = time.time()

n_batches = int(mnist.train.num_examples/batch_size)
for i in range(n_epochs): # train the model n_epochs times
    total_loss = 0

    for _ in range(n_batches):
        x_batch, y_batch = mnist.train.next_batch(batch_size)
        loss_batch, _ = sess.run([loss, train_op], feed_dict={x: x_batch, y: y_batch})
        total_loss += loss_batch
    print('Average loss epoch {0}: {1}'.format(i, total_loss/n_batches))

print('Total time: {0} seconds'.format(time.time() - start_time))
print('Optimization Finished!') # should be around 0.35 after 25 epochs

# test the model
correct_prediction = tf.equal(tf.argmax(y, 1), tf.argmax(y_out, 1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
acc = sess.run(accuracy, feed_dict={x: mnist.test.images, y: mnist.test.labels})
print('Accuracy {0:7.3f}%'.format(acc*100))
{% endhighlight %}

### Discussion

1. For the output layer, the activation function often depends on the cost function.<br>
    E.g, <br>
    + **MSE** cost function in a regression task with **linear** activation function. <br>
    + **Cross-Entropy** loss with the **softmax** activation function.
2. Generally speaking, as training epoches increase, the test accuracy will also increase, but it is not guaranteed. <br>
   See [more](http://cs.nyu.edu/~wanli/dropc/).

### Result

+ Total time: 81.06000566482544 seconds
+ Accuracy  98.340%
