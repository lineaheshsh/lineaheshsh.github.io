---
layout: post
title: Spring Boot 2장
summary: 열심히 기록하자!
author: Lee Chang Ho
date: '2020-11-26 14:58:00'
category: 프로젝트
published: true
image:  스프링부트.png
tags:   Spring Boot, Spring, Gradle, JUnit
---

### 스프링 부트 2장(JUnit)  

###### 이번 화에서는 "스프링 부트"에서 "테스트 코드"를 작성하는 방법에 대해 기록한다.  
###### 책에서는 "테스트 코드"가 필요한 이유를 다음과 같이 설명한다.  

+ 단위 테스트는 개발단계 초기에 문제를 발견하게 도와줍니다.
+ 단위 테스트는 개발자가 나중에 코드를 리팩토링하거나 라이브러리 업그레이드 등에서 기존 기능이 올바르게 작동하는지 확인할 수 있습니다(예, 회귀 테스트).
+ 단위 테스트는 기능에 대한 불확실성을 감소시킬 수 있습니다.
+ 단위 테스트는 시스템에 대한 실제 문서를 제공합니다. 즉, 단위 테스트 자체가 문서로 사용할 수 있습니다.

###### 그럼 이제 테스트 코드를 작성하는 방법에 대해 살펴보도록 하자.  
###### 현재 패키지(com.zzangho.project.springboot) 하위에 web이라는 패키지를 추가로 만든다.  
###### 앞으로 컨트롤러와 관련된 클래스들은 모두 이 패키지에 담는다.  
###### 그리고 테스트해볼 컨트롤러를 만들어 보자. [New -> Java Class]를 선택하여 HomeController Class를 만든다. 그 뒤 아래와 같이 간단한 API 코드를 작성한다.

```java
package com.zzangho.project.springboot.web;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

###### 작성한 코드가 제대로 작동하는지 테스트를 해보자. WAS를 실행하지 않고, 테스트 코드로 검증해 본다. src/test/java 디렉토리에 앞에서 생성했던 패키지를 그대로 다시 생성해 본다.  
###### 그리고 테스트 코드를 작성할 클래스를 생성한다. 일반적으로는 테스트 클래스는 대상 클래스 이름에 Test를 붙인다. 그러므로 HomeControllerTest로 생성한다.
###### 생성된 클래스에 다음과 같은 테스트 코드를 추가하자.  

```java
package com.zzangho.project.springboot.web;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.is;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

/**
 * 테스트를 진행할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킴
 * 여기서는 SpringRunner라는 스프링 실행자를 사용
 * 스프링 부트 테스트와 JUnit 사이에 연결자 역할
 */
@RunWith(SpringRunner.class)
/**
 * 여러 어노테이션 중, Web(Spring MVC)에 집중할 수 있는 어노테이션
 * 선언할 경우 @Controller, @ControllerAdvice 등을 사용할 수 있다
 * 단, @Service, @Component, @Repository 등은 사용 불가
 */
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {

    @Autowired
    private MockMvc mvc;    // 웹 API 테스트할 때 사용, 이 클래스를 통해 HTTP GET, POST 등에 대한 API 테스트 가능

    @Test
    public void hello가_리턴된다() throws Exception {
        String hello = "hello";

        mvc.perform(get("/hello"))  // MockMvc를 통해 /hello 주소로 HTTP GET 요청. 체이닝 지원
                .andExpect(status().isOk()) // mvc.perform의 결과를 검증. HTTP Header의 Status를 검증(200, 404, 500).
                .andExpect(content().string(hello));    // Controller에서 "hello"를 리턴하기 때문에 값이 맞는지 검증
    }
}

```

### 코드를 모두 작성했다면 다음과 같이 메소드 왼쪽의 화살표를 클릭해보자(아래 그림 참고)  
![이미지]({{ site.url }}/images/테스트코드1번.png)
### 그럼 다음과 같이 테스트가 통과하는 것을 확인할 수 있다.  
![이미지]({{ site.url }}/images/테스트코드1번결과.png)
