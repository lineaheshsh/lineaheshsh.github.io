---
layout: post
title: max virtual memory areas vm.max_map_count
summary: 열심히 기록하자!
author: Lee Chang Ho
date: '2021-04-23 18:08:00'
category: 검색엔진
published: true
image:  엘라스틱서치.png
tags:   검색엔진 엘라스틱서치 ElasticSearch
---

## max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

###### 로컬에서 도커를 이용하여 elasticsearch를 띄우다 max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144] 에러를 만났다.   
###### 리눅스 계열의 환경이라면 아래 명령어를 입력하여 해결을 할 수가 있다.
```bash
sudo sysctl -w vm.max_map_count=262144
또는
echo 262144 > /proc/sys/vm/max_map_count
/bin/sh: 11: cannot create 
```
###### 하지만 나의 경우 window 환경으로 위의 명령어가 실행이 되지 않는다  
###### 어떻게 해야 하는 것인가(구글링을 열심히 해보자....)

#### 구글링을 통해 찾은 방법
---
- 도커 컨테이너 환경에 들어가서 sudo sysctl -w vm.max_map_count=262144 명령어를 사용한다

###### 위의 방법을 보자마자 오!? 그러면 되겠네 어차피 컨테이너의 안에 들어가면 리눅스 환경이니 되겠다!! 

###### 하지만..
```bash
sudo echo 262144 > /proc/sys/vm/max_map_count
/bin/sh: 11: cannot create /proc/sys/vm/max_map_count: Read-only file system
```  

###### read-only로 잡혀 수정이 안되어버린다.  
###### 계속 방법을 찾던 중 친구가 나에게 방법을 알려주었다.

```bash
wsl -d docker-desktop
```

###### 해당 명령어를 cmd 창에서 실행을 하면 리눅스 환경으로 cmd창이 바뀐다. 그 다음 명령어를 치니 잘 된다!
<!--stackedit_data:
eyJoaXN0b3J5IjpbNTk0MDkzNDczXX0=
-->