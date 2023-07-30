---
layout: post
title: "네트워크 - HTTP 캐시"
date: 2023-07-30
tags: [Network]
comments: true
---

# HTTP Cache 헤더 옵션

데이터 요청/응답을 할때, 사이에 캐시를 거쳐서 가는 식으로 캐시를 확인한다. 해당 요청에 대한 데이터가 캐시에 이미 있으면, 서버에 요청을 보내지않고 캐시를 가지고 온다. 그렇게 비용을 아낄 수 있음

**Cache Hit -> 적중**
**Cache Miss -> 부적중**

Cache가 너무 오래된 경우, 데이터베이스에서 업데이트된 부분이 반영이 바로 안될 수 있음. 적절히 사용해야함. 그래서 캐시가 너무 오래된 경우, 캐시를 얼마나 오래 저장할 것인지 설정할 수 있다.

## Cache-Control: Max-Age (선호) >>> Expires

**캐시가 너무 오래된 경우**

- Cache가 오래됐는데,(Max-Age, Expires가 다됨) 서버에서 물어보니 변경이 없는 경우
  - `304 Not Modified`
  - 캐시를 그대로 쓰기
  - Max-Age, Expires 업데이트
- Cache가 오래됐는데, 서버에 물어보니 바뀐 경우
  - `200 ok`
- Cache가 오래됐는데, 서버 체크 없이 그냥 캐시를 주는 경우
  - `must-revalidate` 쓰면 max-age 넘어갔을 때 서버체크하도록해줌
  - State-While-Revalidate
    - 일단 오래된거 주고, 뒤에서 서버체크 후 업데이트됐으면 업데이트하기
    - 캐시를 이미 쓰는 것이기 때문에 부정확하다
    - swr이 하는 일..
  - State-If-Error
    - 서버가 주지말라고 에러를 뱉어도 오래된 캐시를 준다.
    - 매우 특수해서 쓸일이 있나..

## `Cache-Control`

서버쪽에서 캐시를 얼마나 가지고 있어야하는지 알려주는 헤더, 브라우저에서 알고 있다. (저장을 얼마나 해놓을지!)

요청헤더, 응답헤더로 다르게 쓰일 수도 있지만, 주로 응답헤더에서 쓰는 경우가 많다.

### no-store

캐시를 저장하지 않는 것, 무조건 네트워크 요청

### no-cache

캐시를 저장하고, 캐시의 유효기간을 무시하고 신선도를 체크하란 뜻, 서버에게 항상 물어봄.

서버에서 `304 Not Modified` 나오면 캐시를 사용

### must-revalidate

유통기한(max-age) 지나면 서버에 신선도를 검사하기

### public

캐시가 공유됨, Authorization이 있으면 공유되도록 public 설정해도 private로 설정됨..

쓸일이 있나?

## Pragma

HTTP 1.0 버전에나 쓰였던 것

안씀..

## 캐시 신선도 체크 (Etag, Last-Modified)

실제 데이터가 마지막으로 변경된 시간(최종변경시각)도 중요하다. 데이터가 업데이트되었는지 안되었는지 체크하는데 중요함.

### Last-Modified

Last-Modified 헤더가 들어올때가 있다. Date 형태로 받아와짐

### ETag

`W/값` 형식으로 이루어져있다. ETag 자동생성기도 있음. 내용바뀔때마다 이전 값과 겹치지 않게 생성해줌.

캐시와 서버의 데이터가 같은지 확인하는 값

### If-Match, If-None-Match...

`If-Match: <etag_value>` 헤더를 추가해서 요청을 보낼 때, Etag를 적어서 보내주면, 서버의 Etag와 비교해서 똑같으면 캐시를 사용한다. 데이터가 바뀌지 않았으니 서버에게 더 물어볼 필요가 없다고 인지.

`If-None-Match: <etag_value>` 헤더를 추가해서 요청을 보내면 Etag를 서버와 비교해서 캐시에 있는게 똑같으면 서버에서 가져온다. If-Match와 반대

`If-Modified-Since` 헤더는 `Last-Modified` 헤더와 짝임. 둘을 비교해서 `If-Modified-Since`의 날짜를 `Last-Modified` 헤더의 날짜와 비교해서 Last-Modified 값이 더 옛날이면 서버에서 가지고 온다.
