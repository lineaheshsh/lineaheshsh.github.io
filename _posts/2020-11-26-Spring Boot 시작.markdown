---
layout: post
title: Spring Boot 시작
summary: 열심히 기록하자!
author: Lee Chang Ho
date: '2020-11-26 14:28:00'
category: 프로젝트
published: true
image:  
tags:   Spring Boot, Spring, Gradle
---

### 스프링 부트 시작(Gradle)


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
