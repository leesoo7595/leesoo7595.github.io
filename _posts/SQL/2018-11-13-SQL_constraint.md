---
layout: post
title: "SQL : Constraint - 제약"
date: 2018-11-13
tags: [SQL]
comments: true
---

# 27강 제약

## 제약

테이블에 제약을 설정함으로써 저장될 데이터를 제한할 수 있다.

```sql
CREATE TABLE sample (
	a INTEGER NOT NULL,
	b INTEGER NOT NULL UNIQUE,
	c VARCHAR(30)
);
```

위처럼 열에 대해 정의하는 제약을 **열 제약**이라고 한다.

```sql
CREATE TABLE sample (
	no INTEGER NOT NULL,
	sub_no INTEGER NOT NULL,
	name VARCHAR(30),
	PRIMARY KEY (no, sub_no)
);
```

위처럼 한 개의 제약으로 복수의 열에 제약을 설명하는 경우를 **테이블 제약**이라 부른다.

제약에는 이름을 붙일 수 있다. 이는 `CONSTRAINT` 키워드를 사용해서 지정한다.

```sql
CREATE TABLE sample (
	no INTEGER NOT NULL,
	sub_no INTEGER NOT NULL,
	name VARCHAR(30),
	CONSTRAINT pkey_sample PRIMARY KEY (no, sub_no)
);
```

### 제약 추가

1. 열 제약 추가

열 제약을 추가할 경우 `ALTER TABLE`로 열 정의를 변경할 수 있다. 기존 테이블을 변경할 경우에는 제약을 위반하는 데이터가 있는지 먼저 검사한다. 제약을 위반하는 데이터가 있을 경우 에러가 발생한다.

```
ALTER TABLE sample MODIFY name VARCHAR(30) NOT NULL;
```

위는 `NOT NULL` 제약을 추가하는 예이다.

2. 테이블 제약 추가

테이블 제약은 `ALTER TABLE`의 `ADD` 하부명령으로 추가할 수 있다. 열 제약을 추가할 때와 마찬가지로 기존의 행을 검사해 추가할 제약을 위반하는 데이터가 있으면 에러가 발생한다.

```
ALTER TABLE sample ADD CONSTRAINT pkey_sample PRIMARY KEY(no);
```

### 제약 삭제

테이블 제약은 나중에 삭제할 수 있다.

1. 열 제약 삭제

열 제약의 경우, 제약을 추가할 때와 동일하게 열 정의를 변경한다.

```
ALTER TABLE sample MODIFY name VARCHAR(30);
```

위는 앞서 제약을 추가한 name열의 `NOT NULL` 제약을 삭제하는 예이다.

2. 테이블 제약 삭제

테이블 제약은 `ALTER TABLE`의 `DROP` 하부명령으로 삭제할 수 있다. 삭제할 때는 제약명을 지정한다.

```
ALTER TABLE sample DROP CONSTRAINT pkey_sample;
```

```
ALTER TABLE sample DROP PRIMARY KEY;
```

## 기본키

```
CREATE TABLE sample (
	p INTEGER NOT NULL,
	a VARCHAR(30),
	CONSTRAINT pkey_sample PRIMARY KEY(p)
);
```

열 p가 sample 테이블의 기본키이다. 기본키로 지정할 열은 NOT NULL 제약이 설정되어 있어야 한다.

기본키는 대량의 데이터에서 원하는 데이터를 찾아낼 때 키가 되는 요소를 지정해 검색하는 것이다.

기본키로 설정된 열이 중복하는 데이터 값을 가지면 제약에 위반된다. 즉, 기본키로 지정해 유일한 값을 가지도록 하는 구조를 **기본키 제약**이라고 한다.

- 복수열로 기본키 구성

기본키를 구성하는 열은 복수여도 상관없다.
