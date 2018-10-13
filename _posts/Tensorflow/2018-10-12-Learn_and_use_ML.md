---
layout: post
title: "Learn and use ML - (1) Basic classification"
date: 2018-10-12
tags: [Tensorflow]
comments: true
---

참고 : https://www.tensorflow.org/tutorials/keras/basic_classification <br>
<br>
해당 글은 위 사이트를 번역한 것 입니다. <br>
<br>

# Train your first neural network: basic classification

해당 가이드는 스니커즈와 셔츠와 같이, 옷 입기에 대한 분석 이미 모델의 신경망 네트워크를 훈련시키는 것이다. 당신이 모든 세부사항을 이해하지 않아도 괜찮다. 이것은 우리가 목표하는 텐서플로우 프로그램의 세부사항을 재빠르게 완료하기 위한 것이다. (확인 요망) <br>
<br>

이 가이드는 `tf.keras`를 사용하고, 텐서플로우에서 상위 레벨의 API를 빌드하고 모델을 훈련시킨다. <br>
<br>

```python
# TensorFlow and tf.keras
import tensorflow as tf
from tensorflow import keras

# Helper libraries
import numpy as np
import matplotlib.pyplot as plt

print(tf.__version__)
```
`1.11.0`


## Import the Fashion MNIST dataset

 이 가이드는 10개의 카테고리에서 70,000개의 회색조 이미지들을 포함하고 있는 `Fashion MNIST` 데이터셋을 사용한다. 아래와 같이 저해상도의 개별 부분이 이미지로 보여준다.


<br>
<br>
Fashion MNIST는 컴퓨터 비전을위한 기계 학습 프로그램의 "Hello, World"로 자주 사용되는 고전적인 MNIST 데이터 세트 예시입니다. MNIST 데이터셋은 우리가 사용할
