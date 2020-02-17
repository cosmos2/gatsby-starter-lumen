---
title: python super에 대하여
date: "2019-09-20T17:44:00.169+09:00"
template: "post"
draft: false
slug: "/posts/python-super/"
category: "python"
tags:
  - "python"
  - "super"
description: "python에서 상속할 때 사용하는 super()에 대해"
---


# super()

파이썬에서 부모 클래스의 메소드를 상속할 때 super()를 사용한다.

```python
class LoggingDict(dict):
    def __setitem__(self, key, value):
      logging.info('Setting %r to %r' % (key, value))
      super().__setitem__(key, value)
      # better than
      # dict.__setitem__(key, value)
```

위 예제를 보면 LoggingDict는 super()로 dict의  `` __setitem__`` 을 상속 받는다. ``dict.__setitem__(key, value)`` 를 사용하는 것 보다 super()를 사용하는 것이 왜 좋을까?


- 상속받는 클래스가 dict 에서 다른 Somthing 이란 클래스로 바뀌어도 super()를 사용하면 알아서 상속한다.
- 네이밍 없이 부모 클래스를 참고하기에 코드 유지보수가 쉬워진다.
- 런타임에 결정되기 때문에 우리는 따로 신경쓸 필요가 없다.



```python
class C(B):
    def method(self, arg):
      super().method(arg)
      # same as
      # super(C, self).method(arg)
```

이렇게 써도 된다.



LoggingDict처럼 단일 상속하는 경우보다 역동적인 실행환경에서 다중 상속을 지원할 때 super()의 유용함이 드러난다.



## 다이아몬드 다이어그램

![diamond_inheritance](/media/diamond_inheritance.png)


동일한 부모 클래스 A를 상속하는 클래스 B와 클래스 C를 상속하는 D가 있다.
클래스 D를 호출하면 어떻게 될까?



```python
class A:
  
   def __init__(self):
      print("Class A __init__()")

      
class B(A):

   def __init__(self):
      print("Class B __init__()")
      A.__init__(self)

      
class C(A):

   def __init__(self):
      print("Class C __init__()")
      A.__init__(self)



class D(B, C):

   def __init__(self):
      print("Class D __init__()")
      B.__init__(self)
      C.__init__(self)


test = D() # 어떻게 될까?
```



```
# 호출 결과
Class D __init__()
Class B __init__()
Class A __init__() # class A is called
Class C __init__()
Class A __init__() # class A is called
```



결과를 보면 클래스 A가 두번 호출이 된다.
이 문제는 super()를 사용하면 간단히 해결이 된다.



```python
class A:

   def __init__(self):
      print("Class A __init__()")
      super().__init__()


class B(A):

   def __init__(self):
      print("Class B __init__()")
      super().__init__()


class C(A):

   def __init__(self):
      print("Class C __init__()")
      super().__init__()


class D(B, C):

   def __init__(self):
      print("Class D __init__()")
      super().__init__()

test = D()
```



```
# super()를 사용한 호출 결과
Class D __init__()
Class B __init__()
Class C __init__()
Class A __init__()
```



직접 부모 클래스를 호출 할 때와 달리, super()를 통해 호출하면 상속구조를 따라 알아서 호출한다. 클래스의 상속 구조를 잘 모르더라도 알아서 호출 해 준다.



## MRO

메소드를 호출할 때 베이스 클래스를 찾아 가는 순서를 정하게 되는데 이를 **MRO(Method Resolution Order)**라고 한다.

가장 처음 예로 들었던 LoggingDict와 OrderedDict를 상속하는 새로운 클래스를 만들어 MRO를 살펴보자.

```python
class LoggingOD(LoggingDict, collections.OrderedDict):
  pass

>> pprint(LoggingOD.__mro__)
(<class '__main__.LoggingOD'>,
 <class '__main__.LoggingDict'>,
 <class 'collections.OrderedDict'>,
 <class 'dict'>,
 <class 'object'>)
```



클래스 호출 시, 호출된 해당 클래스, 해당 클래스의 베이스 클래스, 베이스 클래스의 베이스...마지막으로 모든 클래스의 베이스인 object가 호출될 때 까지 스퀸스가 담긴 튜플이 나온다.
시퀸스 순서는 **부모 클래스 전에 자식 클래스**, 부모가 여럿일 경우 클래스의 ``__base__``의 순서를 참고해 순서를 유지한다.



LoggingOD mro를 정리보면

- LoggingOD는 부모클래스인 LoggingDict와 OrderedDict에 선행한다.
- LoggingDict는 OrderedDict에 선행한다. 왜냐하면 ``LoggingOD.__base__``가 (LoggingDict, OrderedDict) 순서이기 때문에.
- dict는 LoggingDict, OrderedDict의 부모이기 때문에 두 클래스보다 나중에 온다.
- dict는 부모클래스인 object에 선행한다.



## 유연하게 인자 전달하기

``__setitem__`` 처럼 고정된 인자를 받는 메소드는 위치인자로 전달해도 괜찮다. 하지만 더 유연하게 인자를 전달하려면 키워드 인자로 넘겨주는 방법이 있다.



```python
class Shape:
    def __init__(self, shapename, **kwds):
        self.shapename = shapename
        super().__init__(**kwds)
        
class ColoredShape(Shape):
    def __init__(self, color, **kwds):
        self.color = color
        super().__init__(**kwds)
        

```



## target method를 확실하게 하는 법

방금 전 예를 보면 ``__init__``은  ``object`` 클래스도 가지고 있으니 상관없지만 임의로 만든 메서드는 최상단 클래스에 존재하지 않아 AttributeError가 날 수 있다.
``object`` 클래스가 호출되지 전에 타켓메서드를 보장해줄 수 있는 Root 클래스를 만들면 super()를 사용하는 콜을 전달하지 않고 막아준다.



```python
class Root:
    def draw(self):
        assert not hasattr(super(), 'draw')

class Shape(Root):
    def __init__(self, shapename, **kwds):
        self.shpename = shapename
        super().__init__(**kwds)
    def draw(self):
        print(f'Drawing. Setting shape to:{self.shpename}')
        super().draw()

class ColoredShape(Shape):
    def __init__(self, color, **kwds):
        self.color = color
        super().__init__(**kwds)
    def draw(self):
        print(f'Drawing. Setting color to: {self.color}')
        super().draw()


cs = ColoredShape(color='red', shapename='circle')


```



## super() 를 사용하지 않는 서드파티 클래스를 상속하는 경우

super()도 사용하지 않고 Root 클래스를 상속하지도 않는 제 3의 클래스를 상속해야 하는 경우에는 adapter 클래스를 만들어 상속하는 방법이 있다.



```python
class Moveable:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def draw(self):
        print('Drawing at position: ', self.x, self.y)

class MoveableAdapter(Root):
    def __init__(self, x, y, **kwds):
        self.moveable = Moveable(x, y)
        super().__init__(**kwds)

    def draw(self):
        self.moveable.draw()
        super().draw()

class MovableColoredShape(ColoredShape, MoveableAdapter):
    pass

```



하나는 adapter 클래스를 만들어 상속하고 다른 하나는 바로 Moveable 클래스를 상속한 뒤 mro를 확인하고 draw()를 실행해 보았다.



```python
# ...
class MovableColoredShape(ColoredShape, MoveableAdapter):
    pass
  
class MoveableColoredShapeNoAdapter(ColoredShape, Moveable):
    pass


ms_adapter = MovableColoredShape(color='blue', shapename='triangle', x=10, y=30)
ms_no_adapter = MoveableColoredShapeNoAdapter(color='blue', shapename='triangle', x=10, y=30)


```





```bash
# adapter 로 상속
>> pprint(MovableColoredShape.__mro__)
(<class '__main__.MovableColoredShape'>,
 <class '__main__.ColoredShape'>,
 <class '__main__.Shape'>,
 <class '__main__.MoveableAdapter'>,
 <class '__main__.Root'>,
 <class 'object'>)

>> ms_adapter.draw()
Drawing. Setting color to: blue
Drawing. Setting shape to:triangle
Drawing at position:  10 30

# adapter 없이 Moveable 상속
>> pprint(ms_no_adapter)
(<class '__main__.MovableColoredShapeWithoutAdapter'>,
 <class '__main__.ColoredShape'>,
 <class '__main__.Shape'>, 
 class '__main__.Root'>,
 <class '__main__.Moveable'>,
 <class 'object'>)

>> ms_no_adapter.draw()
Drawing. Setting color to: blue
Drawing. Setting shape to:triangle
Traceback (most recent call last):
  File "suepr_test.py", line 50, in <module>
    mss.draw()
  File "suepr_test.py", line 19, in draw
    super().draw()
  File "suepr_test.py", line 11, in draw
    super().draw()
  File "suepr_test.py", line 3, in draw
    assert not hasattr(super(), 'draw')
AssertionError
```



adapter 클래스 없이 Moveable 클래스를 상속하면 mro에서 root 뒤로 Moveable이 오는 것을 확인 할 수 있다.
Shape 클래스에서 super()로 draw()를 호출하면 Root의 draw()가 호출된다(Root는 Root를 상속하지 않는 draw()가 호출될 것을 막고 타켓 메서드를 보장하기 위한 클래스기 때문에 assertion을 해놓았다). 그리고 Root의 draw()에서 super()로 호출된 Moveable에는 draw attribute가 있기 때문에 AssertionError이 난다.
