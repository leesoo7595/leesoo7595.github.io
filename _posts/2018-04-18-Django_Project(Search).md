---
layout: post
title: "Django_Project(search)"
date: 2018-04-13 00:00:00
img:
tags: [Django_Project]
---

### 구현하고 싶은 기능
1. url로 keywrod를 받아서 상품 제목과 설명에서 이를 가지고 있는 상품을 결과값으로 출력
  - 최근 검색 기록
2. 상품 가격, 상품 인기순(위시리스트 많은 것)으로 필터링 해서 상품 결과를 출력
 - 상품 리스트 보여줄 때
 - 상품 검색할 때

---

### 상품 검색하기 - 기본
- Django Query Filter / Django Search Filter 적용하기
- url : /search/q={keyword}

#### 예외처리
1. 키워드가 `None`인 경우
2. 키워드에 맞는 상품이 없는 경우

#### 수정전
url : http://localhost:8000/search/?keyword=

```py
# apis/ search.py
class SearchTravelInformationView(generics.ListCreateAPIView):
    serializer_class = TravelInformationSerializer
    permission_classes = (
        permissions.IsAuthenticated,
    )
    def get_queryset(self):
        keyword = self.request.query_params.get('keyword', None)
        # keyword가 none인 경우엔 모든게 보여짐
        # 예외처리
        # 1. 키워드가 none인 경우 return
        # 2. 검색한 키워드 결과가 없어서 빈 쿼리셋인 경우

        queryset = TravelInformation.objects.filter(
            Q(name__contains=keyword) | Q(description__contains=keyword)
            | Q(description_title__contains=keyword)
        ).distinct()
        return queryset
```

---

#### Django Rest Filter 사용하기
- 설치하기

```console
pip install django-filter
```

- settings.py 에 추가

```py
# settings/base.py
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': ('django_filters.rest_framework.DjangoFilterBackend',)
}
```

#### 수정후
url : http://localhost:8000/search/?search=

```py
# apis/search.py

from rest_framework import generics, permissions, filters,

class SearchTravelInformationView(generics.ListCreateAPIView):
    serializer_class = TravelInformationSerializer
    permission_classes = (
        permissions.IsAuthenticated,
    )
    # filter_backends 의 경우 반드시 튜플형태로
    filter_backends = (filters.SearchFilter,)
    queryset = TravelInformation.objects.all()
    search_fields = ('name', 'description', 'description_title')
```
- search_fields : `CharField`와 `TextField` 와 같은 텍스트 필드의 속성만 검색 가능하다. 만약 many-to-many 혹은 외래키로 관계가 되어있는 경우는 lookup `__`를 사용해서 접근가능하다.
- `search_fields`에 특정 문자들을 추가하면 문자에 맞는 동작을 한다.
  - `^` : 검색시작
  - `=` : 추가적으로 연결 ex) `search_fields = ('=username', '=email')`
  - `@` : full-text 검색 (MySQL 백엔드에서만 사용가능)
  - `$` : Regex 검색
