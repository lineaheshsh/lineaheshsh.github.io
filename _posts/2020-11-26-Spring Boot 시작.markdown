---
layout: post
title: Spring Boot 시작
summary: 열심히 기록하자!
author: Lee Chang Ho
date: '2020-11-26 14:28:00'
category: 프로젝트
published: true
image:  스프링부트.png
tags:   Spring Boot, Spring, Gradle
---

### 스프링 부트 시작(Gradle)  

###### 엊그제 교보문고에 엘라스틱서치 관련 서적을 사러갔다가 스프링 부트 책을 하나사서 왔다. 딱 보자마자 사야겠다라고 생각을 하였다. 그 이유는 책 안에 스프링 부트, 테스트, JPA, AWS 배포 등 최근에 해봐야겠다고 생각한 것들이 하나의 프로젝트로 쭈욱 이어져 있었기 때문이다. 책을 사서 공부하면서 이번에 블로그에 기록을 남겨 볼까 한다.

#### 이 글은 "스프링 부트와 AWS로 혼자 구현하는 웹 서비스" 책을 기반으로 작성하고 있습니다.  
#### 필자의 경우 Spring Tool을 STS만 쓰다가 인텔리제이가 좋다고 하여 한번 써보고 싶어 이번 프로젝트는 인텔리제이로 진행을 하게 되었습니다.  

##### 해당 프로젝트의 경우 Spring Initializr로 생성하지 않고 빈 프로젝트를 만들어 하나하나씩 만들어 가는 과정입니다.  

###### 먼저 "New Project -> Gradle" 프로젝트를 만듭니다. 그 뒤에 아래와 같이 build.gradle 파일을 정의 합니다.

#### build.gradle 파일 정의
```java
// 이 프로젝트의 플러그인 의존성 관리를 위한 설정
buildscript {
    ext {
        springBootVersion = '2.1.7.RELEASE' // 전역변수 설정
    }
    repositories {  // 각종 의존성(라이브러리)들을 어떤 원격 저장소에서 받을지를 정함.
        mavenCentral()
        jcenter()
    }
    dependencies {  // 특정 버전을 명시하면 안됨. 버전을 명시하지 않아야 버전 관리가 한 곳에 집중되고, 버전 충돌 문제도 해결됨
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'    // 스프링 부트의 의존성들을 관리해 주는 플러그인(필수)

group 'com.zzangho.project'
version '1.0-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}

```

###### 그 뒤에 src/main/java 경로 밑에 패키지를 하나 만들어준다. 필자의 경우 처음에 GroupID를 com.zzangho.project라고 지었기 때문에 com.zzangho.project.springboot라고 새로운 패키지를 하나 만들어 주었다. (아래 이미지 참고)
![이미지]({{ site.url }}/images/springboot패키지.png)
###### 그런 뒤에 Application이라는 Java Class 파일을 만들어 준 뒤 아래와 같이 소스를 추가한다.

```java
package com.zzangho.project.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

// 아래 어노테이션으로 인해 스프링 부트의 자동 설정, 스프링 Bean 읽기와 생성을 모두 자동으로 설정 됨
// 특히나 @SpringBootApplication이 있는 위치부터 설정을 읽어가기 때문에 이 클래스는 항상 프로젝트의 최상단에 위치해야만 함
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args); // 내장 WAS 실행
    }
}

```
