---
layout: post
title: "[ML] Image Classification (2)"
date: 2018-10-25
tags: [ML]
comments: true
---

## 분석 이미지 띄우기

```python
%matplotlib inline

import matplotlib.pyplot as plt
import matplotlib.image as mpimg

# Parameters for our graph; we'll output images in a 4x4 configuration
nrows = 4
ncols = 4

# Index for iterating over images
pic_index = 0
```

위는 그림을 나타내줄 변수만 구성한 것임... 이것땜에 두 시간 삽질 ^^..

```python
# Set up matplotlib fig, and size it to fit 4x4 pics
fig = plt.gcf()
fig.set_size_inches(ncols * 4, nrows * 4)

pic_index += 8
next_cat_pix = [os.path.join(train_cats_dir, fname)
                for fname in train_cat_fnames[pic_index-8:pic_index]]
next_dog_pix = [os.path.join(train_dogs_dir, fname)
                for fname in train_dog_fnames[pic_index-8:pic_index]]

for i, img_path in enumerate(next_cat_pix+next_dog_pix):
  # Set up subplot; subplot indices start at 1
  sp = plt.subplot(nrows, ncols, i + 1)
  sp.axis('Off') # Don't show axes (or gridlines)

  img = mpimg.imread(img_path)
  plt.imshow(img)

plt.show()

```

고양이 사진 4장, 개 사진 4장 2라인으로 그림 보여주기

## Building a Small Convnet from Scratch to Get to 72% Accuracy

convnet에 들어갈 이미지는 150x150 컬러 이미지이다. (데이터 사전 처리의 다음 섹션에서는 모든 이미지를 150x150 크기로 조정 한 후 신경망에 입력하는 처리 기능을 추가할 것이다.) <br>

아키텍처를 코딩해보자. 우리는 3개의 {convolution + relu + maxpooling} 모듈을 쌓을 것이다.
이 Convolution은 3x3 창에서 작동하며 maxpooling 레이어는 2x2 창에서 작동한다.
<br>
<br>
구글 번역 쓰고있지만 뭔 소린지 하나도 모루겟군...
<br>
<br>

첫 번째 컨볼 루션은 16 개의 필터를 추출하고, 다음 필터는 32 개의 필터를 추출하고, 마지막 필터는 64 개의 필터를 추출한다.

NOTE: 이것은 널리 사용되고 이미지 분류에 잘 작동하는 것으로 알려진 구성이다. 또한, 훈련 예(1,000)가 비교적 적기때문에, 단지 3개의 모델로 (convolutional) 모듈을 사용하면 모델이 작아 지므로 과적합의 위험이 줄어든다.(연습 2에서 더 자세히 살펴볼 것이다.)

<br>
<br>
헐...tensorflow 설치하다 다 꼬임 ...ㅠㅠㅠㅠㅠㅠ일단 보류 ㅠㅠㅠㅠㅠ삽질 엄청 햇네
