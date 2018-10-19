---
layout: post
title: "Spring Boot : @RestController의 비밀!"
date: 2018-10-19
tags: [Spring Boot]
comments: true
---

# @Controller와 @RestController의 차이

`@Controller`를 썼을 때, 예를 들어 어떤 `String`값을 리턴해서 보여주고 싶다면,

```Java
@Controller
public class UserController {

    @RequestMapping("/")
    public @RequestBody String index() {
        return "Hello Spring MVC";
    }
}

```

이런식으로 `@RequestBody`를 리턴타입 앞에 명시해주어야한다. <br>
<br>
반면, `@RestController`는

 ```Java
 @RestController
 public class UserController {

     @RequestMapping("/")
     public String index() {
         return "Hello Spring MVC";
     }
 }

 ```

 이런식으로 `@RequestBody`를 생략해서 쓸 수 있다. 그래서 `@RestController`를 쓰는 것~~
 <br>

 더 추가할 것이 있으면 추가하겠음!! 끗
