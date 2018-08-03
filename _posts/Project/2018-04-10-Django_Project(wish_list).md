---
layout: post
title: "Django_Project(wish_list)"
date: 2018-04-10 00:00:00
img:
categories:
- Project
tags: [Django_Project]
---

### 구현하고 싶은 기능
- 유저가 상품을 좋아요 눌렀을 때, 위시리스트에 상품이 추가

---

### Model
- User model
  - wish_product = models.ManyToManyField()
- Wishlist model
  - travle_info, user / unique_together : 상품을 누른 유저가 같은 상품을 위시 리스트에 추가하지 않기 하게 위해서.
- Travel Product model
- post요청으로 travle_id, user token -> APIView() user  -> 201, userlike 정보 보여주기
- delete 요청으로 travle_id, user token -> api_view

#### 오류
- get_user_model() 로 User선언하면 아직 만들어지지 않은 모델이라 불러올 수 없다는 오류가 발생했었다. 그러나 settings.AUTH_USER_MODEL로 선언하면 오류는 안남 ->  권장하는 방법이 AUTH_USER_MODEL이라고 하심 왜 그런지에 대해 알아올것
- [참고사이트](https://stackoverflow.com/questions/38203301/django-core-exceptions-improperlyconfigured-auth-user-model-refers-to-model-au)

#### settings.AUTH_USER_MODEL vs get_user_model
```py
# 추천
author = models.ForeignKey(settings.AUTH_USER_MODEL)

# 비추
author = models.ForeignKey(get_user_model())
```
- 외래키나 many-to-many 관계에서 User 모델을 지정 할 때, 위와 같은 방법으로 지정하는 것을 권장
- `settings.AUTH_USER_MODEL`은 모든 앱이 로드 될때까지 검색이 연기 되지만, `get_user_model`은 앱이 import 될 때 모델 클래스를 검색하려고 시도한다.
- `get_user_model`은 User 모델이 앱 캐시에 로드 된 것을 보장 할 수 없다. 특정 설정에서만 작동하는 hit-and-miss 시나리오다. 만약 일부 앱 순서가 변경되어 앱을 로드하는 중 중단 될 수 있다.(내 경우에 앱 순서로 인해 User앱을 가져오는데 문제)
- `settings.AUTH_USER_MODEL`은 문자열을 외래키 모델로 전달하여 외래키를 가져올 때 모델 클래스 검색에 실패하면 모든 모델 클래스가 캐시에 로드 될 때까지 검색이 지연된다.

---

### Serializer
- WishListSerializer()
 - user가 있는 사람인지, 상품이 있는지 체크해야 한다.
 - travel_info = PrimaryKeyRelatedField, user = write_only
- 삭제 부분: 유저가 등록된 유저인지 확인하고, 상품있는걸 체크하면 해당 좋아요의 pk를 찾고 이를 삭제해야한다.

#### 예외처리
1. 이미 위시리스트에 담긴 상품을 post로 다시 요청했을 경우.
2. 위시리스트에 없는 travel_info pk를 받았을 때 오류 처리

#### 어려웠던 것들
1. `WishListSerializer()`에서 `uniqueTogetherValidator`를 사용했는데, delete에서 같은 `travel_info` pk값이 들어가는 경우 유효성 오류가 발생한다. 이를 방지하기 위해 serializer를 분리하기로 결정
2. 예외처리 : APIException 사용


----

### APIView

```py
class WishTravelListCreateView(APIView):
    permission_classes = (
        permissions.IsAuthenticated,
    )

    def get(self, request):
        user = User.objects.get(username=request.user)
        wish_lists = user.wish_products.all()
        serializer = TravelInformationWishListSerializer(wish_lists, many=True)
        return Response(serializer.data, status.HTTP_200_OK)

    def post(self, request):
        """
        위시리스트 생성
        :param request:
        :return:
        """

        context = {
            "request": self.request,
        }

        serializer = WishTravelSerializer(data=request.data, context=context)

        if serializer.is_valid(raise_exception=True, ):
            wishlist_created = serializer.save()
            if wishlist_created:
                return Response(serializer.data, status=status.HTTP_201_CREATED)

        else:
            return Response(status.HTTP_400_BAD_REQUEST)
```


### Generic APIView

```py
class WishTravelListCreateView(generics.ListCreateAPIView):
    permission_classes = (
        permissions.IsAuthenticated,
    )

    def get_serializer_context(self):
        return {'request': self.request}

    def get_queryset(self):
        user = User.objects.get(username=self.request.user)
        wish_lists = user.wish_products.all()
        return wish_lists
    def get_serializer_class(self):
        if self.request.method == 'POST':
            return WishTravelSerializer
        return TravelInformationWishListSerializer
```
