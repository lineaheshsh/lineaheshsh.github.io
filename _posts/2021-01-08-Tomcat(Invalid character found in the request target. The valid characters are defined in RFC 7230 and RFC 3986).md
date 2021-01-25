---
layout: post
title: Tomcat(Invalid character found in the request target. The valid characters are defined in RFC 7230 and RFC 3986)
summary: 열심히 기록하자!
author: Lee Chang Ho
date: '2021-01-08 15:37:00'
category: 프로젝트
published: true
image:  tomcatlogo.png
tags:   Tomcat setting URL
---

 ---
##### Invalid character found in the request target. The valid characters are defined in RFC 7230 and RFC 3986 오류
 ---
###### 톰캣의 특정 버전부터는 RFC 7230, RFC 3986에 의하여 특수문자를 URI에 허용하지 않는다.  (회사에서 테스트하다가 오류가 나서 찾아봄)  
###### 따라서 Get 방식으로 던지던 많은 파라미터에서 특수문자가 있다면 발생할 수 있는 문제이다.    

###### 해결방법은 Tomcat의 server.xml에 다음 옵션을 추가해주면 된다.  *relaxedQueryChars* 
###### 아래 설정을 참고하자 실제로 내가 적용한 형태다.  relaxedQueryChart의 Value는 URI에 들어가는 특수문자를 정의해 주면 된다.  

```xml
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
			   URIEncoding="UTF-8"
			   relaxedQueryChars="^{}[]|&quot;" />
``` 

###### 필자의 경우 인텔리제이를 사용하여 실제 Tomcat이 설치 된 디렉토리 아래 conf/server.xml을 수정하였다.   
###### 수정 후에는 재기동을 잊지 말도록!
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2MTMwNTU3ODRdfQ==
-->