---
layout: post
title: Python 모듈과 패키지
author: Leesoo
layout: post
---

## 모듈과 패키지

### module
python file은 각각 하나의 모듈로 취급된다.

### import module
기본 게임 모듈에서 부가적인 기능을 불러와서 사용하는 형태

module/shop.py
```python
def buy_item():
    print('Buy item!')

buy_item()
```

module/game.py
```python
def play_game():
    print('Play game!')

play_game()
```

module/lol.py
```python
import game
import shop

print('= Turn on game =')
game.play_game()
shop.buy_item()
```

### __name__변수

```module/lol.py```가 실행될 때, game과 shop module이 import되는 순간 실행되는 것을 막기 위해
각 모듈 이름을 확인할 수 있는 전역변수 __name__을 이용한다.

module/shop.py
```python
def buy_item():
    print('Buy item!')

if __name__ == '__main__':
    buy_item()
```

module/game.py
```python
def play_game():
    print('Play game!')

if __name__ == '__main__':
    play_game()
```
그리고 ```lol.py```를 실행해본다.

### package

패키지란 모듈들을 모아두는 폴더 <br>
패키지로 사용할 폴더에 ```__init__.py```파일을 넣어주면 패키지로 취급된다. <br>
```shop.py```와 ```game.py```를 ```functions``` 패키지에 넣을 경우

```terminal
├── functions
│   ├── __init__.py
│   ├── game.py
│   └── shop.py
└── lol.py
```
패키지 import : <br>
1. from functions import game, shop
2. import functions -> 사용시 functions.game, functions.shop

#### *, __all__
1. *: 해당 모듈의 모든 식별자들을 불러온다. <br>
2. __all__ : 각 모듈에서 자신이 import될 때 불러올 목록을 지정하고자 할 때

example
```Python
>>> from game.sound import *
>>> echo.echo_test()
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
NameError: name 'echo' is not defined
```

__all__의 용도

```python
# C:/doit/game/sound/__init__.py
__all__ = ['echo']
```

특정 디렉터리의 모듈을 ```*``` 을 이용하여 import할 때에는 위와 같이 해당 디렉터리의 ```__init__.py``` 파일에
```__all__```이라는 변수를 설정하고 import할 수 있는 모듈을 정의해 주어야 한다. <br>

위와 같이 ```__init__.py```를 변경한 후 위 예제를 수행하면 원하던 결과가 출력이 된다.

```python
>>> from game.sound import *
>>> echo.echo_test()
echo
```
