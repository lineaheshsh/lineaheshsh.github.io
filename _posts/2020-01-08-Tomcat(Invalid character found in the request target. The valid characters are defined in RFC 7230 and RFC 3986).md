---
layout: post
title: Tomcat(Invalid character found in the request target. The valid characters are defined in RFC 7230 and RFC 3986)
summary: 열심히 기록하자!
author: Lee Chang Ho
date: '2021-01-08 15:37:00'
category: 프로젝트
published: true
image:  엘라스틱서치.png
tags:   Tomcat setting URL
---

 ---
##### Invalid character found in the request target. The valid characters are defined in RFC 7230 and RFC 3986 오류
 ---
###### 톰캣의 특정 버전부터는 RFC 7230, RFC 3986에 의하여 특수문자를 URI에 허용하지 않는다.  (회사에서 테스트하다가 오류가 나서 찾아봄)  
###### 따라서 Get 방식으로 던지던 많은 파라미터에서 특수문자가 있다면 발생할 수 있는 문제이다.    

###### 아래 설정을 해주도록 하자
 
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTk3NDU2NDY1N119
-->