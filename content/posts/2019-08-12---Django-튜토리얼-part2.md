---
title: Django 튜토리얼 part2 
date: "2019-08-12T18:15:00.169+09:00"
template: "post"
draft: false
slug: "/posts/django-tutorial-part2/"
category: "python"
tags:
  - "django"
  - "장고"
description: "[PART2] 가장 많이 쓰이는 웹 어플리케이션 프레임워크인 장고 튜토리얼을 해보자"
---


## django
---
장고는 파이썬으로 만들어진 model-view-template 웹 프레임워크다. 전에 일하면서 간단한 api를 만들 일이 있었는데 그 때는 빠르게 작업하고 싶어서 Flask를 사용했다.

Flask는 필요한 부분을 하나씩 붙여나가며 만들어야하기 때문에 당시 api 작업하기에 알맞다고 생각해서 사용했다(도큐먼트는 정말 보기 힘들었다).

그래서 ~~도큐먼트가 더 이쁜~~ 이번에는 장고를 배워보고자 공식 문서에 있는 튜토리얼을 해보기로 했다.

공식 문서의 튜토리얼은 [이곳](https://docs.djangoproject.com/ko/2.2/intro/tutorial01/)에서 확인할 수 있다.

[**Django 튜토리얼 part1**](https://saturnkim.dev/posts/django-tutorial-part1/)

## part 2
---

### 데이터베이스 설치 및 선택

장고에서 데이터베이스는 SQLite가 기본 설정이다.
다른 데이터베이스를 사용하려면 해당 데이터베이스 바인딩을 설치하고 settgings.py DATABASES의 default의 값을 변경해주면 된다.

SQLite를 사용할 것이라면 DATABASES 변경없이 진행하면 된다.

나는 SQLite로 진행하기로 했으니 TIME_ZONE만 변경해주었다.

settings.py에서 서울로 바꿔주었다.

```
TIME_ZONE = 'Asia/Seoul'
```

settings.py의 상단에 INSTALLED_APPS라는 것이 있는데 현재 장고 프로젝트에 활성화된 앱들을 보여준다. 해당 앱들은 다른 프로젝트에서도 사용할 수 있다. 필요없는 앱은 주석처리하거나 삭제하면 된다.

기본 어플리케이션 중에 몇개는 하나 이상의 데이터베이스 테이블을 사용하기 때문에 테이블을 미리 만들어줘야 한다. migrate 명령어를 사용하면 INSTALLED_APPS의 설정과 settings.py의 데이터베이스 설정을 읽어 새로운 데이터베이스 테이블을 생성한다.

실행은 다음과 같다.

```
$ python manage.py migrate

Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying sessions.0001_initial... OK
```

migration이 적용되었다.


### model 만들기

polls/models.py에 모델을 작성한다.

```
import datetime

from django.db import models
from django.utils import timezone

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date pulished')

    def __str__(self):
        return self.question_text

    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self):
        return self.choice_text

```

Question 모델은 질문텍스트(question_text)와 발행일(pub_date), 2개의 필드를 가진다.
Choice 모델은 선택지(choice_text)와 투표(vote)를가지고 Question 모델을 foreign key로 가진다.

DB의 필드는 Field 클래스의 인스턴스로 표현되는데 CharFiled는 문자 필드, DateTimeField는 날짜/시간 필드이다. 몇몇 Field 클래스는 필수 인자가 필요한데, CharField같은 경우에는 max_length를 넣어줘야한다.

Field 클래스 생성자에 첫번째 인자로 human-readable 이름을 지정할 수 있는데 위에서는 Question.pub_date에 'date published'라는 이름을 지정해주었다. 첫번째 인자를 넣지 않으면 기계가 읽기 좋은 형식의 이름을 사용한다.

ForeignKey를 조금 덧붙여 설명하면 위에서 작성한 모델은 one-to-many지만 many-to-many, one-to-one등도 지원한다. foreign key 접근은 question.choice_set으로 가능하다.
장고는 '_id'를 붙여서 foreign key 필드를 자동 생성한다. 위 경우는 'question_id'가 생성된다.

이제 프로젝트에 polls 앱을 설치해보자.
앱을 프로젝트에 포함시키려면 settings.py에서 본 INSTALLED_APPS에 추가하면 된다.

PollsConfig 클래스는 polls/apps.py에 있으므로 'polls.apps.PollsConfig'를 INSTALLED_APPS에 추가한다.

```
# Application definition

INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

polls 앱이 프로젝트에 포함되었으니 마이그레이션을 하자.

```
$ python manage.py makemigrations polls

Migrations for 'polls':
  polls/migrations/0001_initial.py
    - Create model Question
    - Create model Choice

```

아까와 달리 makemigrations 명령을 실행했다. 이 명령어는 변경사항을 migration으로 저장한다. 변경사항을 디스크상 파일로 저장하며 필요하다면 해당 앱의 migrations 디렉토리에서 수동으로 변경사항을 조정할 수 있다.
튜토리얼의 마이그레이션은 polls/migrations/0001_initail.py에 저장되어 있다.

이제 migrate 을 해보자.

```
$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Rendering model states... DONE
  Applying polls.0001_initial... OK

```

아직 적용되지 않았던 마이그레이션을 수집해 실행했다.
모델에서 변경사항과 데이터베이스 스키마의 동기화가 이루어졌다.

모델 변경에서 기억해야할 3단계 지침은 아래와 같다.
1. 모델을 변경했다.
2. makemigrations 명령어로 변경사항을 마이그레이션으로 만든다.
3. migrate 명령어로 변경사항을 DB에 적용한다.


### API 경험하기

장고로 파이썬 쉘을 실행해서 방금 만든 모델을 DB API를 통해 조작해볼 수 있다.

```
$ python manage.py shell

```

장고를 사용해 쉘을 실행하면 파이썬 쉘에서 장고에서 동작하는 모든 명령을 실험해볼 수 있다.

```
Python 3.7.3 (default, Mar 27 2019, 16:54:48)
[Clang 4.0.1 (tags/RELEASE_401/final)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from polls.models import Choice, Question # 아까 만들었던 모델을 불러오자

>>> Question.objects.all() # 아직 아무것도 없다.
<QuerySet []>

>>> from django.utils import timezone
>>> q = Question(question_text="Hey jude", pub_date=timezone.now()) # 새로운 질문지를 만들자
>>> q.save()
>>> q.id
1

>>> q.question_text
'Hey jude'
>>> q.pub_date
datetime.datetime(2019, 8, 8, 11, 18, 8, 967054, tzinfo=<UTC>)

>>> q.question_text = "socker" # 이름을 바꿀 수 있다.
>>> q.save()
>>> q.question_text
'socker'
>>> Question.objects.all()
<QuerySet [<Question: socker>]>

>>> q = Question.objects.get(pk=1) # primary key가 1인것을 get!
>>> q.was_published_recently()
True

>>> q.choice_set.all() # 선택지는 아직 만들지 않아서 없다.
<QuerySet []>

# 선택지를 만들어보자
>>> q.choice_set.create(choice_text='very much', votes=0)
<Choice: very much>
>>> q.choice_set.create(choice_text='not much', votes=0)
<Choice: not much>

>>> c = q.choice_set.create(choice_text='just listen', votes=0) # 새로 생성한 선택지를 다른 변수에 할당할 수 있다.

>>> c.question
<Question: socker> # 질문지는 여전히 socker 인것을 확인할 수 있다.

>>> q.choice_set.all()
<QuerySet [<Choice: very much>, <Choice: not much>, <Choice: just listen>]>
>>> q.choice_set.count()
3

>>> current_year = timezone.now().year
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: very much>, <Choice: not much>, <Choice: just listen>]>

>>> c = q.choice_set.filter(choice_text__startswith='just listen')
>>> c.delete()
(1, {'polls.Choice': 1})

>>> q.choice_set.all()
<QuerySet [<Choice: very much>, <Choice: not much>]>
```


### 장고 관리자

장고는 관리자 페이지를 쉽게 만들 수 있다.

먼저 관리자를 만들어야한다.

```
Username: admin
Email address: admin@heaven.com
Password: *****
Password (again): *****
The password is too similar to the email address.
This password is too short. It must contain at least 8 characters.
This password is too common.
Bypass password validation and create user anyway? [y/N]: y

```

관리자를 만들었으니 서버를 켜고 관리자페이지에 들어가보자

```
$ python manage.py runserver
```

http://127.0.0.1:8000/admin/으로 접속할 수 있다.

[!django_admin](media/admin_index.png)

편집 가능한 그룹과 사용자를 볼 수 있다. 하지만 작업했던 poll 앱이 보이질 않는다.
그럼 보이게 하면 된다.

polls/admin.py에서 Question 객체가 관리 인터페이스를 가지고 있다고 알려주면 된다.

```
from django.contrib import admin

from .models import Question

admin.site.register(Question)

```

이제 다시 관리자페이지에 접속하면 polls 앱이 추가된 것을 볼 수 있다.


**part 3 에서 계속**
