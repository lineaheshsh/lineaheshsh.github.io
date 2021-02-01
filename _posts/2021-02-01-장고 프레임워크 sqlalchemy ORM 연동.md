---
layout: post
title: 장고 프레임워크 sqlalchemy ORM 연동
summary: 열심히 기록하자!
author: Lee Chang Ho
date: '2021-02-01 14:17:00'
category: 프로젝트
published: true
image:  django.png
tags:   장고 Django 프레임워크 파이썬 Python 부트스트랩 sqlalchemy
---

## 장고 프레임워크 sqlalchemy 연동하기
---
#### 심심풀이 Bootstrap 적용
---
###### 일단 페이지가 너무 밋밋하여 부트스트랩 템플릿을 하나 적당히 골라서 기존 페이지에 입혀 주었다. 

###### 소스는 https://github.com/lineaheshsh/djangoProject/tree/master 여기를 참고하자
###### 바뀐 조회 화면
![이미지]({{ site.url }}/images/부트스트랩조회화면.png)

###### 바뀐 등록 화면
![이미지]({{ site.url }}/images/부트스트랩등록화면.png)


---
#### sqlalchemy 연동
---
###### 첫번째로 sqlalchemy을 설치를 해야 한다. 그리고 우리는 여깃 mysql을 사용할 것이니 pymysql도 같이 설치를 해주자
```python
pip install sqlalchemy
pip install pymysql
```

###### 그 다음에 DB연결을 해주는 database.py 파일을 만들자
```python
from  sqlalchemy  import  create_engine

from  sqlalchemy.orm  import  scoped_session, sessionmaker

from  sqlalchemy.ext.declarative  import  declarative_base

  

id = "root"

password = "1q2w3e4r5t"

host = "localhost"

db_name = "covid"

  

engine = create_engine("mysql+pymysql://{0}:{1}@{2}/{3}?charset=utf8".format(id, password, host, db_name), convert_unicode=False, pool_pre_ping=True)

db_session = scoped_session(sessionmaker(autocommit=False, autoflush=False, bind=engine))

  

Base = declarative_base()

Base.query = db_session.query_property()

  

def  init_db():

Base.metadata.create_all(engine)
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2NjczNDg4OTAsMTg2MTc4ODcyNSwtNz
cxODk0MjYyXX0=
-->