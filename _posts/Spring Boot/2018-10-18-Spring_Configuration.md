---
layout: post
title: "Spring Boot : Spring MVC - Auto-configuration"
date: 2018-10-18
tags: [Spring Boot]
comments: true
---

# Spring MVC - Auto-configuration 하기

`Config`라는 `Java class`를 만들고, `@Configuration` 어노테이션을 붙인다. <br>
<br>

보통 `WebMvcConfigurer` 인터페이스를 `implements`하여 `configure`한다. <br>

`HandlerMapping, HandlerAdapter, ExceptionHandler`를 커스터마이징하고 싶을 떄, `WebMvcRegistrations` 인터페이스를 사용하기 <br>
<br>

**@EnableWebMvc** 어노테이션을 붙이는 것은 완전히 Spring MVC를 혼자서 커스터마이징 할 때 사용하는 것!!
<br>

자세한 내용 추후 추가 <br>
일단 기본적인 것만! 끗
