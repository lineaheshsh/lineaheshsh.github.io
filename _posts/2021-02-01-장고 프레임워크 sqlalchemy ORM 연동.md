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

###### index.html페이지를 아래 처럼 바꿔주자
```python
<!DOCTYPE html>

<html  lang="en">

<head>

{% load static %}

  

<meta  charset="UTF-8">

<title>{% block  title %}{% endblock %}</title>

  

<link  rel="stylesheet"  href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css"  integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh"  crossorigin="anonymous">

<script  src="https://code.jquery.com/jquery-3.4.1.slim.min.js"  integrity="sha384-J6qa4849blE2+poT4WnyKhv5vZF5SrPo0iEjwBvKU7imGFAV0wwj1yYfoRSJoZ+n"  crossorigin="anonymous"></script>

<script  src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js"  integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo"  crossorigin="anonymous"></script>

<script  src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js"  integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6"  crossorigin="anonymous"></script>

<!-- offcanvas -->

<!--

<link rel="stylesheet" href="{% static 'css/offcanvas.css' %}" />

<script src="{% static 'js/offcanvas.js' %}" /> -->

<style>

  

html,

body {

overflow-x: hidden; /* Prevent scroll on narrow devices */

}

  

body {

padding-top: 56px;

}

  

@media (max-width: 991.98px) {

.offcanvas-collapse {

position: fixed;

top: 56px; /* Height of navbar */

bottom: 0;

left: 100%;

width: 100%;

padding-right: 1rem;

padding-left: 1rem;

overflow-y: auto;

visibility: hidden;

background-color: #343a40;

transition: visibility .3s ease-in-out, -webkit-transform .3s ease-in-out;

transition: transform .3s ease-in-out, visibility .3s ease-in-out;

transition: transform .3s ease-in-out, visibility .3s ease-in-out, -webkit-transform .3s ease-in-out;

}

.offcanvas-collapse.open {

visibility: visible;

-webkit-transform: translateX(-100%);

transform: translateX(-100%);

}

}

  

.nav-scroller {

position: relative;

z-index: 2;

height: 2.75rem;

overflow-y: hidden;

}

  

.nav-scroller .nav {

display: -ms-flexbox;

display: flex;

-ms-flex-wrap: nowrap;

flex-wrap: nowrap;

padding-bottom: 1rem;

margin-top: -1px;

overflow-x: auto;

color: rgba(255, 255, 255, .75);

text-align: center;

white-space: nowrap;

-webkit-overflow-scrolling: touch;

}

  

.nav-underline .nav-link {

padding-top: .75rem;

padding-bottom: .75rem;

font-size: .875rem;

color: #6c757d;

}

  

.nav-underline .nav-link:hover {

color: #007bff;

}

  

.nav-underline .active {

font-weight: 500;

color: #343a40;

}

  

.text-white-50 { color: rgba(255, 255, 255, .5); }

  

.bg-purple { background-color: #6f42c1; }

  

.lh-100 { line-height: 1; }

.lh-125 { line-height: 1.25; }

.lh-150 { line-height: 1.5; }

  
  

</style>

</head>

<body  class="bg-light">

<nav  class="navbar navbar-expand-lg fixed-top navbar-dark bg-dark">

<a  class="navbar-brand mr-auto mr-lg-0"  href="#">Django Framework + Python + sqlalchemy Study</a>

<button  class="navbar-toggler p-0 border-0"  type="button"  data-toggle="offcanvas">

<span  class="navbar-toggler-icon"></span>

</button>

  

<div  class="navbar-collapse offcanvas-collapse"  id="navbarsExampleDefault">

<ul  class="navbar-nav mr-auto">

<li  class="nav-item active">

<a  class="nav-link"  href="#">Dashboard <span  class="sr-only">(current)</span></a>

</li>

<li  class="nav-item">

<a  class="nav-link"  href="#">Notifications</a>

</li>

<li  class="nav-item">

<a  class="nav-link"  href="#">Profile</a>

</li>

<li  class="nav-item">

<a  class="nav-link"  href="#">Switch account</a>

</li>

<li  class="nav-item dropdown">

<a  class="nav-link dropdown-toggle"  href="#"  id="dropdown01"  data-toggle="dropdown"  aria-haspopup="true"  aria-expanded="false">Settings</a>

<div  class="dropdown-menu"  aria-labelledby="dropdown01">

<a  class="dropdown-item"  href="#">Action</a>

<a  class="dropdown-item"  href="#">Another action</a>

<a  class="dropdown-item"  href="#">Something else here</a>

</div>

</li>

</ul>

<form  class="form-inline my-2 my-lg-0">

<input  class="form-control mr-sm-2"  type="text"  placeholder="Search"  aria-label="Search">

<button  class="btn btn-outline-success my-2 my-sm-0"  type="submit">Search</button>

</form>

</div>

</nav>

  

<div  class="nav-scroller bg-white shadow-sm">

<nav  class="nav nav-underline">

<a  class="nav-link active"  href="#">Dashboard</a>

<a  class="nav-link"  href="#">

Friends

<span  class="badge badge-pill bg-light align-text-bottom">27</span>

</a>

<a  class="nav-link"  href="#">Explore</a>

<a  class="nav-link"  href="#">Suggestions</a>

<a  class="nav-link"  href="#">Link</a>

<a  class="nav-link"  href="#">Link</a>

<a  class="nav-link"  href="#">Link</a>

<a  class="nav-link"  href="#">Link</a>

<a  class="nav-link"  href="#">Link</a>

</nav>

</div>

  

<main  role="main"  class="container">

{% block  content %}

  

{% endblock %}

</main>

</body>

</html>
```

###### 그 다음으로 board.html을 아래 처럼 바꿔주자
```python
{% extends  '../index.html' %}

  

{% block  content %}

<div  class="d-flex align-items-center p-3 my-3 text-white-50 bg-purple rounded shadow-sm">

<div  class="lh-100">

<h6  class="mb-0 text-white lh-100">Bootstrap</h6>

<small>Since 2011</small>

</div>

</div>

  

<div  class="my-3 p-3 bg-white rounded shadow-sm">

<h6  class="border-bottom border-gray pb-2 mb-0">Recent updates</h6>

{% for  list  in  boardList %}

<div  class="media text-muted pt-3">

<svg  class="bd-placeholder-img mr-2 rounded"  width="32"  height="32"  xmlns="http://www.w3.org/2000/svg"  preserveAspectRatio="xMidYMid slice"  focusable="false"  role="img"  aria-label="Placeholder: 32x32"><title>Placeholder</title><rect  width="100%"  height="100%"  fill="#007bff"/><text  x="50%"  y="50%"  fill="#007bff"  dy=".3em">32x32</text></svg>

<p  class="media-body pb-3 mb-0 small lh-125 border-bottom border-gray">

<strong  class="d-block text-gray-dark"><a  href="{{list.pk}}/">@{{list.ttl}} / {{list.writer}}</a></strong>

{{list.contents}}<br>

{{list.keyword}} / {{list.date}}

</p>

</div>

{% endfor %}

<small  class="d-block text-right mt-3">

<button  type="button"  class="btn btn-dark"><a  href="new_board">글쓰기</a></button>

</small>

</div>

  

{% endblock %}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg4NDE5ODE1NiwtNzcxODk0MjYyXX0=
-->