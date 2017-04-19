---
author: kimmy
layout: post
title:  Minimal Neural Network Case Study
date:   2017-04-15 08:43:23 +08:00
categories: DeepLearning
tags:
- Python
- Notes
- Deep Learning
- CNN
- Neural Network
---

* content
{:toc}



## Generate the spiral dateset


{% highlight python %}
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline


N = 100  # number of points per class
D = 2    # dimensionality
K = 3    # number of classes

X = np.zeros((N * K, D))
y = np.zeros((N * K), dtype=np.uint8)
y_oh = np.zeros((N * K, K), dtype=np.uint8)
{% endhighlight%}

### Archimedean Spiral

$$\rho = a + b\theta$$

or<br>

$$x = (\alpha + \beta \theta)\ cos(\theta)$$

$$y = (\alpha + \beta \theta)\ sin(\theta)$$


{% highlight python %}
for j in range(3):
    idx = np.arange(j * N, (j + 1) * N)
    radius = np.linspace(0, 1, N)
    theta = np.linspace(j, (j + 1), N) * 4 + np.random.randn(N) * 0.2
    X[idx] = np.c_[radius * np.cos(theta), radius * np.sin(theta)]

    y[idx] = j        # Normal labels
    y_oh[idx, j] = 1  # One-hot encoding
{% endhighlight%}


{% highlight python %}
# s: size, c: color, here is 0, 1, 2
plt.figure(figsize=(5, 5))
plt.scatter(X[:, 0], X[:, 1], c=y, s=40, cmap=plt.cm.winter)
{% endhighlight%}

![spiral]({{ '/styles/images/spiral.png' | prepend: site.baseurl }})


## Training a Softmax Linear Classifier


{% highlight python %}
W = 0.01 * np.random.randn(D, K)
b = np.zeros((1, K))
{% endhighlight%}


{% highlight python %}
scores = np.dot(X, W) + b
exp_scores = np.exp(scores)
probs = exp_scores / np.sum(exp_scores, axis=1, keepdims=True)
correct_log_porbs = -np.log(probs[range(num_examples), y])
{% endhighlight%}


{% highlight python %}
data_loss = np.mean(correct_log_porbs)
reg_loss = 0.5 * reg * np.sum(W * W)
loss = data_loss + reg_loss
{% endhighlight%}

### Computing the Analytic Gradient with Backpropagation

$$Prob_k = \dfrac{e^{f_k}}{\sum_j e^{f_j}}$$

$$Loss_i = -log(p_{y_i})$$

$$\dfrac{\partial L_i}{\partial f_k}= p_k -1 \ (y_i = k)$$

E.g. `p = [0.2, 0.3, 0.5]`, and that the correct class was the middle one (with probability 0.3), `df = [0.2, -0.7, 0.5]`


{% highlight python %}
ds = probs
ds[range(num_examples), y] -= 1
ds /= num_examples
{% endhighlight%}

### Backpropagate gradient into `W` and `b`


{% highlight python %}
dW = np.dot(X.T, ds)
db = np.sum(ds, axis=0, keepdims=True)
dW += factor* W
{% endhighlight%}

### Performing a parameter update


{% highlight python %}
W += -step_size * dW
b += -step_size * db
{% endhighlight%}

### Putting it all together: Training a Softmax Classifier


{% highlight python %}
# initialize parameters randomly
W = 0.01 * np.random.randn(D,K)
b = np.zeros((1,K))

# some hyperparameters
step_size = 1e-0
reg = 1e-3 # regularization strength

# gradient descent loop
num_examples = X.shape[0]
for i in range(200):

    # evaluate class scores, [N x K]
    scores = np.dot(X, W) + b
    exp_scores = np.exp(scores)
    probs = exp_scores / np.sum(exp_scores, axis=1, keepdims=True)  # [N x K]

    # compute the loss: average cross-entropy loss and regularization
    correct_log_porbs = -np.log(probs[y_oh > 0])
    data_loss = np.mean(correct_log_porbs)
    reg_loss = 0.5 * reg * np.sum(W * W)  # regularization strength
    loss = data_loss + reg_loss
    if i % 10 == 0:
        print('iteration %d: loss %f' % (i, loss))

    # compute the gradient on scores
    dscores = probs
    dscores[y_oh > 0] -= 1
    dscores /= num_examples

    # backpropate the gradient to the parameters (W,b)
    dW = np.dot(X.T, dscores)
    db = np.sum(dscores, axis=0, keepdims=True)

    dW += reg * W # regularization gradient

    # perform a parameter update
    W += -step_size * dW
    b += -step_size * db
{% endhighlight%}

    iteration 0: loss 1.099322
    iteration 10: loss 0.907159
    iteration 20: loss 0.835570
    iteration 30: loss 0.802978
    iteration 40: loss 0.785974
    iteration 50: loss 0.776269
    iteration 60: loss 0.770379
    iteration 70: loss 0.766642
    iteration 80: loss 0.764193
    iteration 90: loss 0.762546
    iteration 100: loss 0.761417
    iteration 110: loss 0.760630
    iteration 120: loss 0.760075
    iteration 130: loss 0.759679
    iteration 140: loss 0.759394
    iteration 150: loss 0.759188
    iteration 160: loss 0.759038
    iteration 170: loss 0.758928
    iteration 180: loss 0.758847
    iteration 190: loss 0.758787


### Evaluate training set accuracy


{% highlight python %}
scores = np.dot(X, W) + b
predicted_class = np.argmax(scores, axis=1)
print('training accuracy: %.2f' % (np.mean(predicted_class == y)))
{% endhighlight%}

    training accuracy: 0.52


### Plot the learned decision boundaries


{% highlight python %}
i = np.linspace(-1, 1, 200)
pos_x, pos_y = np.meshgrid(i, i)

# test_X.shape = test_y.shape == (40000, 2)
test_X = np.stack((pos_x.ravel(), pos_y.ravel())).T
test_scores = np.dot(test_X, W) + b
test_y = np.argmax(test_scores, axis=1)

# reshape test_y for plot
test_y = test_y.reshape(200, 200)

fig = plt.figure(figsize=(7, 7))
plt.contourf(pos_x, pos_y, test_y, cmap=plt.cm.winter, alpha=0.4)
plt.scatter(X[:, 0], X[:, 1], c=y, s=40, cmap=plt.cm.winter)
plt.xlim(-1, 1); plt.ylim(-1, 1)
{% endhighlight%}

![softmax classifier]({{ '/styles/images/spiral_softmax.png' | prepend: site.baseurl }})


## Training a Neural Network


{% highlight python %}
# initialize parameters of a 2-layer NN randomly
h = 100  # size of hidden layer
W = 0.01 * np.random.randn(D, h)
b = np.zeros((1, h))
W2 = 0.01 * np.random.randn(h, K)
b2 = np.zeros((1, K))
{% endhighlight%}


{% highlight python %}
# crucial ReLU activation
hidden_layer = np.maximum(0, np.dot(X, W) + b)
scores = np.dot(hidden_layer, W2) + b2
{% endhighlight%}

### Backpropate the gradient to the `W2` and `b2`


{% highlight python %}
dW2 = np.dot(hidden_layer.T, dscores)
db2 = np.sum(dscores, axis=0, keepdims=True)

# continue backpropagation through hidden_layer
dhidden = np.dot(dscores, W2.T)

# backprop the ReLU non-linearity
dhidden[hidden_layer <= 0] = 0

# finally readch the first layer weights and biases
dW = np.dot(X.T, dhidden)
db = np.sum(dhidden, axis=0, keepdims=True)
{% endhighlight%}

### Perform the parameter update


{% highlight python %}
# initialize parameters randomly
h = 100 # size of hidden layer
W = 0.01 * np.random.randn(D, h)
b = np.zeros((1, h))
W2 = 0.01 * np.random.randn(h, K)
b2 = np.zeros((1, K))

# some hyperparameters
step_size = 1e-0
reg = 1e-3 # regularization strength

# gradient descent loop
num_examples = X.shape[0]

for i in range(10000):

    # evaluate class scores, [N x K]
    hidden_layer = np.maximum(0, np.dot(X, W) + b) # note, ReLU activation
    scores = np.dot(hidden_layer, W2) + b2

    # compute the class probabilities
    exp_scores = np.exp(scores)
    probs = exp_scores / np.sum(exp_scores, axis=1, keepdims=True)

    # compute the loss: average cross-entropy loss and regularization
    corect_logprobs = -np.log(probs[range(num_examples), y])
    data_loss = np.mean(corect_logprobs)
    reg_loss = 0.5 * reg * np.sum(W * W) + 0.5 * reg * np.sum(W2 * W2)
    loss = data_loss + reg_loss
    if i % 1000 == 0:
        print('iteration %d: loss %f' % (i, loss))

    # compute the gradient on scores
    dscores = probs
    dscores[range(num_examples),y] -= 1
    dscores /= num_examples

    # backpropate the gradient to the parameters
    dW2 = np.dot(hidden_layer.T, dscores)
    db2 = np.sum(dscores, axis=0, keepdims=True)

    # continue backpropagation through hidden_layer
    dhidden = np.dot(dscores, W2.T)

    # backprop the ReLU non-linearity
    dhidden[hidden_layer <= 0] = 0

    # finally readch the first layer weights and biases
    dW = np.dot(X.T, dhidden)
    db = np.sum(dhidden, axis=0, keepdims=True)

    # add regularization gradient contribution
    dW2 += reg * W2
    dW += reg * W

    # perform a parameter update
    W += -step_size * dW
    b += -step_size * db
    W2 += -step_size * dW2
    b2 += -step_size * db2
{% endhighlight%}

    iteration 0: loss 1.098637
    iteration 1000: loss 0.389487
    iteration 2000: loss 0.278037
    iteration 3000: loss 0.256094
    iteration 4000: loss 0.252357
    iteration 5000: loss 0.250228
    iteration 6000: loss 0.247153
    iteration 7000: loss 0.245801
    iteration 8000: loss 0.245287
    iteration 9000: loss 0.245073


### Evaluate training set accuracy


{% highlight python %}
hidden_layer = np.maximum(0, np.dot(X, W) + b)
scores = np.dot(hidden_layer, W2) + b2
predicted_class = np.argmax(scores, axis=1)
print('training accuracy: %.2f' % (np.mean(predicted_class == y)))
{% endhighlight%}

    training accuracy: 0.99


### Plot the resulting classifier


{% highlight python %}
i = np.linspace(-1, 1, 200)
pos_x, pos_y = np.meshgrid(i, i)

# test_X.shape = test_y.shape == (40000, 2)
test_X = np.stack((pos_x.ravel(), pos_y.ravel())).T
hidden_layer = np.maximum(0, np.dot(test_X, W) + b)
test_scores = np.dot(hidden_layer, W2) + b2
test_y = np.argmax(test_scores, axis=1)

# reshape test_y for plot
test_y = test_y.reshape(200, 200)

fig = plt.figure(figsize=(7, 7))
plt.contourf(pos_x, pos_y, test_y, cmap=plt.cm.winter, alpha=0.4)
plt.scatter(X[:, 0], X[:, 1], c=y, s=40, cmap=plt.cm.winter)
plt.xlim(-1, 1); plt.ylim(-1, 1)
{% endhighlight%}


![neural network classifier]({{ '/styles/images/spiral_nn.png' | prepend: site.baseurl }})

<br><br><br>
> Take Reference from [CS231n](http://cs231n.github.io/neural-networks-case-study/).
