---
layout: post
title: "[ML] Image Classification (1)"
date: 2018-10-25
tags: [ML]
comments: true
---

참고 :
- https://developers.google.com/machine-learning/practica/image-classification/check-your-understanding
- https://www.kaggle.com/c/dogs-vs-cats/data

# Cat vs Dog Image classification

## 연습문제 1: Building a Convent from Scratch

우리는 고양이와 개를 구별할 수 있는 분류 기준 모델을 처음부터 구축할 것이다. 이를 위해서 다음을 따른다.

1. 예제 데이터 탐색
2. 분류 문제를 해결하기 위해 처음부터 작은 신경망을 빌드하자
3. 훈련과 검증 정확성을 평가하자

### Explore the Example Data

예제 데이터를 다운로드하고, 고양이와 개를 2000개의 JPG로 찍은 사진을 zip으로 압축하여 `/tmp`에서 로컬로 추출해보겠다. <br>
<br>
Note: 이 연습에 사용된 2,000개의 이미지는 Kaggle에서 제공되는 "Dogs vs. Cats"데이터 세트에서 발췌한 것으로 25,000개의 이미지가 포함되어있다.
여기서는 전체 데이터 집합의 하위 집합을 사용하여 교육 목적의 교육 시간을 줄인다.

```
!wget --no-check-certificate \
    https://storage.googleapis.com/mledu-datasets/cats_and_dogs_filtered.zip \
    -O /tmp/cats_and_dogs_filtered.zip
```
압축된 개와 고양이 사진들을 /tmp 경로에 저장하였다.

```python
import os
import zipfile

local_zip = '/tmp/cats_and_dogs_filtered.zip'
zip_ref = zipfile.ZipFile(local_zip, 'r')
zip_ref.extractall('/tmp')
zip_ref.close()
```

`.zip`의 내용은 기본 디렉토리 `/tmp/cats_and_dogs_filtered`에 추출된다.이 디렉토리에는 교육 및 유효성 검사 데이터 세트에 대한 기차 및 유효성 검사 하위 디렉토리가 포함되어있다. (교육, 유효성 검사 및 테스트 세트에 대한 재 학습을위한 기계 학습 충돌 과정 참조) 차례로 각각 고양이와 개 하위 디렉토리가 포함된다. 각각의 디렉토리를 정의 해보자.

```python
base_dir = '/tmp/cats_and_dogs_filtered'
train_dir = os.path.join(base_dir, 'train')
validation_dir = os.path.join(base_dir, 'validation')

# Directory with our training cat pictures
train_cats_dir = os.path.join(train_dir, 'cats')

# Directory with our training dog pictures
train_dogs_dir = os.path.join(train_dir, 'dogs')

# Directory with our validation cat pictures
validation_cats_dir = os.path.join(validation_dir, 'cats')

# Directory with our validation dog pictures
validation_dogs_dir = os.path.join(validation_dir, 'dogs')
```

이제 고양이와 개의 train 디렉토리에서 파일 이름이 어떻게 보이는지 보자.(파일 이름 지정 규칙은 유효성 검사 디렉토리에서 동일하다.)

```python
train_cat_fnames = os.listdir(train_cats_dir)
print train_cat_fnames[:10]

train_dog_fnames = os.listdir(train_dogs_dir)
train_dog_fnames.sort()
print train_dog_fnames[:10]
```
열차 및 유효성 검사 디렉토리에서 고양이와 개 이미지의 총 개수를 알아보자

```python
print 'total training cat images:', len(os.listdir(train_cats_dir))
print 'total training dog images:', len(os.listdir(train_dogs_dir))
print 'total validation cat images:', len(os.listdir(validation_cats_dir))
print 'total validation dog images:', len(os.listdir(validation_dogs_dir))
```

고양이와 개 이미지는 총 1,000개의 교육용 이미지와 500 개의 테스트 이미지가 있다.
이제 고양이와 개 데이터 세트의 모습을 더 잘 이해하기 위해 몇 장의 그림을 살펴 보자. 먼저 matplot 매개 변수를 구성하자.

<br>
이건 다음 껏부터...에러땜에 잠시 막힘
