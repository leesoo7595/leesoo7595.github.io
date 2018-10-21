---
layout: post
title: "Spring Boot : Component로 Bean 등록 방법으로 컨버팅하기 (3)"
date: 2018-10-21
tags: [Spring Boot]
comments: true
---

이전글
- https://leesoo7595.github.io/2018/10/21/Spring_ConfigurableWebBindingInitializer/
- https://leesoo7595.github.io/2018/10/21/Spring_WebMvcConfigurer_addFormatters_customizing/

위 방법보다 더욱 간단한 방법인 `@Component`를 통해 `LibraryConverter` 클래스를 빈으로 등록하여 곧바로 컨버팅하게 한다. <br>
<br>

**매우 간단ㅎㅎㅎㅎ** <br>
<br>

## LibraryConverter 클래스

```java
@Component
public class  LibraryConverter implements Converter<String, Library> {
     @Override
    public Library convert(String id) {
         Library library = new Library();
         library.setId(Integer.parseInt(id));
         return null;
     }
 }
```

이렇게만 해주면 사실 끝!! 앞의 방법들은....안쓸듯 우워 <br>
