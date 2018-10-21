---
layout: post
title: "Spring Boot : WebMvcConfigurer 사용하여 컨버터하기 (2)"
date: 2018-10-21
tags: [Spring Boot]
comments: true
---

이전글
- https://leesoo7595.github.io/2018/10/21/Spring_ConfigurableWebBindingInitializer/

이전글과 이어서 같은 결과를 내기 위해 이번엔 `@WebMvcConfigurer`를 사용하여 String으로 받아야하는 값을 Library로 받을 수 있도록 컨버팅해보겠다. <br>
<br>
<br>

`Library` 클래스와 `LibraryController` 클래스는 그대로!

<br>

## Application 클래스

```java
@SpringBootApplication
public class Application {
   public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

```

## MyWebConfig 클래스

```java
@Configuration
public class MyWebConfig implements WebMvcConfigurer {
     // addConverter에 새로만든 LibraryConverter 추가
    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new LibraryConverter());
    }
}

```

위 `WebMvcConfigurer` 클래스를 `implements`하여 `addFormatters`를 `LibraryConverter`를 추가하는 코드로 커스터마이징하여 <br>
String 값으로 받아야하는 아이를 `MyWebConfig`를 통해 Library 값 형태로 컨버팅시켜주게끔 한다. <br>

### Result

```java
status 200
```
