---
layout: post
title: "Django_Project(sign_up)"
date: 2018-03-29 00:00:00
img:
tags: [Django_Project]
---

### 구현하고 싶은 기능
1. 회원가입 할때, username을 email로 받는 것
2. 페이스북 로그인
3. 모든 기능은 APIView로 구현

----

## Sign_up
### UserModel 만들기
- authentication : 회원이 있는지 없는지 <- authorized : 유저가 가진 권한
- AbstractUser : 장고 기본 사용자의 전체 구현을 추상 모델로 제공한다.

#### email filed = charfield
- 기존의 `email` 필드를 사용하지 않고 `CharField`를 사용했다. 왜냐하면 이메일로만 Usermodel을 구현하면 Oath 로그인 방식에서 문제가 생길 수 있다. 예로 페이스북 로그인의 경우 전화번호로도 가입을 할 수 있는데 이메일로만 만들 경우 문제 된다.
- email validator 구현해야 한다.(backends 단에서 해야할듯)
- 이메일 필드가 `CharField`기 때문에, 데이터베이스에 유저 객체를 저장하기 전에 `pre_signal`로 이메일유효성 검사를 실시

*올바르지 않은 주소의 경우 return 부분에 대해 다시 한번 고민해야 할것*
*AbstractBaseUser 직접 구현해보려고 한 결과 정말 만들게 많다..... 그래서 강사님이 조언해주셨던 것 처럼 AbstractUser 을 상속 받아서 사용했다.*

#### emai, username field 모두 사용
- username,email(), password, img_profile, phonenumber, is_facebook_user
- username은 유효성이 검증된 email을 같은 데이터로 저장

### APIView구현
https://iheanyi.com/journal/user-registration-authentication-with-django-django-rest-framework-react-and-redux/

#### 해야 할 것
- img 파일을 IOS쪽에서 어떤식으로 넘길것인지(파일로 아니면 url 주소로 ) -> media 파일 주소로 넘어온다.
- password 유효성 검사에 대해서도 알아볼것 (길이, 문자, 회원관련 정보 ) ->

#### 하면서 막혔던 부분
[validate_password 참고 사이트](https://gist.github.com/leafsummer/f4d67b58a4cc77174c31935d7e299c9e)
- 회원이 이미지를 업로드 하지 않는 경우, 디폴트 이미지 처리 : 디폴트 이미지를 이미지 필드에 지정할 때, `upload_to`가 media이기 때문에 논리적으로 맞지 않다. 또한 이미지를 그렇게 저장했을 경우, 회원이 접속하면 무조건 서버에 접속을 해야하는 네트워크상 비효율적인 부분이 발생. 그래서 이미지 필드값을 `null=True`로 지정하고 클라이언트 측에서 null인 경우에 가지고 있는 디폴트 이미지를 출력하는 것으로 합의
- 비밀번호 두개 일치하는 것과 유효성 검사부분에서 많은 시간을 할애했다.
  - Serializer에서 `write_only = True`는 필드를 업데이트하거나 생성할 때는 사용하지만, serialize 할때는 사용하지 않는다.
-> password2도 받으나 직렬화 할때 사용하지 않을때 이용
  - deserialization 할 때, 필드가 제공되지 않으면 일반적으로 오류가 발생한다. 이 경우 `required = False`를 주면, 직렬화 할때 이 속성은 생략 될 수 있다. -> post로 email만 받을때 사용
  - 유저 객체를 만들어서 검사를 하지 않기 때문에, username과 비슷한 비밀번호 검사는 불가

-----

## 일반 Login
### APIView구현
[토큰로그인방식_참고자료](http://sanghaklee.tistory.com/47)
- get방식으로 username, password 를 받고 토근 생성 그리고 토큰이랑 유저 정보 제공
- 로그인 성공시 `status = 200`, 로그인 실패시 `stauts = 401`

---

## 페이스북 로그인
페이스북 로그인 access_token -> Serializer 에서 user 유효성 검사 -> 이때 페이스북 아이디를 가지고 유저를 생성 -> 토큰을 생성 -> 토큰과 유저 반환
