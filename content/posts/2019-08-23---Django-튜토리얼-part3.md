---
title: Django 튜토리얼 part3 
date: "2019-08-23T23:58:00.169+09:00"
template: "post"
draft: false
slug: "/posts/django-tutorial-part3/"
category: "python"
tags:
  - "django"
  - "장고"
description: "[PART3] 가장 많이 쓰이는 웹 어플리케이션 프레임워크인 장고 튜토리얼을 해보자"
---


## django
---
장고는 파이썬으로 만들어진 model-view-template 웹 프레임워크다. 전에 일하면서 간단한 api를 만들 일이 있었는데 그 때는 빠르게 작업하고 싶어서 Flask를 사용했다.

Flask는 필요한 부분을 하나씩 붙여나가며 만들어야하기 때문에 당시 api 작업하기에 알맞다고 생각해서 사용했다(도큐먼트는 정말 보기 힘들었다).

그래서 ~~도큐먼트가 더 이쁜~~ 이번에는 장고를 배워보고자 공식 문서에 있는 튜토리얼을 해보기로 했다.

공식 문서의 튜토리얼은 [이곳](https://docs.djangoproject.com/ko/2.2/intro/tutorial01/)에서 확인할 수 있다.

[**Django 튜토리얼 part1**](https://saturnkim.dev/posts/django-tutorial-part1/)
[**Django 튜토리얼 part2**](https://saturnkim.dev/posts/django-tutorial-part2/)

## part 3
---

### 뷰를 추가하자

전에 작업하던 polls 앱에 뷰를 추가해서 뭔가 다른 것들을 보여주자.

먼저 테스트 삼아 간단하게 request와 함께 question_id를 받아서 HttpResponse 객체를 반환하도록 해보자.

```
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

그리고 polls.urls 모듈에 뷰와 연결될 path를 설정하자.

```
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

이건 어떻게 동작하는 걸까?
예를 들어 '/polls/666/'을 사용자가 요청한다고 하자. 
1. 장고에서 ROOT_URLCONF 설정에 의해 mysite.urls 모듈을 불러온다.
2. mysite.urls에서 urlpatterns 변수를 찾아 순서대로 패턴을 검색하여 매칭되는 것을 찾는다.
3. 일치하는 텍스트를 제외한 나머지 텍스트('666/')을 polls.urls의 URLconf로 전달한다.
4. 위와 동일한 방식으로 urlpatterns에서 일치하는 것을 찾는다.
5. detail 뷰 함수가 호출된다.

path를 추가했으니 뷰를 조금 더 다듬어 보자.
지금은 뷰가 단순히 받아온 정보만 출력하고 있으니 모델에서 실제 결과를 가져오게끔 바꿔보자.

```
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
```

Question 모델에서 리스트를 가져와 반환하게끔 바꾸었다.
여기에서 TEMPLATES을 사용하도록 조금 더 바꾸어보자.

~프로젝트의 템플릿 설정은 mysite.settings의 TEMPLATES을 참고한다.~

polls 디렉토리에 templates라는 디렉토리를 만들고 그 안에 다시 polls 디렉토리를 만든 뒤, index.html 파일을 생성하자.
경로는 **polls/templates/polls/index.html** 이 될 것이다.

polls/templates에 바로 템플릿을 넣지 않는 이유는 동일한 이름의 템플릿이 다른 앱에 있는 경우 장고가 이를 구분하지 못하기 때문이다. 정확한 템플릿을 지정하기 위해 위 처럼 앱의 이름으로 된 디렉토리에 템플릿을 넣는다.

이제 템플릿을 작성해보자.

**polls/templates/polls/index.html**
```
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

템플릿을 이용해서 뷰의 index 함수를 다시 개조해보자.

```
from django.template import loader

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html') # polls/index.html 템플릿을 불러온다
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```

템플릿을 불러오고 템플릿에서 사용되는 변수명과 python 객체를 연결하기 위해 context를 만들어 전달했다. 하지만 더 간단한 방법이 있다.
render 함수를 사용하면 된다.

```
from django.shortcuts import render

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    # template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': latest_question_list,
    }
    # return HttpResponse(template.render(context, request))
    return render(request, 'polls/index.html', context)
```

HttpResponse 객체를 반환하는 것과 render를 사용하는데 차이를 보기 위해 이전 코드에서 사용하지 않는 줄을 주석처리 했다.

render 함수는 request 객체를 첫번째로 받고, 템플릿, 그리고 context 객체를 선택적으로 받을 수 있다. 그리고 render 함수를 통해 HttpResponse 객체가 반환된다.

템플릿을 사용할 것이라면 render 함수를 임포트하여 간단하게 사용할 수 있다.

### 404 에러 핸들링

뷰의 detail 함수를 가지고 404에러를 어떻게 핸들링 할지 알아보자.

1. try / except 사용하기

```
from django.http import Http404
# ...
def detail(request, question_id):
    try:
      question = Question.object.get(pk=question_id)
    except Question.DoesNotExist:
      raise Http404('No question')
    return render(request, 'polls/detail.html', {'question': question})
```

2. get_object_or_404
```
from django.shortcuts import get_object_or_404, render
#...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```
get_object_or_404는 장고 모델을 첫번째 인자로, 키워드 인자를 받아 모델 관리자의 get 함수에 넘긴다. 객체가 없는 경우 http404를 일으킨다.

~get_list_or_404도 비슷하게 동작하는데 get 대신 filter를 사용한다.~


이제 detail의 템플릿을 작성해보자.

```
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```

### URL 네임스페이스 설정

만약 polls 앱 외에도 여러 앱이 사용된다면 장고가 앱들의 URL을 구분하기 어려워질 수 있다. 예를 들어 detail이라는 뷰가 polls 앱과 blog 앱에 사용되면 어떻게 구분해야 할까?

방법은 URLconf에 네임스페이스를 추가하면 된다.

polls/urls.py에서 설정해보자.

```
from django.urls import path

from . import views

app_name = 'polls' # 네임스페이스 추가!
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

네임스페이스를 설정하면 템플릿에서 정확한 뷰를 지정해줄 수 있다.

index.html 템플릿을 수정하자.

```
{% comment %} 원래 polls/templates/polls/index.html {% endcomment %}
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>

{% comment %} 정확한 뷰를 지칭하게 변경 {% endcomment %}
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```


**part 4 에서 계속**
