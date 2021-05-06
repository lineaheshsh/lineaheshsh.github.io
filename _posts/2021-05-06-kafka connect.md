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

###### 간단한 데이터 파이프라인을 만들 때에도 직접 Producer와 Consumer를 만들었지만 이 Kafka Connect를 이용하면 직접 만들 필요 없이 설정만 해주면 된다.

---
#### SOURCE, SINK??
---
- Source  
###### 저장소에서 데이터를 가져와 Kafka로 데이터를 "전달"
- Sink
###### Kafka에서 데이터를 가져와 저장소로 "저장"

###### Source와 Sink는 개념은 이것이 전부이다. 중요한 것은 어디서 가져오고 어디로 넣는지 인데  아래 링크를 타고 들어가보면 많은 Plugin이 존재한다는 것을 볼 수 있다.(J
###### https://www.confluent.io/hub/?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.kafka-connectors_mt.mbm_rgn.apac_lng.eng_dv.all_con.kafka-connectors&utm_term=%2Bkafka%20%2Bconnector&creative=&device=c&placement=&gclid=CjwKCAjwhMmEBhBwEiwAXwFoEWGTxwFT2El8BB9tnx3x3e30lnbThm1hRmitiqudbOyZYNUkg5g2BBoCfk4QAvD_BwE
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExMTUyMzk5M119
-->