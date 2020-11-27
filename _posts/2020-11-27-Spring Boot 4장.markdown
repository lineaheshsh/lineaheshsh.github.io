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
###### 머스테치의 파일 위치는 기본적으로 `src/main/resources/templates`이다.  
###### 이 위치에 머스테치 파일을 두면 스프링 부트에서 자동으로 로딩한다.  
###### 첫페이지를 담당할 index.mustache를 생성해보자 위치는 `src/main/resources/template`에 생성한다.

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

###### 머스테치 스타터 덕분에 컨트롤러에서 문자열을 반환할 때 `앞의 경로와 뒤의 파일 확장자는 자동으로 지정`된다. 앞의 경로는 `src/main/resources/templates`로, 뒤의 파일 확장자는 `.mustache`가 붙는 것이다.  

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
###### 2개의 라이브러리를 index.mustache에 추가해야 한다. 하지만 , 여기서는 바로 추가하지 않고 `레이아웃` 방식으로 추가를 한다. 레이아웃 방식이란 `공통 영역을 별도의 파일로 분리하여 필요한 곳에서 가져다 쓰는 방식`을 이야기 한다.  
###### 이번에 추가할 라이브러리들인 부트스트랩과 제이쿼리는 머스테치 화면 어디서나 필요하다. 매번 해당 라이브러리를 머스테치 파일에 추가하는 것은 귀찮은 일이니, 레이아웃 파일들을 만들어 추가한다.  
###### src/main/resources/templates 디렉토리에 layout 디렉토리를 추가하고 footer.mustache,header.mustache 파일을 생성한다.

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>스프링 부트 웹서비스</title>
    <meta http-equiv="content-type" content="text/html; charset=UTF-8" />

    <link rel="stylesheet" href="http://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
</head>
<body>
```
---
```html
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>

<!-- index.js 추가 -->
<script src="/js/app/index.js"></script>
</body>
</html>
```

###### 코드를 보면 css와 js의 위치가 서로 다른것을 볼 수있다. `페이지 로딩속도를 높이기 위해` css는 header에, js는 footer에 두었다. HTML은 위에서부터 코드가 실행되기 때문에 head가 다 실행되고서야 body가 실행된다.  
###### 즉, head가 다 불러지지 않으면 사용자 쪽에선 백지 화면만 노출이된다. 특히 js의 용량이 크면 클수록 body 부분의 실행이 늦어지기 때문에 js는 body 하단에 두어 화면이 다 그려진 뒤에 호출하는 것이 좋다.  

###### 이제 index.mustache의 코드를 다음과 같이 변경해보자.
```mustache
{{>layout/header}}
    <h1>스프링 부트로 시작하는 웹 서비스</h1>
{{>layout/footer}}
```

###### 레이아웃으로 파일을 분리했으니 index.mustache에 글 등록 버튼을 추가해 보자.
```html
{{>layout/header}}
    <h1>스프링 부트로 시작하는 웹 서비스</h1>

    <div class="col-md-12">
        <div class="row">
            <div class="col-md-6">
                <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
            </div>
        </div>
    </div>
{{>layout/footer}}
```

###### 이동할 페이지의 주소는 /posts/save이다.  
###### 이 주소에 해당하는 컨트롤러를 생성해보자. 페이지와 관련된 컨트롤러는 모두 IndexController를 사용하자.  

```java
@RequiredArgsConstructor
@Controller
public class IndexController {
    ...
    
    @GetMapping("/posts/save")
    public String postsSave() {
        return "posts-save";
    }
```

###### 이제 posts-save 페이지를 만들어보자
```html
{{>layout/header}}
<h1>게시글 등록</h1>

<div class="col-md-12">
    <div class="col-md-4">
        <form>
            <div class="form-group">
                <label for="title">제목</label>
                <input type="text" class="form-control" id="title" placeholder="제목을 입력하세요">
            </div>
            <div class="form-group">
                <label for="author">작성자</label>
                <input type="text" class="form-control" id="author" placeholder="작성자를 입력하세요">
            </div>
            <div class="form-group">
                <label for="content">내용</label>
                <textarea type="text" class="form-control" id="content" placeholder="내용을 입력하세요"></textarea>
            </div>
        </form>
        <a href="/" role="button" class="btn btn-secodnary">취소</a>
        <button type="button" class="btn btn-primary" id="btn-save">등록</button>
    </div>
</div>
{{>layout/footer}}
```

###### UI가 완성되었으니 다시 프로젝트를 실행하여 브라우저에서 http://localhost:8080/로 접근하여 '글 등록'이라고 되어있는 버튼을 클릭해보자. 아래와 같이 페이지가 열릴 것이다.  
![이미지]({{ site.url }}/images/4장게시글등록.png)

###### 하지만 아직 게시글 등록 화면에 `등록 버튼 기능이 없다.` API를 호출하는 JS가 전혀 없기 때문이다. 그래서 src/main/resources에 static/js/app 디렉토리를 생성하자.  
###### 여기에 index.js를 생성한다. 코드는 다음과 같다.

```javascript
var main = {
    init : function () {
        var _this = this;
        
        // 등록 버튼 초기화
        $('#btn-save').on('click', function () {
            _this.save();
        });
    },
    save : function () {
        var data = {
            title: $('#title').val(),
            author: $('#author').val(),
            content: $('#content').val()
        };

        $.ajax({
            type: 'POST',
            url: '/api/v1/posts',
            dataType: 'json',
            contentType: 'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function() {
            alert('글이 등록되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error))
        });
    }
};

main.init();
```

###### 자 그럼 생성된 index.js를 머스테치 파일이 쓸 수 있게 footer.mustache에 추가해 보자
```html
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>

<!-- index.js 추가 -->
<script src="/js/app/index.js"></script>
</body>
</html>
```

###### index.js 호출 코드를 보면 `절대 경로`(/)로 바로 시작한다. 스프링 부트는 기본적으로 src/main/resources/static에 위치한 자바스크립트, CSS, 이미지 등 정적 파일들은 URL에서 /로 설정된다.

###### 모든 코드가 완성되었다. 등록 기능을 브라우저에서 직접 테스트 해보자.  
![이미지]({{ site.url }}/images/4장게시글등록중.png)
###### 등록 버튼을 클릭하면 다음과 같이 "글이 등록되었습니다"라는 Alert가 노출된다.
![이미지]({{ site.url }}/images/4장게시글등록완료.png)

###### 전체 조회 화면 만들기  
###### 전체 조회를 위해 index.mustache의 UI를 변경해보자
