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

###### 그 다음으로는 models.py에 Model을 만들어주자
```python
from  djangoProject.database  import  Base

from  django.db  import  models

from  sqlalchemy  import  Column, Integer, String, DateTime

  

# Create your models here.

class  Board(models.Model):

ttl = models.CharField(max_length=50)

contents = models.TextField()

writer = models.CharField(max_length=20)

  

# MySQL DB Create Model

class  MySQLBoard(Base):

__tablename__ = 'covid_board'

idx = Column(Integer, primary_key=True, unique=True, autoincrement=True)

ttl = Column(String)

contents = Column(String)

writer = Column(String)

keyword = Column(String)

date = Column(DateTime)

  

def  __repr__(self):

return  "<MySQLBoard(idx='%d', ttl='%s', contents='%s', writer='%s', keyword='%s', date='%s')>" % (

self.idx, self.ttl, self.contents, self.writer, self.keyword, str(self.date))
```
###### 위에서 MySQLBoard라는 Model을 새로 만들었다.
###### __tablename__ 이 부분이 테이블명과 맵핑이 되는 부분인듯 하다. 나머지 밑에 부분들은 테이블의 컬럼으로 보면 된다.
###### 이제 views.py에서 연동한 sqlalchemy을 통해 데이터를 가져오고 등록해 보자

###### 아래 부분을 상단에 추가해 주고
```python
from  djangoProject.database  import  db_session
import  datetime
```

###### def board함수에 아래처럼 소스를 수정하자
```python
def  board(request):

  

	query_result = db_session.query(MySQLBoard).all()

	  

	# 페이지에 데이터를 던져주어 페이지에서 boardList 데이터를 받을 수 있다.

	return  render(request, 'main/board/board.html', {'boardList':query_result})
```

###### 그리고 new_board함수도 마찬가지로 아래처럼 소스를 수정하자
```python
## 데이터 등록

def  new_board(request):

  

	## request method가 POST이면 아래 로직 수행

	if  request.method == 'POST':

	  

	mysqlBoard = MySQLBoard(ttl=request.POST['ttl'], contents=request.POST['contents'], writer=request.POST['writer'], keyword=request.POST['keyword'], date=datetime.datetime.now())

	db_session.add(mysqlBoard)

	db_session.commit()

	## 등록이 성공하면 목록 페이지로 이동

	return  redirect('/board/')

	  

	## POST 요청이 아니면 등록 화면으로 이동

	return  render(request, 'main/board/new_board.html')
```

###### 이상으로 sqlalchemy 연동이 완료되었다. 이제 테스트를 해보자

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc5NDQwMjg2Nyw1ODE1MTMwNDgsMTg2MT
c4ODcyNSwtNzcxODk0MjYyXX0=
-->