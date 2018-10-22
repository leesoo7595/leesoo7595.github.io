---
layout: post
title: "Spring Boot : Test - Mocking vs SpyBean"
date: 2018-10-22
tags: [Spring Boot]
comments: true
---

# Mocking vs SpyBean

`MockBean`은 기본값이 null값이다.
<br>
<br>
반면, `SpyBean`은 기본값이 `SpringApplication.class`에 지정되어있는 Bean의 값이다.
<br>
<br>

테스트할 때, 이것만 숙지하고 가려서 필요한 것을 쓰면 될 것이다. 끝
