---
layout: post
title: "Python_class 실습"
date: 2018-05-25 00:00:00
img:
tags: [Python]
categories:
- Python
---

## 실습 문제
```console
도서 관리 프로그램
    Library, Book, User클래스 구현
        프로그램 시작시 도서 5권 정도를 가진 상태로 시작

    Library
        attrs
            name: 도서관명
            book_list: 도서 목록 (Book인스턴스의 목록)
        methods
            add_book
            remove_book
        property
            info: 가지고 있는 도서 목록을 보기좋은 텍스트로 출력 (빌려간 도서는 출력 안해도 됨)

    Book
        attrs
            title: 제목
            location: 현재 자신이 어떤 Library 또는 User에게 있는지를 출력
        property
            is_borrowed: 대출되었는지 (location이 User인지 Library인지 확인)

    User
        attrs
            name: 이름
            book_list: 가지고 있는 도서 목록
        methods
            borrow_book(library, book_name): library로부터 book을 가져옴
            return_book(library, book_name): library에 book을 다시 건네줌
```

----

## class 정의
```python

class Book:
    books_title = ['python', 'java', 'swift', 'objectC', 'C++']

    def __init__(self, title, location, is_borrowed=False):
        """
        처음 5권의 책 생성
        books_title 이 빈 리스트인 경우 한권의 책만 생성
        """
        self.location = location
        self.title = title
        self.__is_borrowed = is_borrowed

        if self.books_title:
            self.make_books(location)

    def make_books(self, location):
        """
        책 제목을 통해 Book 인스턴스 생성 그리고 도서관에 책 추가
        """
        for book in self.books_title:
            self.books_title.remove(book)
            book = Book(book, location)
            location.new_book(book)

    @property
    def is_borrowed(self):
        """
        대출 했을 경우 True,
        아닌경우 False
        """
        return self.__is_borrowed

    @is_borrowed.setter
    def is_borrowed(self, new_status):
        self.__is_borrowed = new_status

class Library:
    def __init__(self, name, book_lists=list()):
        self.name = name
        self.book_lists = book_lists

    def new_book(self, book):
        """
        새로운 책 추가하는 함수
        :param book:
        :return:
        """
        self.book_lists.append(book)

    def add_book(self, title):
        """
        책이 반납되었을 경우 실행
        :param title:
        :return:
        """
        for book in self.book_lists:
            if book.title == title:
                book.is_borrowed = False

    def remove_book(self, title):
        """
        책이 대출되었을 경우 실행
        :param title:
        :return:
        """
        for book in self.book_lists:
            if book.title == title:
                book.is_borrowed = True

    @property
    def info(self):
        """
        가지고 있는 도서 목록 출력
        빌려간 도서는 출력 안함
        """
        return print('\n'.join(book.title for book in self.book_lists if not book.is_borrowed))


class User:
    def __init__(self, name, book_lists=list()):
        self.name = name
        self.book_lists = book_lists

    def borrow_book(self, library, book_name):
        library.remove_book(book_name)
        self.book_lists.append(book_name)

    def return_book(self, library, book_name):
        library.add_book(book_name)
        self.book_lists.remove(book_name)

    def __repr__(self):
        return f'{self.name}(이)가 빌린 책: ' + ','.join(self.book_lists)
```

----

## 실행결과
```python

>> fastcamp_library = Library('패스크캠퍼스도서관')
>> book1 = Book('hello world',fastcamp_library)
>> fastcamp_library.new_book(book1)
>> book2 = Book('안녕',fastcamp_library)
>> fastcamp_library.new_book(book2)
>> kahee = User('가희')
>> kahee.borrow_book(fastcamp_library,'안녕')
>> book2.is_borrowed
True
>> fastcamp_library.info
C++
objectC
swift
java
python
hello world
>> kahee
가희(이)가 빌린 책: 안녕
>> kahee.return_book(fastcamp_library,'안녕')
>> kahee.book_lists
[]
>> fastcamp_library.info
C++
objectC
swift
java
python
hello world
안녕
>> kahee.borrow_book(fastcamp_library,'python')
>> kahee
가희(이)가 빌린 책: python
>> fastcamp_library.info
C++
objectC
swift
java
hello world
안녕
```
