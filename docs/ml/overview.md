# Code Tutorial : Machine Learning

This is a tutorial for Machine Learning (Basic).

* Topic 1 : How to set up the ML environment   

- Installation of Ubuntu

- Installation of Machine Learning Library

![Screenshot](../../ml/img/overview-01.jpeg)

-  Test Calculation

```python
import tensorflow.compat.v1 as tf
tf.disable_v_behavior()

hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
sess.run(hello)

a = tf.constant(10)
b = tf.constant(32)

sess.run(a+b)
```

* Topic 2 : How to implement
