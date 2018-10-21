---
layout: post
title: "Spring Boot : ConfigurableWebBindingInitializer 사용하여 컨버터하기 (1)"
date: 2018-10-21
tags: [Spring Boot]
comments: true
---

Library 클래스를 만들어서 Integer값으로 도서관 id를 받아야하는 `Controller GetMapping`을 만든다. <br>

Library class

```java
 public class Library {
     private Integer id;
     private String admin;
     private Date borrowed;
     private Date returned;
     private String name;
     public Integer getId() {
        return id;
    }
     public void setId(Integer id) {
        this.id = id;
    }
     public String getAdmin() {
        return admin;
    }
     public void setAdmin(String admin) {
        this.admin = admin;
    }
     public Date getBorrowed() {
        return borrowed;
    }
     public void setBorrowed(Date borrowed) {
        this.borrowed = borrowed;
    }
     public Date getReturned() {
        return returned;
    }
     public void setReturned(Date returned) {
        this.returned = returned;
    }
     public String getName() {
        return name;
    }
     public void setName(String name) {
        this.name = name;
    }
}
```

LibraryController class

```java
 @RestController
public class LibraryController {
     @GetMapping("/bs/{id}")
     public Library visitLibrary(@PathVariable("id") Library library) {
         return library;
     }
}
```

LibraryConverter Class

```java
// String을 Library 형태로 컨버터시켜준다.
public class LibraryConverter implements Converter<String, Library> {
     @Override
    public Library convert(String id) {
        Library library = new Library();
        library.setId(Integer.parseInt(id));
        return null;
    }
}
```

Application Class

```java
@SpringBootApplication
public class Application {
     @Bean
    public ConfigurableWebBindingInitializer initializer() {
        ConfigurableWebBindingInitializer initializer = new ConfigurableWebBindingInitializer();
        ConfigurableConversionService conversionService = new FormattingConversionService();
        // conversionService에 새로만든 LibraryConverter 추가
        conversionService.addConverter(new LibraryConverter());
        // initializer에 ConversionService를 셋팅하기
        initializer.setConversionService(conversionService);
        return initializer;
    }
     public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
```
