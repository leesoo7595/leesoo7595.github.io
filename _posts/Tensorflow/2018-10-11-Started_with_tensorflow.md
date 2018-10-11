---
layout: post
title: "Get Started with Tensorflow"
date: 2018-10-11
tags: [Tensorflow]
comments: true
---

참고 : https://www.tensorflow.org/tutorials/

이 글은 위 사이트를 번역한 것 입니다.

# Get Started with tensorflow

텐서플로우는 어떤 생산물을 연구하기 위한 머신러닝 오픈소스이다. <br>
텐서플로우는 데스크탑, 모바일, 웹, 클라우드를 개발하는 입문자나 전문가를 위해 API를 제공한다. <br>
<br>

```python
import tensorflow as tf
mnist = tf.keras.datasets.mnist

(x_train, y_train),(x_test, y_test) = mnist.load_data()
x_train, x_test = x_train / 255.0, x_test / 255.0

model = tf.keras.models.Sequential([
  tf.keras.layers.Flatten(),
  tf.keras.layers.Dense(512, activation=tf.nn.relu),
  tf.keras.layers.Dropout(0.2),
  tf.keras.layers.Dense(10, activation=tf.nn.softmax)
])
model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

model.fit(x_train, y_train, epochs=5)
model.evaluate(x_test, y_test)
```
