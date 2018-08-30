---
layout: post
title: "[Django] Modeling - Foreignkey & ManyToMany"
img:
tags: [Django]
---


해당 글은 장고 문서를 참고하여 작성했습니다. <br>
참고 : https://docs.djangoproject.com

## ManyToMany 필드

### 단순 ManyToMany 관계

`manytomany`인 다대다 관계의 예로를 `pizza`와 `topping`을 들 수 있다.
다음은 단순 다대다 관계를 `many_to_many`패키지의 `models.py`에서 모델링한 것이다.
```python
from django.db import models


class Topping(models.Model):
  name = models.CharField(max_length=50)

  def __str__(self):
    return self.name


class Pizza(models.Model):
  name = models.CharField(max_length=50)
  toppings = models.ManyToManyField(
    Topping,
    related_name='pizzas',
  )

  def __str__(self):
    return self.name
```
그리고 터미널창에 `./manage.py makemigrations`, `./manage.py migrate`해보자 <br>
정상으로 `migrate`되었다면,  `./manage.py shell`로 `ipython`을 실행시켜서 동작시켜보자 <br>

```terminal
In [1]: from models.many_to_many.models import Topping, Pizza

In [2]: Pizza.objects.create(name='페퍼로니')
Out[2]: <Pizza: 페퍼로니>

In [3]: Pizza.objects.create(name='콤비네이션')
Out[3]: <Pizza: 콤비네이션>

In [4]: Pizza.objects.create(name='치즈')
Out[4]: <Pizza: 치즈>

In [5]: Pizza.objects.create(name='슈퍼슈프림')
Out[5]: <Pizza: 슈퍼슈프림>

In [6]: 페퍼로니, 콤비네이션, 치즈, 슈퍼슈프림 = Pizza.objects.all()

In [7]: 페퍼로니
Out[7]: <Pizza: 페퍼로니>

...

In [10]: Topping.objects.create(name='햄')
Out[10]: <Topping: 햄>

In [11]: Topping.objects.create(name='치즈')
Out[11]: <Topping: 치즈>

In [12]: Topping.objects.create(name='페퍼로니햄')
Out[12]: <Topping: 페퍼로니햄>

In [13]: Topping.objects.create(name='피망')
Out[13]: <Topping: 피망>

In [14]: Topping.objects.create(name='올리브')
Out[14]: <Topping: 올리브>

In [15]: 햄, 피자치즈, 페퍼로니햄, 피망, 올리브 = Topping.objects.all()

In [16]: 햄
Out[16]: <Topping: 햄>
```
위처럼 Topping과 Pizza 종류를 만들고 변수명을 지정해주었다.
그리고 `ManyToManyField`로 지정해준 변수 `toppings`를 터미널창에서 사용해보자.

```termial
In [37]: 치즈.toppings
Out[37]: <django.db.models.fields.related_descriptors.create_forward_many_to_many_manager.<locals>.ManyRelatedManager at 0x7fe7540d2898>

In [38]: 치즈.toppings.all()
Out[38]: <QuerySet []>

In [39]: 치즈.toppings.add(피자치즈)

In [40]: 치즈.toppings.all()
Out[40]: <QuerySet [<Topping: 치즈>]>
```

피자의 `toppings`는 피자와 다대다 관계로 이루어진 토핑들을 의미한다.
반대로 토핑과 연결된 피자를 알아보려면 다음과 같다.

```terminal
In [1]: from models.many_to_many.models import Topping, Pizza

In [2]: t = Topping.objects.first()

In [3]: t
Out[3]: <Topping: 햄>

In [4]: t.pizzas.all()
Out[4]: <QuerySet [<Pizza: 슈퍼슈프림>]>

In [5]: t.pizzas
Out[5]: <django.db.models.fields.related_descriptors.create_forward_many_to_many_manager.<locals>.ManyRelatedManager at 0x7eff89073fd0>
```

토핑의 'pizzas'는 토핑과 다대다 관계로 이루어진 피자들을 의미한다. 이는 `Pizza 모델클래스`에서 `related_name`으로 정의할 수 있다.

그렇게 토핑이 있는 피자의 관계를 피자 클래스에서 related_name='pizzas'로 스스로를 가리키는 말을 정의할 수 있다.

### 중간모델이 있는 ManyToMany 관계

중간모델이 있는 ManyToMany 관계를 예를 들면 사람과 그룹으로 멤버를 이루는 관계를 들 수 있다. 이는 `Person` 클래스와 `Group` 클래스로 나타낼 수 있으며, 중간모델으로 `Membership` 클래스를 두어 어떤 사람이 그룹에 가입할 때 `가입한 날짜`나 `가입한 이유` 등을 중간모델에 입력할 수 있도록 한다.

```python
class Person(models.Model):
    name = models.CharField(max_length=128)

    def __str__(self):
        return self.name


class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(
        Person,
        through='Membership',
    )

    def __str__(self):
        return self.name


class Membership(models.Model):
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    date_joined = models.DateField()
    invite_reason = models.CharField(max_length=64)

    def __str__(self):
        return '{person} - {group} ({date})'.format(
            person=self.person.name,
            group=self.group.name,
            date=self.date_joined,
        )
```

`Membership` 클래스에서는 외래키(`ForeignKey`)를 명시적으로 두 개인 `Person`과 `Group` 클래스를 두게 한다. 그리고 추가 정보로 입력하게 될 `date_joined`과 `invite_reason`을 두었다.

중간모델은 지켜야 할 항목이 있다.
1. ManyToMany 필드로 두 테이블의 관계를 중간모델에서 나타낼 때, 무조건 한 테이블에서 하나의 `ForeignKey`로 나타내거나 두 개 이상일 경우 `related_name`으로 한 테이블에서 어떤 것을 가리키는지 명시적으로 지정해주어야 할 것이다.
2. 중간모델에 보통 2개의 `ForeignKey`를 지정할 수 있다. 3개 이상의 `ForeignKey` 필드를 선언할 경우에는 `through_fields` 옵션을 설정해주어야한다.
```python
class Person(models.Model):
    name = models.CharField(max_length=128)

    def __str__(self):
        return self.name


class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(
        Person,
        through='Membership',
        through_fields=('person', 'group'),
    )

    def __str__(self):
        return self.name


class Membership(models.Model):
    # Person Class에서 person이 필수적으로 입력하는 정보로 명시
    person = models.ForeignKey(
      Person,
      related_name='memberships',
      on_delete=models.CASCADE,
    )
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    # Person Class에서 추천인의 정보를 명시하고 싶을 때 위와 같은 클래스를 참조하므로 위와 같이 related_name을 지정해주어야한다.
    recommender = models.ForeignKey(
      Person,
      related_name='memberships_by_recommender',
      on_delete=models.SET_NULL,
      blank=True,
      null=True,
    )
    date_joined = models.DateField()
    invite_reason = models.CharField(max_length=64)

    def __str__(self):
        return '{person} - {group} ({date})'.format(
            person=self.person.name,
            group=self.group.name,
            date=self.date_joined,
        )
```
