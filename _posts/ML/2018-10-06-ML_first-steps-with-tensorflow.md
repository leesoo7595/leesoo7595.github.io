---
layout: post
title: "[ML] First Steps about Tensorflow"
date: 2018-10-06
tags: [ML]
comments: true
---

참고 : https://developers.google.com/machine-learning/crash-course/first-steps-with-tensorflow/toolkit

# 텐서플로우 첫걸음 : 도구

## 텐서플로우 도구함 현재 계층 구조

- 1층 : **CPU, GPU, TPU**  커널은 하나 이상의 플랫폼에서 작동
- 2층 : **Tensorflow C++**
- 3층 : **Tensorflow Python** C++ 커널을 래핑하는 오퍼레이션 제공
- 4층 : **tf.layers, tf.losses, tf.metrics** 일반 모델 구성요소용 재사용 가능 라이브러리
- 5층 : **Tensorflow Estimators** 높은 수준의 개체 지향 API

## 텐서플로우 도구함 계층구조

- 텐서플로우 : 낮은 수준의 API
- tf.layers/tf.losses/tf.metrics : 일반 모델 구성요소용 라이브러리
- 에스티메이터(tf.estimator) : 높은 수준의 OOP API

## 텐서플로우 구성 요소

- 그래프 프로토콜 버퍼
- 분산된 그래프를 실행하는 런타임

이 구성요소는 자바 컴파일러 및 JVM과 유사하다. JVM이 여러 하드웨어 플랫폼에서 구현되는 것과 마찬가지로 텐서플로우도 여러 CPU, GPU에서 구현된다. <br>

**어느 API를 사용해야 하는가?** <br>
높은 수준의 추상화를 사용해야한다. 추상화 수준이 높을수록 더 사용하기 쉽지만, 유연성이 떨어진다. 그래서 먼저 최고 수준의 API로 시작하여 모든 작업을 실행하는 것이 좋다. <br>
특별한 모델링 문제를 해결하기 위해 더 유연한 추상화가 필요하면, 한 수준 아래로 이동한다. 각 수준은 낮은 수준의 API를 사용하여 제작되므로 계층구조를 낮추는 것이 합리적이다. <br>

## tf.estimator API

tf.estimator은 많은 실습에서 사용된다. 낮은 수준의 텐서플로우를 사용해도 실습의 모든 작업을 실행할 수 있지만, tf.estimator를 사용하면 코드 행 수가 크게 줄어들게 된다. <br>

tf.estimator는 scikit-learn API와 호환된다. scikit-learn은 Python의 매우 인기 있는 오픈소스 ML 라이브러리로, 100,000명이 넘는 사람들이 이용하고 있다. <br>

tf.estimator로 구현된 선형 회귀 프로그램의 형식은 대체로 다음과 같다. <br>

```Python
import tensorflow as tf

# Set up a linear classifier
classifier = tf.estimator.LinearClassifier()

# Train the model on some example data
classifier.train(input_fn=train_input_fn, steps=2000)

# Use it to predict
predictions = classifier.predict(input_fn=predict_input_fn)
```
