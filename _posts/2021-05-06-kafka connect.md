---
layout: post
title: kafka connect
summary: 열심히 기록하자!
author: Lee Chang Ho
date: '2021-02-16 23:08:00'
category: 프로젝트
published: true
image:  kafka/logo.png
tags:   Apache Kafka Connect Debezium
---

## Kafka Connect
###### 카프카를 사용하다보면 Kafka Connect라는 용어를 들어볼 수 있다.  
###### 그럼 카프카 커넥트를 한번 알아보자.

---
#### Kafka Connect란?
--- 
###### 기본적으로 Kafka는 Broker, Producer, Consumer로 구성이 되어있다.
###### 여기에서 Broker는 카프카이며 Producer는 메세지를 생산하는 역할, Consumer는 메세지를 소비하는 역할을 담당하고 있다.  
###### 이 구성에서 Producer와 Consumer의 역할을 대체할 수 있도록 나온것이 Kafka Connect이다.

###### 간단한 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTU1MDM0OTU0N119
-->