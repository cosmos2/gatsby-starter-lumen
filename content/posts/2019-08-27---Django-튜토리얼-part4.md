---
title: Django 튜토리얼 part4 
date: "2019-08-24T23:47:00.169+09:00"
template: "post"
draft: false
slug: "/posts/django-tutorial-part4/"
category: "python"
tags:
  - "django"
  - "장고"
description: "[PART4] 가장 많이 쓰이는 웹 어플리케이션 프레임워크인 장고 튜토리얼을 해보자"
---


## django
---
장고는 파이썬으로 만들어진 model-view-template 웹 프레임워크다. 전에 일하면서 간단한 api를 만들 일이 있었는데 그 때는 빠르게 작업하고 싶어서 Flask를 사용했다.

Flask는 필요한 부분을 하나씩 붙여나가며 만들어야하기 때문에 당시 api 작업하기에 알맞다고 생각해서 사용했다(도큐먼트는 정말 보기 힘들었다).

그래서 ~~도큐먼트가 더 이쁜~~ 이번에는 장고를 배워보고자 공식 문서에 있는 튜토리얼을 해보기로 했다.

공식 문서의 튜토리얼은 [이곳](https://docs.djangoproject.com/ko/2.2/intro/tutorial01/)에서 확인할 수 있다.

[**Django 튜토리얼 part1**](https://saturnkim.dev/posts/django-tutorial-part1/)
[**Django 튜토리얼 part2**](https://saturnkim.dev/posts/django-tutorial-part2/)
[**Django 튜토리얼 part3**](https://saturnkim.dev/posts/django-tutorial-part3/)

## part 4
---

### 간단한 폼 만들기

detail.html 템플릿에 form을 추가해보자

```python
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
{% endfor %}
<input type="submit" value="Vote">
</form>
```

{% csrf_token %} 태그는 사이트간 요청 위조(Cross Site Request Forgeries)에 대항하기 위한 템플릿 태그다. 내부 URL로 향하는 모든 POST 요청에 해당 템플릿 태그를 사용하면 된다니 편리하다.



이제 vote 뷰가 동작하도록 실제 구현해보자.

```python
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except KeyError: # 아무것도 선택하지 않으면 request.POST['chice']에서 KeyError가 일어난다.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': 'You did not select at all',
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        
        return HttpResponseRedirect(reverse('polls:results', args=[question_id]))
        
```

마지막에 return 부분을 보면 HttpResponse가 아니라 HttpResponseRedirect를 리턴한다.
vote 뷰에서는 POST 요청을 처리하기 때문에 리다이렉트 시키는 HttpResponseRedirect 객체를 반환한다.

HttpResponseRedirect는 리다이렉트 시킬 경로를 인자로 받는데 full qualified URL(e.g. 'https://www.google.com/', 절대경로(e.g. '/vote/', 상대경로(e.g 'vote/') 3가지 방식으로 리다이렉트 시킬 path를 받는다. 상대경로의 경우엔 현재 path에 따라 full URL을 구현한다. 그리고 HttpResponseRedirect는 HTTP status code를 302를 반환한다.

그리고 HttpResponseRedirect 안에서 reverse()를 사용하고 있는데 이것은 URL을 하드코딩하지 않도록 도와준다.
reverse는 인자로 *viewname***,** *urlconf=None***,** *args=None***,** *kwargs=None***,** *current_app=None* 등을 받을 수 있다. viewname은 URL pattern name이나 호출 가능한 뷰 오브젝트를 사용할 수 있다.
예제에서는 URLconf를 이용하고 있다. reverse는 **'polls'**앱에서 urlpatterns 리스트에서 **'results'**를 를 찾아 전달받은 args인자와 함께 아래와 같은 문자열을 반환한다.

```
'/polls/1/results/'
```

최종적으로 reverse가 반환한 path로 results 뷰로 redirect된다.



이제 vote 뷰가 리다이텍트할 results 뷰와 템플릿을 작성해보자

**polls/views.py**

```python
from django.shortcuts import get_object_or_404, render


def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```



**polls/results.html**

```html
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```



이제 서버를 켜서 /polls/1/로 가서 투표를 해보자. 투표 결과가 반영된 results 페이지를 볼 수 있다.



### 제너릭 뷰로 바꾸어보기

지금까지는 함수형 뷰(Function Based View)로 작성했다. 이제 이걸 제너릭 뷰로 바꾸어보자.

제너릭뷰는 간단하고 반복되는 패턴들을 추상화하여 쉽고 적은 코드로도 뷰를 구현할 수 있도록 한 클래스 기반의 뷰이다.

detail 이나 results 뷰처럼 간단하고 중복되는 뷰를 바꾸어보자.



#### URLconf 수정

전에 작성했던 URLconf를 수정하자.

```python
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    # path('', views.index, name='index'),
    path('', views.IndexView.as_view(), name='index'), # 수정
    # path('<int:question_id>/', views.detail, name='detail'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'), # 수정
    # path('<int:question_id>/results/', views.results, name='results'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'), # 수정
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

index, detail, results의 뷰 이름이 바뀌고 패턴의 이름이 qeustions_id 에서 pk로 변경되었다.



#### view 수정

```python
#...
from django.views import generic
#...

# def index(request):
#     latest_question_list = Question.objects.order_by('-pub_date')[:5]
#     context = {
#         'latest_question_list': latest_question_list,
#     }
#     return render(request, 'polls/index.html', context)

class IndexView(generic.ListView): # 수정한 index view
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        return Question.objects.order_by('-pub_date')[:5]

# def detail(request, question_id):
#     question = get_object_or_404(Question, pk=question_id)
#     return render(request, 'polls/detail.html', {'question': question})

class DetailView(generic.DetailView): # 수정한 detail view
    model = Question
    template_name = 'polls/detail.html'

# def results(request, question_id):
#     question = get_object_or_404(Question, pk=question_id)
#     return render(request, 'polls/results.html', {'question': question})

class ResultsView(generic.DetailView): # 수정한 results view
    model = Question
    template_name = 'polls/results.html'

#...
```

수정되기 전 함수형 뷰를 주석처리하였다.
새로 작성한 제너릭 뷰를 보면 전보다 훨씬 짧고 간결해졌다.

ListView와 DetailView 두가지 제너릭 뷰를 사용했다.



*제너릭 뷰에 대한 자세한 내용은 해당문서를 [**참조**](https://docs.djangoproject.com/ko/2.2/topics/class-based-views/generic-display/)*



**part 4 에서 계속**