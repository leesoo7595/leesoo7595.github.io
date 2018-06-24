---
layout: post
title: "Django_Project(image)"
date: 2018-04-09 00:00:00
img:
tags: [Django_Project]
---

### 구현하고 싶은 기능
- 이미지 파일을 저장할 때, 각 기종마다 이미지 사이즈를 저장하여 원본 이미지를 리사이징하는 기능 구현

---

### 썸네일
- 라이브러리 종류들
- 왜 django imagekit 을 사용했는지?
- 동작 원리
- DRF 에서 왜 read_only로 설정해야 오류가 안나는지에 대해서..
https://github.com/matthewwithanm/django-imagekit/issues/391
https://github.com/matthewwithanm/django-imagekit/issues/205
https://www.pydoc.io/pypi/django-imagekit-4.0.2/autoapi/cachefiles/strategies/index.html


### Pillow 활용한 썸네일 만들기

### 정리
- 결과적으로 프로젝트에서 `imagekit`을 거둬내기로 결정했다. 로컬환경에서는 제대로 동작하는 것이 확인이 되었지만 서버에 올렸을 때 여러가지 문제가 있었다.
  - `IMAGEKIT_DEFAULT_CACHEFILE_STRATEGY` 설정을 Justintime이 아닌 다른 설정으로하여 파일을 save할때 캐시파일도 함께 생성되도록 설정했다. 이 경우 s3에 생성된 파일이 있고 다시 크롤링을 해서 파일을 저장시키면 정상 작동을 했다.
  - 그러나 처음 크롤링하는 경우에 파일이 save가 되고 캐시가 생성되지 않는 문제가 발생.
  - 여러가지 방법을 찾아봤지만 이미지를 생성할때 binary data를 넘겨주는 부분에서도 오류가 발생.
