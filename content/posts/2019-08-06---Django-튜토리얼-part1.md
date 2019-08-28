---
title: Django 튜토리얼 part1 
date: "2019-08-06T17:21:00.169+09:00"
template: "post"
draft: false
slug: "/posts/django-tutorial-part1/"
category: "python"
tags:
  - "django"
  - "장고"
description: "[PART1] 가장 많이 쓰이는 웹 어플리케이션 프레임워크인 장고 튜토리얼을 해보자"
---


## django
---
장고는 파이썬으로 만들어진 model-view-template 웹 프레임워크다. 전에 일하면서 간단한 api를 만들 일이 있었는데 그 때는 빠르게 작업하고 싶어서 Flask를 사용했다.

Flask는 필요한 부분을 하나씩 붙여나가며 만들어야하기 때문에 당시 api 작업하기에 알맞다고 생각해서 사용했다(도큐먼트는 정말 보기 힘들었다).

그래서 ~~도큐먼트가 더 이쁜~~ 이번에는 장고를 배워보고자 공식 문서에 있는 튜토리얼을 해보기로 했다.

공식 문서의 튜토리얼은 [이곳](https://docs.djangoproject.com/ko/2.2/intro/tutorial01/)에서 확인할 수 있다.


## part 1
---

### 프로젝트 만들기

장고를 설치한 후 디렉토리를 만들고 프로젝트를 시작해보자

```
$ mkdir django-tutorial
$ django-admin startproject mysite
$ cd mysite
```

mysite 안에 들어가면 manage.py와 mysite 디렉토리가 하나 더 있다.
manage.py는 커맨드라인의 유틸리티, mysite에는 프로젝트를 위한 파이썬 패키지들이 들어가 있다.

제대로 프로젝트가 동작하는지 알아보자.

```
$ python manage.py runserver
```

```
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 17 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.

August 06, 2019 - 05:31:01
Django version 2.2.4, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

개발 서버를 띄웠다.
http://127.0.0.1:8000/으로 가면 우주선이 이륙하는 걸 볼 수 있다.

![img](/media/runserver.png)

그럼 이제 장비를(?) 종료하고 다음으로 넘어가자.

![img](/media/stop_it.jpg)
~~장비가 종료가 안되면 큰일이 일어난다 주의하자~~

### 앱 만들기

**앱은 특정한 기능(블로그나, 설문조사 앱 등)을 수행하는 웹 어플리케이션이다. 프로젝트에는 다수의 앱이 포함될 수 있고, 반대로 앱은 다수의 프로젝트에 포함될 수 있다.**

앱은 파이썬 경로 어디든 만들 수 있지만 튜토리얼에서는 최상위에서 바로 임포트 할 수 있도록 manage.py 옆에 생성하기로 한다.

```
$ python manage.py startapp polls
```

polls 라는 디렉토리가 생성되었다. 이제 뷰를 작성해보자.

### 뷰 작성하기

뷰를 만들기 위해 polls/views.py를 열고 코드를 작성하자.

```python
from django.shortcuts import render
from django.http import HttpResponse

def index(request):
    return HttpResponse("hello world! You are already at index")
```

뷰를 작성하고 호출하려면 연결된 URL이 필요하다. 이를 위해 URLconf가 사용된다고 하는데 아직 튜토리얼에서 그게 뭔지 설명해주지 않았으니 일단 넘어가자. 나중에 분명 나오겠지.

이제 polls 디렉토리 안에서 URLconf를 생성하기 위해 urls.py 파일을 만들자.

polls/urls.py에 코드를 작성하자.

```python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index')
]
```

이제 최상위 URLconf에서 polls.urls 모듈을 바라보게 하기 위해 mysite/urls.py에 필요한 사항을 추가한다.

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

django.urls.include를 import하고 urlpatterns 리스트에 polls 를 바라보는 include 함수를 추가했다.

include 함수를 왜 추가해야하는지 튜토리얼님이 잘 설명해주었으니 그대로 인용한다.

> include() 함수는 다른 URLconf들을 참조할 수 있도록 도와줍니다. Django가 함수 include()를 만나게 되면, URL의 그 시점까지 일치하는 부분을 잘라내고, 남은 문자열 부분을 후속 처리를 위해 include 된 URLconf로 전달합니다.
>
>include()에 숨은 아이디어 덕분에 URL을 쉽게 연결할 수 있습니다. polls 앱에 그 자체의 URLconf(polls/urls.py)가 존재하는 한, "/polls/", 또는 "/fun_polls/", "/content/polls/"와 같은 경로, 또는 그 어떤 다른 root 경로에 연결하더라도, 앱은 여전히 잘 동작할 것입니다.

잘 동작하는지 다시 한번 확인해보자

```
$ python manage.py runserver
```

서버를 띄우고 http://127.0.0.1:8000/polls/ 로 들어가면 아까 뷰에서 작성한 문구가 보인다.

### path()

polls/urls.py에서 path 함수는 필수 인자인 route, view 그리고 선택 가능 인자인 kwargs와 name 모두 4개의 인자를 받았다. 각 인자가 무엇인지 간단히 요약하고 넘어가자.

#### route
URL 패턴을 가진 인자. 요청을 처리할 때 urlpatterns의 첫번째 부터 시작해서 일치하는 패턴을 찾는다. 패턴은 params나 도메인 이름을 검색하지 않는다.
예를 들어 **http://saturnkim.dev/happy/** 가 요청된 경우 URLconf는 **happy/** 만 보고 찾는다.

#### view
일치하는 패턴을 찾으면 HttpRequest 객체를 첫번째 인자로, 경로에서 '캡춰된' 값을 키워드 인수로 특정 view 함수를 호출한다.

#### kwargs
아까는 쓰이지 않았는데 나중에 나오면 덧붙여 정리하도록 하겠음.

#### name
URL에 이름을 붙이면 장고 어디에서나 명확하게 참조할 수 있다. 파일 하나만 수정해도 project 내 모든 URL 패턴을 바꿀 수 있게 도와준다.
좋은 기능인 것 같은데 아직 확 와닿지는 않는다.



**part 2 에서 계속**
