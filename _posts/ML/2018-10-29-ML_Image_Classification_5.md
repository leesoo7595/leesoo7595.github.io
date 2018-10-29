---
layout: post
title: "[ML] Image Classification (5)"
date: 2018-10-29
tags: [ML]
comments: true
---

# Visualizing Intermediate Representations

우리 convnet이 배운 어떤 종류의 기능에 대한 느낌을 얻으려면 재미있는 일은 입력이 convnet을 통과하는 과정에서 어떻게 변형되는지 시각화하는 것입니다. <br>
<br>

학습 세트에서 임의의 고양이 또는 개 이미지를 선택한 다음 각 행이 레이어의 출력이고 그림의 행에있는 각 이미지가 해당 출력 피쳐 맵의 특정 필터인 그림을 생성 해보자. 이 셀을 다시 실행하여 다양한 교육 이미지에 대한 중간 표현을 생성하자.<br>
<br>

```python
import numpy as np
import random
from tensorflow.keras.preprocessing.image import img_to_array, load_img

# Let's define a new Model that will take an image as input, and will output
# intermediate representations for all layers in the previous model after
# the first.
successive_outputs = [layer.output for layer in model.layers[1:]]
visualization_model = Model(img_input, successive_outputs)

# Let's prepare a random input image of a cat or dog from the training set.
cat_img_files = [os.path.join(train_cats_dir, f) for f in train_cat_fnames]
dog_img_files = [os.path.join(train_dogs_dir, f) for f in train_dog_fnames]
img_path = random.choice(cat_img_files + dog_img_files)

img = load_img(img_path, target_size=(150, 150))  # this is a PIL image
x = img_to_array(img)  # Numpy array with shape (150, 150, 3)
x = x.reshape((1,) + x.shape)  # Numpy array with shape (1, 150, 150, 3)

# Rescale by 1/255
x /= 255

# Let's run our image through our network, thus obtaining all
# intermediate representations for this image.
successive_feature_maps = visualization_model.predict(x)

# These are the names of the layers, so can have them as part of our plot
layer_names = [layer.name for layer in model.layers]

# Now let's display our representations
for layer_name, feature_map in zip(layer_names, successive_feature_maps):
  if len(feature_map.shape) == 4:
    # Just do this for the conv / maxpool layers, not the fully-connected layers
    n_features = feature_map.shape[-1]  # number of features in feature map
    # The feature map has shape (1, size, size, n_features)
    size = feature_map.shape[1]
    # We will tile our images in this matrix
    display_grid = np.zeros((size, size * n_features))
    for i in range(n_features):
      # Postprocess the feature to make it visually palatable
      x = feature_map[0, :, :, i]
      x -= x.mean()
      x /= x.std()
      x *= 64
      x += 128
      x = np.clip(x, 0, 255).astype('uint8')
      # We'll tile each filter into this big horizontal grid
      display_grid[:, i * size : (i + 1) * size] = x
    # Display the grid
    scale = 20. / n_features
    plt.figure(figsize=(scale * n_features, scale))
    plt.title(layer_name)
    plt.grid(False)
    plt.imshow(display_grid, aspect='auto', cmap='viridis')
```

보시다시피 이미지의 원시 픽셀에서 점차 추상적이고 압축 된 표현으로 이동한다. 하류 표현은 네트워크가 관심을 기울이는 것을 강조하기 시작하고 점점 더 적은 기능이 "활성화"되었음을 보여준다. 대부분은 0으로 설정된다. 이를 "희소성 (sparsity)"이라고 한다. 표현 희박성은 심층 학습의 핵심 특징이다.

이러한 표현은 이미지의 원본 픽셀에 대한 정보가 점차 줄어들지만 이미지의 클래스에 대한 정보가 점차 정교 해지고 있다. convnet (또는 일반적으로 깊은 네트워크)을 정보 증류 파이프 라인으로 생각할 수 있다.

# Evaluating Accuracy and Loss for the Model

```python
# Retrieve a list of accuracy results on training and test data
# sets for each training epoch
acc = history.history['acc']
val_acc = history.history['val_acc']

# Retrieve a list of list results on training and test data
# sets for each training epoch
loss = history.history['loss']
val_loss = history.history['val_loss']

# Get number of epochs
epochs = range(len(acc))

# Plot training and validation accuracy per epoch
plt.plot(epochs, acc)
plt.plot(epochs, val_acc)
plt.title('Training and validation accuracy')

plt.figure()

# Plot training and validation loss per epoch
plt.plot(epochs, loss)
plt.plot(epochs, val_loss)
plt.title('Training and validation loss')
```

보시다시피, 우리는 패션에서 벗어나는 것처럼 지나치다. 우리의 훈련 정확도(파란색)는 100%에 가까워지고, 우리의 검증 정확도(녹색)는 70%로 떨어진다. 우리의 검증 손실은 단지 5개 이후에 최소에 도달합니다. <br>
<br>

상대적으로 적은 수의 교육 사례 (2000)가 있으므로 `과적합`이 가장 중요한 관심사여야 한다. 모델이 너무 적으면 모델이 새로운 데이터로 일반화되지 않는 패턴을 학습할 때, 즉 모델이 예측을 위해 관련성이없는 기능을 사용하기 시작할 때 과적합이 발생한다. <br>
<br>

모델의 매개 변수를 주어진 데이터 세트에 맞추는 경우 모델에서 습득 한 표현이 이전에 보지 못한 데이터에도 적용될 수 있는지 어떻게 확인할 수 있는가? 교육 자료와 관련된 학습을 피하는 방법은 무엇인가?
<br>
<br>
다음 실습에서 우리는 고양이 대 개 분류 모델에서 과적합을 방지하는 방법을 살펴볼 것입니다.

<br>
<br>
<br>

아니 이제 에러도 안뜨는데 그래프가 안뜨넹 ㅠㅠ 왜구러지아ㅓ닐마ㅓ이ㅏㅁ닝;람
