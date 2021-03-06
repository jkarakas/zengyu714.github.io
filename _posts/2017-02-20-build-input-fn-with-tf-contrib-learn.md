---
author: kimmy
layout: post
title:  "Building Input Functions with tf.contrib.learn"
date:   2017-02-20 21:57:14 +08:00
categories: TensorFlow
tags:
- Python
- Notes
- tf.contrib.learn
- input_fn
---

* content
{:toc}




Use `tf.contrib.learn` to construct a neural network classifier and train it on the Iris data set
to predict flower species based on sepal/petal geometry. See [more](https://www.tensorflow.org/get_started/tflearn)

## Load the Iris CSV data to TensorFlow

+ Each row contains the following data for each flower sample:<br>
    sepal length, sepal width, petal length, petal width, and flower species.
+ Flower species are represented as integers:<br>
    0 denoting Iris setosa, 1 denoting Iris versicolor, and 2 denoting Iris virginica.
+ The Iris data has been **randomized** and split into two separate CSVs: 120 / 30


{% highlight python %}
import tensorflow as tf
import numpy as np
{% endhighlight %}


{% highlight python %}
# Data sets
IRIS_TRAINING = "Iris_CVS_data/iris_training.csv"
IRIS_TEST = "Iris_CVS_data/iris_test.csv"

# Load datasets
training_set = tf.contrib.learn.datasets.base.load_csv_with_header(   # 1
                filename=IRIS_TRAINING,
                target_dtype=np.int,                                  # 2
                features_dtype=np.float32)
test_set = tf.contrib.learn.datasets.base.load_csv_with_header(
                filename=IRIS_TEST,
                target_dtype=np.int,
                features_dtype=np.float32)
{% endhighlight %}

1. `Dataset`s in tf.contrib.learn are `namedtuples`.<br>
    Thus we can can access feature data and target values via the `data` and `target` **fields**.
2. The target is the value you want the model to predict.<br>
    Here is the flower species, which is an integer from 0-2, so the appropriate `numpy` datatype is `np.int`

## Construct a Deep Neural Network Classifier

+ `Estimator`s are a variety of predefined models in `tf.contrib.learn`.
+ Configure a Deep Neural Network Classifier model to fit the Iris data via `tf.contrib.learn.DNNClassifier`.


{% highlight python %}
# Specify that all features have real-value data
feature_columns = [tf.contrib.layers.real_valued_column("", dimension=4)]

# Build 3 layer DNN with 10, 20, 10 units respectively
classifier = tf.contrib.learn.DNNClassifier(
                        feature_columns=feature_columns,
                        hidden_units=[10, 20, 10],
                        n_classes=3,
                        model_dir="Iris_CVS_data/iris_model")
{% endhighlight %}

+ Define the model's feature columns, which specify the **data type** for the features in the data set.
+ Since all the feature data is continuous, so **real-value** is appopriate type.
+ There are four features in the data set(sepal width, sepal height, petal width, and petal height),<br>
so accordingly `dimension` **must** be set to `4` to hold all the data.


## Fit the DNNClassifier to the Iris Training Data


{% highlight python %}
# Fit model
classifier.fit(x=training_set.data, y=training_set.target, steps=2000)
{% endhighlight %}

PROBLEM:
        can't run it in Jupyter, and restart doesn't work.

## Evaluate Model Accuracy

Like `fit`, `evaluate` takes feature data and target values as arguments,
and returns a `dict` with the evaluation results.


{% highlight python %}
accuracy_score = classifier.evaluate(x=test_set.data, y=test.target)["accuracy"]
print('Accuracy: {0:f}'.format(accuracy_score))
{% endhighlight %}

## Classify New Samples

Use the estimator's `predict()` method to classify new samples.


{% highlight python %}
# Classify two new flower samples
new_samples = np.array(
    [[6.4, 3.2, 4.5, 1.5], [5.8, 3.1, 5.0, 1.7]], dtype=float)
y = list(classifier.predict(new_samples, as_iterable=True))
print('Predictions: {}'.format(str(y)))
{% endhighlight %}
