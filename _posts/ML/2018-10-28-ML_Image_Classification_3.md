---
layout: post
title: "[ML] Image Classification (3)"
date: 2018-10-28
tags: [ML]
comments: true
---

python version 3.6.6으로 깔아서 시도하려는데... 자꾸 최신버전(3.7.0)으로 깔려가지고 포맷하고 다시하는 수고를 하였다. <br>
그런데 마찬가지로 안돼서...python을 배운 강사님께 도움을 청함...<br>

<br>
어찌저찌 해결하여 다시 플젝 시작하여 하는중 핫...이제 잘되길..
<br>

# Building a Small Convnet from Scratch to Get to 72% Accuracy

convnet에 들어갈 이미지는 150x150 컬러 이미지이다. (데이터 사전 처리의 다음 섹션에서는 모든 이미지를 150x150 크기로 조정 한 후 신경망에 입력하는 처리 기능을 추가한다.) <br>
<br>

아키텍처를 코딩 해보자. 우리는 3개의 {convolution + relu + maxpooling} 모듈을 쌓을 것이다. 우리의 컨볼루션은 3x3 창에서 작동하며 maxpooling 레이어는 2x2 창에서 작동한다. 첫 번째 컨볼 루션은 16개의 필터를 추출하고, 다음 필터는 32개의 필터를 추출하고, 마지막 필터는 64개의 필터를 추출한다. <br>
<br>

NOTE: 이것은 이미지 분류에 널리 사용되며 잘 알려진 구성이다. 또한, 훈련 예 (1,000)가 비교적 적기 때문에, 단지 3개의 모델에 대한 (convolutional) 모듈을 사용하면 모델이 작아지므로 과적합의 위험이 줄어든다. (연습2에서 더 자세히 살펴볼 것이다). <br>
<br>

```python
from tensorflow.keras import layers
from tensorflow.keras import Model

# Our input feature map is 150x150x3: 150x150 for the image pixels, and 3 for
# the three color channels: R, G, and B
img_input = layers.Input(shape=(150, 150, 3))

# First convolution extracts 16 filters that are 3x3
# Convolution is followed by max-pooling layer with a 2x2 window
x = layers.Conv2D(16, 3, activation='relu')(img_input)
x = layers.MaxPooling2D(2)(x)

# Second convolution extracts 32 filters that are 3x3
# Convolution is followed by max-pooling layer with a 2x2 window
x = layers.Conv2D(32, 3, activation='relu')(x)
x = layers.MaxPooling2D(2)(x)

# Third convolution extracts 64 filters that are 3x3
# Convolution is followed by max-pooling layer with a 2x2 window
x = layers.Conv2D(64, 3, activation='relu')(x)
x = layers.MaxPooling2D(2)(x)
```

그 위에 완전히 연결된 두 개의 레이어를 붙인다. 우리는 2등급 분류 문제, 즉 2진 분류 문제에 직면하고 있기 때문에 네트워크의 출력을 Sigmoid 활성화로 끝내므로 우리 네트워크의 출력은 0과 1 사이의 단일 스칼라가되어 현재의 이미지는 클래스 1 (클래스 0과 반대)이다. <br>
<br>

```python
# Flatten feature map to a 1-dim tensor so we can add fully connected layers
x = layers.Flatten()(x)

# Create a fully connected layer with ReLU activation and 512 hidden units
x = layers.Dense(512, activation='relu')(x)

# Create output layer with a single node and sigmoid activation
output = layers.Dense(1, activation='sigmoid')(x)

# Create model:
# input = input feature map
# output = input feature map + stacked convolution/maxpooling layers + fully
# connected layer + sigmoid output layer
model = Model(img_input, output)
```

이 아키텍처를 요약해보자.

```python
model.summary()
```
"출력 모양"열은 각 연속 레이어에서 기능 맵의 크기가 어떻게 변하는 지 보여준다. 컨볼 루션 계층은 패딩으로 인해 기능 맵의 크기를 약간 줄이고 각 풀링 계층은 기능 맵을 반으로 줄인다.
<br>
<br>

다음은 모델 교육을 위한 사양을 구성한다. binary_crossentropy 손실로 모델을 학습 할 것이다. 왜냐하면 바이너리 분류 문제이고 최종 활성화는 sigmoid이기 때문이다. (손실 측정법에 대한 자세한 내용은 Machine Learning Crash 과정을 참조하자.) rmsprop 최적화 알고리즘을 학습률 0.001로 사용합니다. 훈련 중, 우리는 분류의 정확성을 모니터하는 것이 목적이다. <br>
<br>

NOTE: 이 경우 RMSprop 최적화 알고리즘을 사용하면 SGD (Stochastic Gradient descent)보다 바람직합니다. RMSprop이 학습 속도 조정을 자동화하기 때문입니다. (Adam과 Adagrad와 같은 다른 옵티 마이저도 교육 중 학습 속도를 자동으로 조정하며 여기서도 똑같이 잘 작동합니다.) <br>
<br>

```python
from tensorflow.keras.optimizers import RMSprop

model.compile(loss='binary_crossentropy',
              optimizer=RMSprop(lr=0.001),
              metrics=['acc'])
```
