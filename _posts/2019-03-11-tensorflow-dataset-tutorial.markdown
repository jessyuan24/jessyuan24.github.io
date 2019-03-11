---
layout: post
title:  "Tensorflow Dataset API"
date:   2019-03-11 12:00:00 +0800
categories: Deep Learning
tags: tensorflow, deep-learning
---
## Tensorflow Dataset API
在Tensorflow中，feed-dict方式对Model传输数据速率慢， 使用管道输入(pipeline)传输速率比较快，而Tensorflow内置了一个API(tf.data)，它可以方便地处理数据以及高效地传输数据给Model。 这篇我将会讲述tf.data API基本机制和一些常用的操作。

tf.data API主要有两个对象：tf.data.Dataset和tf.data.Iterator
tf.data.Dataset：存储一列表数据元素的Tensor对象。
tf.data.Iterator： 提供访问Dataset的数据元素的主要方式

### 大概提纲
* 创建数据集(Create Dataset)
* 创建迭代器(Craete Iterator)
* 例子
* 总结

### 创建数据集(Craete Dataset)
Dataset用于存储我们的数据元素
``` python
# 导入Library
import numpy as np
import tensorflow as tf
```
1. From Numpy
    ```python
    # 创建随机数据
    x = np.random.sample((5,2))
    # 创建Dataset
    dataset = tf.data.Dataset.from_tensor_slices(x)

    dataset
    ```

    ```python
    # 创建两个numpy array
    x, y = np.random.sample((5,2)), np.random.sample((5,1))
    # 两个numpy array同时创建Dataset
    dataset = tf.data.Dataset.from_tensor_slices((x, y))

    dataset
    ```

2. From Tensor对象
    ```python
    a = tf.random_uniform([5,2])
    dataset = tf.data.Dataset.from_tensor_slices(a)

    dataset
    ```

3. From Placeholder
    ```python
    # 通过placeholder来创建Dataset，可以动态改变数据的来源，在训练Model的时候非常有用，比如训练数据集和测试数据集
    input = tf.placeholder(tf.float64, shape=[None, 2])
    dataset = tf.data.Dataset.from_tensor_slices(input)

    dataset
    ```

4. From CSV文件
    ```python
    # 通过CSV文件数据创建Dataset
    csv_file = 'GSPC.csv'
    dataset = tf.contrib.data.make_csv_dataset(csv_file, batch_size=32)

    iterator = dataset.make_one_shot_iterator()
    next_element = iterator.get_next()

    Date, Open = next_element['Date'], next_element['Open']

    with tf.Session() as sess:
      print(sess.run([Date, Open]))
    ```

### 创建迭代器(Create Iterator)
Iterator用于访问和获取Dataset的每一个数据元素

1. One-hot Iterator  
one-hot是最简单的迭代器(Iterator)，它可以处理大多数的数据管道输入(pipline)的情况。
    ```python
    # 创建Dataset
    x = np.random.sample((5,2))
    dataset = tf.data.Dataset.from_tensor_slices(x)

    # 创建Iterator
    iterator = dataset.make_one_shot_iterator()
    next_element = iterator.get_next()

    with tf.Session() as sess:
      for _ in range(5):
        # 获取每个元素
        print(sess.run(next_element))
    ```

2. Initializable Iterator  
它可以初始化不同数据的，但还是同一个Dataset， 比如训练数据集和测试数据集。
它配合placeholder来使用
    ```python
    # 创建Dataset
    input = tf.placeholder(tf.float64,shape=[None, 2])
    dataset = tf.data.Dataset.from_tensor_slices(input)

    train_x = np.random.sample((5,2))
    test_x = np.random.sample((3,2))

    # 创建Iterator
    iterator = dataset.make_initializable_iterator()
    next_element = iterator.get_next()

    with tf.Session() as sess:
      # trainning model
      # 使用trainning数据集，并初始化Iteraotr
      sess.run(iterator.initializer, feed_dict={input: train_x})
      for _ in range(5):
        print(sess.run(next_element))
        
      print("--------")
      
      # evaluate model
      # 使用测试数据集，初始化Iterator
      sess.run(iterator.initializer, feed_dict={input: test_x})
      for _ in range(3):
        print(sess.run(next_element))
    ```

3. Reinitializable Iterator  
reinitializable可以初始化两个不同数据和不同Dataset
    ```python
      # 创建Dataset
    training_dataset = tf.data.Dataset.from_tensor_slices(np.random.sample((5,2)))
    test_dataset = tf.data.Dataset.from_tensor_slices(np.random.sample((3,2)))

    # 创建Iterator
    iterator = tf.data.Iterator.from_structure(training_dataset.output_types,
                                              training_dataset.output_shapes)
    next_element = iterator.get_next()

    # 创建两个不同数据集的初始化
    training_init_op = iterator.make_initializer(training_dataset)
    test_init_op = iterator.make_initializer(test_dataset)

    with tf.Session() as sess:
      sess.run(training_init_op)
      for _ in range(5):
        print(sess.run(next_element))
      
      print("---------")
      
      sess.run(test_init_op)
      for _ in range(3):
        print(sess.run(next_element))
    ```

4. Feedable Iterator  
feedable类似与reinitialazable，但是feedable是创建两个不同的Iterator，来自不同数据集，切换Iterator时不用重新初始化
    ```python
    training_dataset = tf.data.Dataset.from_tensor_slices(np.random.sample((5,2)))
    test_dataset = tf.data.Dataset.from_tensor_slices(np.random.sample((3,2)))

    handle = tf.placeholder(tf.string, shape=[])
    iterator = tf.data.Iterator.from_string_handle(handle,
                                                  training_dataset.output_types,
                                                  training_dataset.output_shapes)
    next_element = iterator.get_next()

    training_iterator = training_dataset.make_one_shot_iterator()
    test_iterator = test_dataset.make_initializable_iterator()

    with tf.Session() as sess:
      training_handle = sess.run(training_iterator.string_handle())
      test_handle = sess.run(test_iterator.string_handle())
      
      for _ in range(5):
        print(sess.run(next_element, feed_dict={handle: training_handle}))
      
      print("----------")
      
      sess.run(test_iterator.initializer)
      for _ in range(3):
        print(sess.run(next_element, feed_dict={handle: test_handle}))
    ```

### 例子
前面的例子都是通过在Session中打印get_next()的值，下面通过一个例子，来实现Dataset的数据传值给Model来训练。
1. One-hot的例子
    ```python
    # 生成数据
    features = np.random.sample((100, 1))
    labels = 2 * features + 1.5

    epoches = 10
    batch_size = 32

    # 创建Dataset
    dataset = tf.data.Dataset.from_tensor_slices((features, labels))
    dataset = dataset.batch(batch_size).repeat()

    iterator = dataset.make_one_shot_iterator()
    x, y = iterator.get_next()

    # 建立Model
    layer1 = tf.layers.dense(x, 4, activation=tf.tanh)
    layer2 = tf.layers.dense(layer1, 4, activation=tf.tanh)
    predictions = tf.layers.dense(layer2,1, activation=tf.sigmoid)

    loss = tf.losses.mean_squared_error(predictions, y)
    optimizer = tf.train.AdamOptimizer().minimize(loss)

    with tf.Session() as sess:
      sess.run(tf.global_variables_initializer())
      for i in range(epoches):
        _, l = sess.run([optimizer, loss])
        print("Epoch: {}, Loss: {}".format((i+1), l))
    ```

2. Initializable例子
    ```python
    # 生成数据
    training_data = (np.random.sample((100,2)), np.random.sample((100, 1)))
    test_data = (np.random.sample((30, 2)), np.random.sample((30,1)))

    epoches = 10
    batch_size = 32

    input, labels = tf.placeholder(tf.float32, shape=[None, 2]), tf.placeholder(tf.float32, shape=[None,1])
    dataset = tf.data.Dataset.from_tensor_slices((input, labels)).batch(batch_size).repeat()


    iterator = dataset.make_initializable_iterator()
    x, y = iterator.get_next()

    # 建立Model
    layer1 = tf.layers.dense(x, 4, activation=tf.tanh)
    layer2 = tf.layers.dense(layer1, 4, activation=tf.tanh)
    predictions = tf.layers.dense(layer2,1, activation=tf.sigmoid)

    loss = tf.losses.mean_squared_error(predictions, y)
    optimizer = tf.train.AdamOptimizer().minimize(loss)

    with tf.Session() as sess:
      sess.run(tf.global_variables_initializer())
      sess.run(iterator.initializer, feed_dict={input: training_data[0], 
                                              labels: training_data[1]})
      print("Training...")
      for i in range(epoches):
        total_loss = 0
        for _ in range(10):
          _, l = sess.run([optimizer, loss])
          total_loss += l
        print("Epoches: {}, Loss: {}".format(i, total_loss / 10))
      
      print("Testing...")
      sess.run(iterator.initializer, feed_dict={input: test_data[0],
                                              labels: test_data[1]})
      print("Test loss: {}".format(sess.run(loss)))
    ```

### 总结
Datset API为我们提供了快捷和高效的方式来生成数据集输入管道(pipeline)，可为Model快速训练，评估，测试。

以上的代码在[github][github]里面

[github]: https://github.com/jessyuan24/dataset_tutorial