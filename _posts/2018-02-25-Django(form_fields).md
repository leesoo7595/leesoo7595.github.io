---
layout: post
title: "Django- Form_fields"
date: 2018-02-25 00:00:00
img:
tags: [Django]
---

- 질문 Form.use_required_attribute
  - `form set` form을 여러개 나오게 하는 것.
이 경우 빈 imput 이 있을 경우 HTML validation 발생한다. 그러나 이걸 False로하면 HTML에서
요효성 검사는 하지 않음. 여러개의 form을 사용할 경우 JavaScript를 사용하는 게 낫다.

---

## Form Fields
-  Form 클래스를 만들 때, 가장 중요한 것은 **필드 폼** 을 잘 정의 하는 것이다.

### Field.clean(value)
- 지정된 필드의 값에 맞지 않으면, `django.forms.ValidationError 예외`를 발생시키거나 지정된 값을 출력

```py

>> f = forms.EmailField()
>> f.clean('hello@gmail.com')
'hello@gamil.com'

>> f.clean('hello')
ValidationError: ['Enter a valid email address.']

```

## Core filed arguments
### Field.required
- 각 Field는 default 값으로 값이 필요함 그래서 `None` 또는 `""` 중 하나를 빈 값으로 전달 시 `clean()`은 `ValidationError 예외`를 발생시킴


```py
# required=Flase를 주면, '' / None이 전달 되도 오류가 발생되지 않음
>>> f = forms.CharField(required=False)
>>> f.clean('foo')
'foo'
>>> f.clean('')
''
```

### Field.label
- 필드가 표시 될 때, 보여지는 부분

```py
>>> from django import forms
>>> class CommentForm(forms.Form):
...     name = forms.CharField(label='Your name')
>>> f = CommentForm(auto_id=False)
>>> print(f)

<tr><th>Your name:</th><td><input type="text" name="name" required /></td></tr>
```

### Field.label_suffix

```py
>>> class ContactForm(forms.Form):
...     age = forms.IntegerField()
...     nationality = forms.CharField()
...     captcha_answer = forms.IntegerField(label='2 + 2', label_suffix=' =')
>>> f = ContactForm(label_suffix='?')
# as_p() : <p>태그로 랜더링
>>> print(f.as_p())

<p><label for="id_age">Age?</label> <input id="id_age" name="age" type="number" required /></p>
<p><label for="id_nationality">Nationality?</label> <input id="id_nationality" name="nationality" type="text" required /></p>
<p><label for="id_captcha_answer">2 + 2 =</label> <input id="id_captcha_answer" name="captcha_answer" type="number" required /></p>

```

### Field.initial
- 언바운드 폼일 때, 초기 `value` 값을 설정 할 수 있음
- `initial` 값을 딕셔너리 형태로 보내는 경우, 유효성 검사에서 에러가 발생할 수도 있다.
- 바인드 된 데이터 혹은 특정 필드 값이 없을 경우, 유효성 검사에서 대체데이터를 사용하지 않는다.
- 상수 대신에 함수를 전달 할 수 있다. (ex `datetime.date.today`)

### Field.widget
`widget`은 HTML입력 요소를 표현한 것으로서,  HTML 렌더링을 GET/POST 사전에서 데이터를 추출

```py

class SignupForm(forms.Form):

    # widget은 CharField를 사용한다 했을 때 좀더 상세한 조건 넣어주기 위해
    password = forms.CharField(
        label="비밀번호",
        widget=forms.PasswordInput,
    )
```

### Field.help_text
- 각 속성들을 설명하는데 사용함
- 되도록이면 각 속성들에 넣어주는게 좋음

### Field.error_message
- 기본 오류 메시지를 오버라이딩해서 변경할 수 있다.

```py
# required가 빈값일 경우, 설정된 오류를 출력
>>> name = forms.CharField(error_messages={'required': 'Please enter your name'})
>>> name.clean('')
Traceback (most recent call last):
  ...
ValidationError: ['Please enter your name']
```

### Field.validators

### Field.localize
- 각 사용자에 맞춰서 시간이랑 이런거 맞게 표시 해주는 함수

### Field.disabled
(이거 뭔지.. 감이 안옴)
- `boolean` 요소이며, `True`로 설정된 경우, 비활성화된 HTML요소들을 사용자가 사용할 수 없게 한다.

---

## Checking if the field data has changed¶
### Field.has_changed()
- `value` 가 바뀌는 것에 따라서 `True` 혹은 `False` 를 출력

---

## Built-in Field classes
