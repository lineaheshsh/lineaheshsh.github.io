---
layout: post
title: Spring Boot 3장
summary: 열심히 기록하자!
author: Lee Chang Ho
date: '2020-11-26 17:28:00'
category: 프로젝트
published: true
image:  스프링부트.png
tags:   SpringBoot Spring Gradle JPA
---

### 스프링 부트 3장(JPA)  

###### 이번 화에서는 "스프링 부트"에서 "JPA"를 사용하는 방법에 대해 기록한다.  

###### 먼저 build.gradle에 다음과 같이 org.springframework.boot:spring-boot-stater-data-jpa와 com.h2.database:h2 의존성을 등록한다.

```java
dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.projectlombok:lombok') // Lombok 의존성 등록
    compile('org.springframework.boot:spring-boot-starter-data-jpa')    // JPA 의존성 등록
    compile('com.h2database:h2')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

###### 의존성이 등록되었다면, 다음과 같이 패키지를 만든다. (com.zzangho.project.springboot.domain.posts)  
![이미지]({{ site.url }}/images/JPA폴더구조.png)
###### 이 domain 패키지는 도메인을 담을 패키지이다.  
###### 이 패키지에 Posts 클래스를 만든다. 그리고는 아래와 같이 소스코드를 추가한다.

```java
package com.zzangho.project.springboot.domain.posts;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Getter
@NoArgsConstructor
// 기본 생성자 자동 추가
@Entity // 테이블과 링크될 클래스
public class Posts {

    @Id // 해당 테이블의 PK 필드
    @GeneratedValue(strategy = GenerationType.IDENTITY) // PK생성 규칙, 해당 설정은 auto_increment
    private Long id;

    @Column(length = 500, nullable = false) // 기본값 외에 추가로 변경이 필요한 옵션이 있을경우 사용
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    // 해당 클래스의 빌더 패턴 클래스 생성
    // 생성자 상단에 선언 시 생성자에 포함된 필드만 빌더에 포함
    @Builder
    public Posts(String title, String content, String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }
}

```

###### Posts 클래스 작성이 완료되었다면, Posts 클래스로 Databases를 접근하게 해줄 JpaRepository를 생성한다.(아래 이미지 참고)
![이미지]({{ site.url }}/images/JPA폴더구조2.png)  
```java
package com.zzangho.project.springboot.domain.posts;

import org.springframework.data.jpa.repository.JpaRepository;

public interface PostsRepository extends JpaRepository<Posts,Long> {
}

```

###### 보통 ibatis나 MyBatis 등에서 Dao라고 불리는 DB Layer 접근자이며, JPA에선 Repository라고 부르며 인터페이스로 생성한다.
###### 단순히 인터페이스를 생성 후, JpaRepository<Entity 클래스, PK타입>를 상속하면 기본적인 CRUD 메소드가 자동으로 생성된다.
###### @Repository를 추가할 필요도 없으며 여기서 주의할 점은 Entity 클래스와 기본 Entity Repository는 함께 위치해야 하는 점이다. 둘은 아주 밀접한 관계이고, Entity 클래스는 기본 Repository 없이는 제대로 역할을 할 수가 없기 때문이다.  

###### 모두 작성되었다면 간단하게 테스트 코드로 기능을 검증해 보자  

###### test 디렉토리에 domain.posts 패키지를 생성하고, 테스트 클래스는 PostsRepositoryTest란 이름으로 생성한다.  
###### PostsRepositoryTest에서는 다음과 같이 save, findAll 기능을 테스트 한다.  

```java
package com.zzangho.project.springboot.domain.posts;

import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import static org.assertj.core.api.Assertions.assertThat;

import java.util.List;

@RunWith(SpringRunner.class)
@SpringBootTest
public class PostsRepositoryTest {

    @Autowired
    PostsRepository postsRepository;

    // Junit에서 단위 테스트가 끝날 때마다 수행되는 메소드를 지정
    // 보통은 배포 전 전체 테스트를 수행할 때 테스트간 데이터 침범을 막기 위해 사용합니다.
    @After
    public void cleanup() {
        postsRepository.deleteAll();
    }

    @Test
    public void 게시글저장_불러오기() {
        //given
        String title = "테스트 게시글";
        String content = "테스트 본문";

        // 테이블 posts에 insert/update 쿼리 실행
        // id가 있다면 update, 없다면 insert 실행
        postsRepository.save(Posts.builder()
                                  .title(title)
                                  .content(content)
                                  .author("lineaheshsh@naver.com")
                                  .build());

        //when
        List<Posts> postsList = postsRepository.findAll();

        //then
        Posts posts = postsList.get(0);
        assertThat(posts.getTitle()).isEqualTo(title);
        assertThat(posts.getContent()).isEqualTo(content);
    }
}

```

###### 테스트를 해보면 다음과 같이 정상적으로 실행이 되는 것을 확인할 수 있다.  
![이미지]({{ site.url }}/images/테스트코드1번결과.png)

###### 하지만 여기서 한가지 궁금한 점이 있다. "실제로 실행된 쿼리는 어떤 형태일까? 라는 점이다.
###### 실행된 쿼리를 로그로 볼 수 있게 ON/OFF 하는 설정이 있다. 스프링 부트에서는 application.properties, application.yml 등의 파일로 한 줄의 코드로 설정할 수 있도록 지원하고 권장하니 이를 사용하면 된다.

![이미지]({{ site.url }}/images/applicationProperties.png)

###### 옵션은 다음과 같다.
```properties
spring.jpa.show_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect   // MySQL 버전으로 쿼리 로그 변경
```

###### 그럼 다음과 같이 콘솔에서 쿼리 로그를 확인 할 수 있다.  

![이미지]({{ site.url }}/images/3장쿼리실행결과.png)


###### JPA와 H2의 기본적인 기능과 설정을 진행했으니, 본격적으로 API를 만들어 보자

---
### 등록/수정/조회 API 만들기  
###### API를 만들기 위해선 총 3개의 클래스가 필요하다.  
+ Request 데이터를 받을 Dto
+ API 요청을 받을 Controller
+ 트랜잭션, 도메인 기능 간의 순서를 보장하는 Service

![이미지]({{ site.url }}/images/3장스프링웹계층.png)

###### 간단하게 각 영역을 소개하자면 다음과 같다. 

+ Web Layer
    - 흔히 사용하는 컨트롤러(@Controller)와 JSP /Freemarker 등의 뷰 템플릿 영역
    - 이외에도 필터(@Filter), 인터셉터, 컨트롤러 어드바이스(@ControllerAdvice) 등 외부 요청과 응답에 대한 전반적인 영역을 이야기 합니다.
+ Service Layer
    - @Serivce에 사용되는 서비스 영역입니다.
    - 일반적으로 Controller와 Dao 중간 영역에서 사용됩니다.
    - @Transactional이 사용되어야 하는 영역이기도 합니다.
+ Repository Layer
    - Database와 같이 데이터 저장소에 접근하는 영역입니다.
    - 기존에 개발하셨던 분들이라면 Dao영역으로 이해하시면 쉬울것입니다.
+ Dtos
    - Dto(Data Transfer Object)는 계층 간에 데이터 교환을 위한 객체를 이야기하며 Dtos는 이들의 영역을 얘기합니다.
    - 예를 들어 뷰 템플릿 엔진에서 사용될 객체나 Repository Layer에서 결과로 넘겨준 객체 등이 이들을 이야기합니다.
+ Domain Model
    - 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록 단순화시킨 것을 도메인 모델이라고 합니다.
    - 이를테면 택시 앱이라고 하면 배차, 탑승, 요금 등이 모두 도메인이 될 수 있습니다.
    - @Entity를 사용해보신 분들은 @Entity가 사용된 영역 역시 도메인 모델이라고 이해해주시면 됩니다.
    - 다만 무조건 데이터베이스의 테이블과 관계가 있어야만 하는 것은 아닙니다.
    - VO처럼 값 객체들도 이 영역에 해당하기 때문입니다.
    
    
###### 그럼 등록, 수정, 삭제 기능을 만들어 보자. PostsApiController를 web 패키지에, PostsSaveRequestDto를 web.dto 패키지에, PostsService를 service.posts 패키지에 생성한다.

```java
package com.zzangho.project.springboot.web;

import com.zzangho.project.springboot.service.posts.PostsService;
import com.zzangho.project.springboot.web.dto.PostsSaveRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RequiredArgsConstructor
@RestController
public class PostsApiController {
    // @Autowired를 안쓰는 이유는 생성자로 Bean 객체를 받도록 하기 위함.
    // 롬복의 @RequiredArgsConstructor 어노테이션이 "final"이 선언된 모든 필드를 인자값으로 하는 생성자를 대신 생성해준다.
    private final PostsService postsService;

    @PostMapping("/api/v1/posts")
    public Long save(@RequestBody PostsSaveRequestDto requestDto) {
        return postsService.save(requestDto);
    }

}
```

```java
   package com.zzangho.project.springboot.service.posts;

import com.zzangho.project.springboot.domain.posts.Posts;
import com.zzangho.project.springboot.domain.posts.PostsRepository;
import com.zzangho.project.springboot.web.dto.PostsSaveRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    @Transactional
    public Long save(PostsSaveRequestDto requestDto) {
        return postsRepository.save(requestDto.toEntity()).getId();
    }
}

```

```java
package com.zzangho.project.springboot.web.dto;

import com.zzangho.project.springboot.domain.posts.Posts;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class PostsSaveRequestDto {
    private String title;
    private String content;
    private String author;

    @Builder
    public PostsSaveRequestDto(String title, String content, String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }

    public Posts toEntity() {
        return Posts.builder()
                .title(title)
                .content(content)
                .author(author)
                .build();
    }
}

```
