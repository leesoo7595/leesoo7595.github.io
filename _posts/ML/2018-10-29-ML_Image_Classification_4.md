---
layout: post
title: "[ML] Image Classification (4)"
date: 2018-10-29
tags: [ML]
comments: true
---

# Data Preprocessing

소스 폴더에서 그림을 읽고, float32 텐서로 변환하고, 레이블과 함께 네트워크로 피드하는 데이터 생성기를 설정해보자. 우리는 훈련 이미지와 검증 이미지를 위한 하나의 생성기를 가질 것이다. 우리 발전기는 크기 150x150의 이미지 20개와 라벨 (바이너리)을 생산한다. <br>
<br>
이미 알고 있듯이 신경망에 들어가는 데이터는 일반적으로 네트워크에서 처리하기 쉽도록 정규화되어야한다. 픽셀 값을 [0, 1] 범위로 정규화하여 이미지를 전처리한다. (원래 모든 값은 [0, 255] 범위에 있다) <br>
<br>
Keras에서는 rescale 매개 변수를 사용하여 `keras.preprocessing.image.ImageDataGenerator` 클래스를 통해 이 작업을 수행 할 수 있다. 이 `ImageDataGenerator` 클래스를 사용하면 `.flow` (data, labels) 또는 `.flow_from_directory`(directory)를 통해 확장 된 이미지 배치 (및 해당 레이블)의 생성자를 인스턴스화 할 수 있다. 이러한 생성기는 데이터 생성기를 입력으로 받아들이는 Keras 모델 메서드 인 `fit_generator`, `evaluate_generator` 및 `predict_generator`와 함께 사용할 수 있다.<br>
<br>

```python
from tensorflow.keras.preprocessing.image import ImageDataGenerator

# All images will be rescaled by 1./255
train_datagen = ImageDataGenerator(rescale=1./255)
test_datagen = ImageDataGenerator(rescale=1./255)

# Flow training images in batches of 20 using train_datagen generator
train_generator = train_datagen.flow_from_directory(
        train_dir,  # This is the source directory for training images
        target_size=(150, 150),  # All images will be resized to 150x150
        batch_size=20,
        # Since we use binary_crossentropy loss, we need binary labels
        class_mode='binary')

# Flow validation images in batches of 20 using test_datagen generator
validation_generator = test_datagen.flow_from_directory(
        validation_dir,
        target_size=(150, 150),
        batch_size=20,
        class_mode='binary')
```

# Training

15 개 에포크 (epoch)에 대해 2,000 개의 이미지를 모두 사용할 수 있도록 교육하고 1,000 개의 테스트 이미지를 모두 확인해보자. 실행하려면 몇 분이 걸릴 수 있다.

```python
history = model.fit_generator(
      train_generator,
      steps_per_epoch=100,  # 2000 images = batch_size * steps
      epochs=15,
      validation_data=validation_generator,
      validation_steps=50,  # 1000 images = batch_size * steps
      verbose=2)
```
