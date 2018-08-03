---
layout: post
title: "Django_Project(blog)"
date: 2018-04-13 00:00:00
img:
categories:
- Project
tags: [Django_Project]
---

### 구현하고 싶은 기능
1. 로그인된 유저가 예약했던 상품에 대한 후기 작성
2. 상품 후기 조회의 경우 유저 자신의 것만 보여주는 것과 전체 공개 부분 분리

---

### 상품 후기 목록/ 생성
1. 자기가 쓴 상품 후기 목록 보기
  - get 요청
  - ` Blog.objects.select_related('travel_Reservation').filter(travel_Reservation__member=request.user` vs `blog.objects.all().filter(travel_Reservation__member=user)`
  - 목록 출력의 경우 상세 필드들에 접근할 필요가 없기 때문에 `select_related`로 모든 데이터들을 캐시로 저장하는 것보단 DB에 접근하는게 낫지 않을까? (강사님께 여쭤보기)
  - **DB에 여러번 왔다갔다 하는것보단 캐시로 저장하는 편이 낫다는 결론!**

2. 상품 후기 작성하기
  - post 요청
  - body = resrvation pk, title , contents, score , images
  - status.201 , user blog 보여주기
  - 여러장의 이미지가 들어가면 `serialize.ListSerializer`를 알아볼 것


#### select_related
- [참고 사이트](https://jupiny.com/2016/10/06/select_related-prefetch_related/)
- Reservation 테이블과 Blog 테이블은 one_to_one 으로 이루어져있다. 자기가 쓴 상품 후기 목록을 가져오기 위해선 Blog - Reservation -(Foreign)- User를 가져와야 했다. 그런데 QuerySet에서 접근하는 부분에서 어려움이 있었다. 그렇게 검색을 통해 알게된 `select_related`
- `select_related`: 쿼리셋을 가져올 때, releated objects들을 다 불러오는 함수이다. 쿼리가 조금 복잡해지긴 하지만, 데이터를 불러올 cache에 남아 있어서 DB에 다시 접근하는 것을 줄여준다. 그리고 foreign-key, one-to-one 관계에서만 사용가능하다.

```py
In [1]: user = User.objects.first()
In [2]: blog_all = Blog.objects.all().filter(travel_Reservation__member=user).first()
In [3]: blog_select = Blog.objects.select_related('travel_Reservation').filter(travel_Reservation__member = user).first()
# all을 사용하면 DB에 두번을 접근하게 된다.
In [4]: blog_all
Out[4]: <Blog: Blog object (1)>
In [5]: blog_all.title
Out[5]: 'hello'
# select_related을 사용하면 blog_select을 선언할 때 모든 관계를 가져오기 때문에 DB에 한번만 접근하면 된다.
In [6]: blog_select
Out[6]: <Blog: Blog object (1)>
In [7]: blog_select.title
Out[7]: 'hello'
```

####  prefetch_related
- many-to-many, many-to-one 관계에서 사용할 수 있다.


#### 어려웠던 부분
- 여러 장의 이미지를 받았을 경우 serialize에서 어떻게 처리하는가에 대한 부분이 힘들었던 것 같다. `ListSerializer`를 사용 할 수도 있었지만 테이블마다 연결된 관계를 가진 경우에 처리하는 방법을 몰라서 `ListField`를 사용했다. 그러나 이 serialize의 경우 return 할때 `images`가 리스트가 아니기 때문에 오류가 생겼다. 그래서 return 할때는 createserialize 가아닌 다른 serialize를 생성하여 결과물을 출력한다.
- 예외처리부분에서도 field 자체에서 request.user를 받아서 처리하는 부분에서 오류가 발생해서 `validate_travel_reservation` 을 만들어서 예약 번호가 예약 테이블에는 존재하지만 이 예약번호가 해당 유저의 예약 리스트에 없는 경우에는 오류가 발생하도록 처리했다.

---

### APIView
- blog/serialize/blog_create

```py
class BlogImageSerializer(serializers.ModelSerializer):
    class Meta:
        model = BlogImage
        fields = (
            'blog_id',
            'img_field',
            'img_thumbnail',
        )


class BlogCreateSerializer(serializers.ModelSerializer):
    images = serializers.ListField(
        child=serializers.ImageField(allow_empty_file=True)
    )
    travel_reservation = serializers.PrimaryKeyRelatedField(
        queryset=Reservation.objects.all(),
        read_only=False,
        validators=[UniqueValidator(queryset=Blog.objects.all())]
    )

    class Meta:
        model = Blog
        fields = (
            'travel_reservation',
            'title',
            'contents',
            'score',
            'images',
        )

    def create(self, validate_data, **kwargs):
        blog = Blog.objects.create(
            travel_reservation=validate_data['travel_reservation'],
            title=validate_data['title'],
            contents=validate_data['contents'],
            score=validate_data['score'],
        )

        for item in self.validated_data['images']:
            blog.images.create(
                blog_id=blog,
                img_field=item
            )
            blog.save()
        return blog


class BlogListSerializer(serializers.ModelSerializer):
    images = BlogImageSerializer(many=True)

    class Meta:
        model = Blog
        fields = (
            'pk',
            'travel_reservation',
            'title',
            'contents',
            'score',
            'images',
        )

```

- blog/apis/blog_create.py

```py
class BlogView(APIView):
    permission_classes = (
        permissions.IsAuthenticated,
    )

    def get(self, request):
        """
        자신이 쓴 상품 후기 리스트 출력
        :param request:
        :return:
        """
        blog = Blog.objects.all().filter(travel_Reservation__member=request.user)
        serializer = BlogSerializer(blog, many=True)
        return Response(serializer.data, status=status.HTTP_200_OK)

    # blog pk 값 받아서 수정 ,삭제 , 생성
    def post(self, request):
        """
        예약한 상품에 대해서 후기 작성
        :param request:
        :return:
        """
        serializer = BlogCreateSerializer(data=request.data)
        print(serializer)

        if serializer.is_valid(raise_exception=True):
            print(serializer.validated_data)
            blog = serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)

        else:
            return Response(status=status.HTTP_400_BAD_REQUEST)
```

### GenericAPIView
- blog/serialize/blog_create

```py

#  해당 예약이 회원 예약리스트에 없는 경우
class ReservationDoesNotExists(APIException):
    status_code = 400
    default_detail = '해당 예약 번호가 올바르지 않습니다.'
    default_code = 'reservation_DoesNotExists'

class BlogImageSerializer(serializers.ModelSerializer):
    class Meta:
        model = BlogImage
        fields = (
            'blog_id',
            'img_field',
            'img_thumbnail',
        )

class BlogListSerializer(serializers.ModelSerializer):
    images = BlogImageSerializer(many=True)

    class Meta:
        model = Blog
        fields = (
            'pk',
            'travel_reservation',
            'title',
            'contents',
            'score',
            'images',
        )

class BlogCreateSerializer(serializers.ModelSerializer):
    images = serializers.ListField(
        child=serializers.ImageField(allow_empty_file=True), write_only=True
    )

    travel_reservation = serializers.PrimaryKeyRelatedField(
        queryset=Reservation.objects.all(),
        read_only=False,
        validators=[UniqueValidator(queryset=Blog.objects.all())]
    )
    title = serializers.CharField(required=True)
    contents = serializers.CharField(required=True)
    score = serializers.CharField(required=True)

    class Meta:
        model = Blog
        fields = (
            'travel_reservation',
            'title',
            'contents',
            'score',
            'images',
        )

    def to_representation(self, instance):
        """
        해당 인스턴스를 BlogListSerializer로 변경해서 리턴
        :param instance:
        :return:
        """
        data = BlogListSerializer(instance).data
        return data

    def validate_travel_reservation(self, travel_reservation):
        """
        해당 예약 번호가 현재 유저의 예약 리스트에 있는지 검사
        1. 없는 경우 예외 처리
        2. 있으면 그대로 travel_reservation 리턴
        :param travel_reservation:
        :return:
        """
        user_reservation = Reservation.objects.filter(member=self.context['request'].user)
        if travel_reservation in user_reservation:
            return travel_reservation
        else:
            raise ReservationDoesNotExists

    def create(self, validate_data, **kwargs):
        blog = Blog.objects.create(
            travel_reservation=validate_data['travel_reservation'],
            title=validate_data['title'],
            contents=validate_data['contents'],
            score=validate_data['score'],
        )

        for item in self.validated_data['images']:
            blog.images.create(
                blog_id=blog,
                img_field=item
            )
            blog.save()
        return blog
```

- blog/apis/blog_create

```py
class BlogListCreateView(generics.ListCreateAPIView):
    permission_classes = (
        permissions.IsAuthenticated,
    )
    serializer_class = BlogCreateSerializer

    def get_serializer_context(self):
        """
        해당 요청에 대한 유저 정보를 넘겨 주기 위해서 사용
        :return:
        """
        return {'request': self.request}

    def get_queryset(self):
        queryset = Blog.objects.select_related('travel_reservation').filter(
            travel_reservation__member=self.request.user)
        return queryset
```

- `to_representation()`: 읽기 작업시 직렬화를 지원할 때 사용
- `to_internal_value()` : 쓰기 작업시 역직렬화를 지원할때 사용
- 정리 : 두개의 serialize를 사용하여 데이터를 쓸때와 읽을때의 serialize를 두개로 나눠서 사용
