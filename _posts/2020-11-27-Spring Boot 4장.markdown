---
layout: post
title: Spring Boot 4장
summary: 열심히 기록하자!
author: Lee Chang Ho
date: '2020-11-27 15:52:00'
category: 프로젝트
published: true
image:  스프링부트.png
tags:   SpringBoot Spring Gradle Mustache 머스테치
---

### 스프링 부트 4장(머스테치)  

###### 이번 화에서는 머스테치를 통해 화면 영역을 개발하는 방법을 배우자  

###### 서버 템플릿 엔진과 클라이언트 템플릿 엔진의 차이는 무엇인지, 이 책에서는 왜 JSP가 아닌 머스테치를 선택했는지, 머스테치를 통해 기본적인 CRUD 화면 개발 방법 등을 차례로 진행해보자.  

###### 먼저 템플릿 엔진이란 무엇인가. 일반적으로 웹 개발에 있어 템플릿 엔진이란, `지정된 템플릿 양식과 데이터`가 합쳐져 HTML문서를 출력하는 소프트웨어를 이야기한다. 예전에 스프링이나 서블릿을 사용해 본 분들은 아마도 JSP, Freemarker(첨들어보는것...) 등을 떠올리실 테고, 요즘 개발을 시작한 분들은 리액트, 뷰의 View 파일을 떠올릴 것이다.  
###### 둘 모두 결과적으로 지정된 템플릿과 데이터를 이용하여 HTML을 생성하는 템플릿 엔진이라고 한다.(필자의 경우 간혹 Vuejs 를 하였지만 주로 JSP를 이용하였기 때문에 이번에 머스테치란 템플릿 엔진이 무엇인지 무척이나 기대 된다.)  

###### 다만, 조금의 차이가 있다고 한다. 전자는 서버 템플릿 엔진이라 불리고, 후자는 클라이언트 템플릿 엔진이라 불린다.  

+ 서버 템플릿 엔진
###### `서버에서 Java 코드로 문자열`을 만든 뒤 이 문자열을 HTML로 변환하여 `브라우저로 전달`  

+ 클라이언트 텤플릿 엔진
###### `브라우저에서 화면을 생성`하며 즉, `서버에서 이미 코드가 벗어난 경우`  

#### 머스테치란  

###### 수많은 언어를 지원하는 가장 심플한 템플릿 엔진이라고 한다. 루비, 자바스크립트, 파이썬, php, JAVA, 펄, Go, ASP 등 현존하는 대부분 언어를 지원하고 있다. 그러다 보니 자바에서 사용 될 때는 서버 템플릿 엔진으로, 자바스크립트에서 사용 될 때는 클라이언트 템플릿 엔진으로 모두 사용할 수 있다.  
###### 자바 진영에서는 JSP, Velocity, Freemarker, Thymeleaf 등 다양한 서버 템플릿 엔진이 존재한다.  
###### 저자가 생각하는 템플릿 엔진들의 단점은 아래와 같다고 한다.  
+ JSP, Velocity : 스프링 부트에서는 권장하지 않는 템플릿 엔진
+ Freemarker : 템플릿 엔진으로는 너무 과하게 많은 기능을 지원한다. 높은 자유도로 인해 숙련도가 낮을수록 Freemarker 안에 비즈니스 로직이 추가될 확률이 높다.
+ Thymeleaf : 스프링 진영에서 적극적으로 밀고 있지만 문법이 어렵다. HTML 태그에 속성으로 템플릿 기능을 사용하는 방식이 기존 개발자분들께 높은 허들로 느껴지는 경우가 많다. 실제로 사용해 본 분들은 자바스크립트 프레임워크를 배우는 기분이라고 후기를 이야기하기도 한다. 물론 Vue.js를 사용해 본 경험이 있어 태그 속성 방식이 익숙한 분이라면 Thymeleaf를 선택해도 된다.

###### 반면 머스테치의 장점은 다음과 같다.  
+ 문법이 다른 템플릿 엔진보다 심플하다.
+ 로직 코드를 사용할 수 없어 View의 역할과 서버의 역할이 명확하게 분리된다.
+ Mustache.js와 Mustache.java 2가지가 다 있어, 하나의 문법으로 클라이언트/서버템플릿을 모두 사용 가능하다.

###### 먼저 머스테치 플러그인을 설치를 해보자.  
###### plugin에 들어가서 [Marketplace] 에서 mustache를 검색하여 Install을 한다.  
###### 그런 뒤에 가장 먼저 build.gradle에 의존성을 추가 한다.  

```properties
compile('org.springframework.boot:spring-boot-starter-mustache')
```

###### 보는 것 처럼 머스테치는 `스프링 부트에서 공식 지원하는 템플릿 엔진`이다.  
###### 머스테치의 파일 위치는 기본적으로 __src/main/resources/templates__이다.  
###### 이 위치에 머스테치 파일을 두면 스프링 부트에서 자동으로 로딩한다.  
###### 첫페이지를 담당할 index.mustache를 생성해보자 위치는 __src/main/resources/template에 생성한다.

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>스프링 부트 웹서비스</title>
    <meta http-equiv="content-type" content="text/html; charset=UTF-8" />
</head>
<body>
    <h1>스프링 부트로 시작하는 웹 서비스</h1>
</body>
</html>
```

###### 그리고 이 머스테치에 URL을 매핑해보자. URL 매핑은 당연하게 Controller에서 진행한다. web 패키지 안에 IndexController를 생성하자.  

```java
package com.zzangho.project.springboot.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class IndexController {

    @GetMapping("/")
    public String index(Model model) {
        return "index";
    }
}

```

###### 머스테치 스타터 덕분에 컨트롤러에서 문자열을 반환할 때 `앞의 경로와 뒤의 파일 확장자는 자동으로 지정`된다. 앞의 경로는 __src/main/resources/templates__로, 뒤의 파일 확장자는 __.mustache__가 붙는 것이다.  

###### 자 여기까지 코드가 완성됬으면 테스트 코드로 검증해 보자. test 패키지에 IndexControllerTest 클래스를 만든다.  

```java
package com.zzangho.project.springboot.web;

import com.zzangho.project.springboot.domain.posts.Posts;
import com.zzangho.project.springboot.domain.posts.PostsRepository;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.test.context.junit4.SpringRunner;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.boot.test.context.SpringBootTest.WebEnvironment.RANDOM_PORT;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = RANDOM_PORT)
public class IndexControllerTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void 메인페이지_로딩() {
        //when
        String body = this.restTemplate.getForObject("/", String.class);

        //then
        assertThat(body).contains("스프링 부트로 시작하는 웹 서비스");

    }
}

```

###### HTML도 결국은 `규칙이 잇는 문자열`이다. TestRestTemplate를 통해 "/"로 호출했을 때 index.mustache에 포함된 코드들이 있는지 확인하면 된다. 전체 코드를 다 검증할 필요는 없으니, "스프링 부트로 시작하는 웹 서비스" 문자열이 포함되어 있는지만 비교해보자. 테스트를 돌려보면 정상적으로 코드가 수행되는 것을 확인 할 수 있다.  

###### 다음으로는 게시글 등록, 수정, 삭제 기능을 만들어보자.  
---
###### 첫번째로는 게시글 등록 화면을 만들어보자.  
###### 여기서는 오픈소스인 부트스트랩, 제이쿼리를 CDN 형식으로 사용하여 화면을 만들고 있다.
###### 2개의 라이브러리를 index.mustache에 추가해야 한다. 하지만 , 여기서는 바로 추가하지 않고 __레이아웃__ 방식으로 추가를 한다. 레이아웃 방식이란 __공통 영역을 별도의 파일로 분리하여 필요한 곳에서 가져다 쓰는 방식__을 이야기 한다.  
