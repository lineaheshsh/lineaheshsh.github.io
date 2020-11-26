---
layout: post
title: Spring Boot 3장
summary: 열심히 기록하자!
author: Lee Chang Ho
date: '2020-11-26 17:28:00'
category: 프로젝트
published: true
image:  스프링부트.png
tags:   Spring Boot, Spring, Gradle, JPA
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

```java
package com.zzangho.project.springboot.domain.posts;

import org.springframework.data.jpa.repository.JpaRepository;

public interface PostsRepository extends JpaRepository<Posts,Long> {
}

```
