---
layout: post
title: "[HTML] Table"
date: 2018-12-03
tags: [HTML]
comments: true
---

테이블은 데이터를 나타낼 때 이외의 다른 용도로는 절대 사용하면 안된다!!

## 테이블의 기본 구조

```html
<table>
  <caption>두 학생의 성적 비교</caption>
  <tr>
    <th>이름</th>
    <th>나이</th>
    <th>점수</th>
  </tr>
  <tr>
    <td>철수</td>
    <td>23</td>
    <td>70</td>
  </tr>
  <tr>
    <td>영희</td>
    <td>23</td>
    <td>70</td>
  </tr>
</table>
```

### colspan

셀 병합시(가로) 사용하는 것

`<td colspan="2">c</td>`

위 예시는 가로 두 칸을 병합하는 것을 의미

### rowspan

행을 병합시 사용하는 것

`<td rowspan="3">a</td>`

위 예시는 세로 세 칸을 병합하는 것을 의미

## 행의 구조화

```
<table>
  <thead>
    <tr>
      <th>name</th>
      <th>age</th>
      <th>male/female</th>
      <th>score</th>
    </tr>
  </thead>
  <tbody>

  </tbody>
  <tfoot>

  </tfoot>
</table>
```
