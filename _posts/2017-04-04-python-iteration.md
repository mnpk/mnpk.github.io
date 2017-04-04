---
layout: post
title: Python의 반복
---
> 이 글은 한빛미디어에서 출판된 [Fluent Python - 전문가를 위한 파이썬](http://www.aladin.co.kr/shop/wproduct.aspx?ItemId=88728476)의 `14장 "반복형, 반복자, 제네레이터"` 부분을 발췌 정리한 내용입니다.

# iter()

파이썬 인터프리터가 x 객체를 iterate해야 할 때는 언제나 iter(x)를 자동으로 호출한다.

iter() 내장 함수는 다음 과정을 수행한다.

1. 객체가 `__iter__()` 메서드를 구현하는지 확인하고, 이 메서드를 호출해서 iterator를 가져온다.
2. `__iter__()` 메서드가 구현되어 있지 않지만 `__getitem__()`이 구현되어 있다면, 파이썬은 인덱스 0에서 시작해서 항목을 순서대로 가져오는 iterator를 생성한다.
3. 이 과정이 모두 실패하면 파이썬은 `TypeError: C object is not iterable`이라는 메시지와 함께 `TypeError`가 발생한다.


# Iterable #1

sequence는 iterable이다.

모든 파이썬 시퀀스는 `__getitem__()`을 구현하고 있다. 위 2번 조건에 의해서 `__getitem__()`이 구현되어 있다면 iterable이다. 따라서 모든 파이썬 시퀀스는 iterable하다.

```python
items = [1, 2, 3]

for i in items:  # iteration
    print(i)
```


이를 class로 흉내 내면 다음과 같다.

```python
class Foo:
    def __init__(self, items):
        self.items = items

    def __getitem__(self, index):
        return self.items[index]

    def __len__(self):
        return len(self.items)
```

Foo 객체는 `__getitem__()` 을 구현하기 때문에 sequence이고 따라서 iterable이다.

```python
foo = Foo([1, 2, 3])
for item in foo:
    print(item)
```

`__iter__()` 메서드가 없어도 `__getitem__()`을 구현하는 객체에 iter 내장함수가 호출되면 iterator가 생성된다.

```python
it = iter(foo)
print(it)  # <iterator object at 0x00000220B6D87A20>
```

`__iter__()` 메서드를 구현하는 객체뿐만 아니라 정수형 키를 받는 `__getitem__()` 메서드를 구현하는 객체도 iterable로 간주하는 것은 덕 타이핑의 극단적인 형태다.

abc.Iterable 클래스가 `__subclasshook__()` 메서드를 구현하고 있으므로 상속이나 등록은 필요 없이 `__iter__()` 메서드를 구현하고 있다면 abc.Iterable 서브클래스로 판단된다.


```python
>>> class Foo:
...     def __iter__(self):
...         pass
...
>>> from collections import abc
>>> issubclass(Foo, abc.Iterable)
True
>>> f = Foo()
>>> isinstance(f, abc.Iterable)
True
```

그러나 `__getitem__()` 메서드만 구현하는 객체는 `issubclass(x, abc.Iterable)` 테스트를 통과하지 못한다.

객체를 iterate하기 전에 그 객체가 iterable인지 명시적으로 테스트할 필요는 없다. iterable하지 않은 객체를 iterate하려고 시도하면 파이썬이 `TypeError: 'C' object is not iterable.` 이라고 명료한 메시지를 담은 예외를 발생시키기 때문이다.


# Iterable #2

`iter()` 내장 함수가 iterator를 가져올 수 있는 모든 객체와 iterator를 반환하는 `__iter__()` 메서드를 구현하는 객체는 iterable이다.


for 대신 while로 흉내를 내면 다음과 같다.


```python
# iter()는 iterator를 만든다.
it = iter(foo)

while True:
    try:
        # iterator에 next()를 호출해서 다음 항목을 가져온다.
        print(next(it))
    except StopIteration:
        # 더 이상 항목이 없으면 iterator가 StopIteration Exception을 발생시킨다.
        del it
        break
```

# Iterator


Iterator에 대한 표준 인터페이스는 다음 두 개의 메서드를 정의한다.

`__next__()`
다음 항목을 리턴한다. 더 이상 항목이 없으면 StopIteration을 발생시킨다.

`__iter__()`
self를 리턴한다.


abc.Iterator 구현은 다음과 같다.

```python
class Iterator(Iterable):
    __slot__ = ()

    @abstractmethod
    def __next__(self):
        raise StopIteration

    def __iter__(self):
        return self

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Iterator:
            if (any("__next__" in B.__dict__ for B in C.__mro__) and
                any("__iter__" in B.__dict__ for B in C.__mro__)):
                return True
        return NotImplemented
```



# Iterable, Iterator

다시 정리하면,

- iterable
    - `__iter__()`메서드를 구현한다. iterator를 리턴한다.

- iterator
    - `__next__()`메서드를 구현한다. 다음 항목을 리턴한다. 다음 항목이 없으면 StopIteration을 발생시킨다.
    - `__iter__()`메서드를 구현한다. self를 리턴한다.


모든 iterator는 `__iter__()`를 구현하므로 iterable이기도 하다.

iterable은 `__next__()`를 구현하지 않으므로 iterator가 아니다.

iterable의 `__iter__()`는 새로운 iterator를 생성해서 리턴한다.

iterator의 `__iter__()`는 새로운 iterator가 아닌 자기 자신 self를 리턴한다.


예제로 구현해보면 다음과 같다.


```python
class Foo:
    def __init__(self, items):
        self.items = items

    def __iter__(self):  # __iter__()를 구현한다. Foo는 iterable이다.
        return FooIterator(self.items)

class FooIterator:
    def __init__(self, items):
        self.items = items
        self.index = 0

    def __next__(self):  # __next__()를 구현한다.
        try:
            item = self.items[self.index]
        except IndexError:
            raise StopIteration()  # 다음 아이템이 없으면 StopIteration을 발생
        self.index += 1
        return item

    def __iter__(self):  # self를 리턴하는 __iter__()를 구현한다.
        return self


foo = Foo([1,2,3])
it = iter(foo)  # FooIterator를 생성
assert isinstance(it, FooIterator)
```


# Generator

동일한 기능을 파이썬스럽게 구현하려면 generator함수를 사용한다.


```python
class Foo:
    def __init__(self, items):
        self.items = items

    def __iter__(self):
        for item in self.items:
            yield item
        return


foo = Foo([1,2,3])
it = iter(foo)
```

본문 안에 yield 키워드를 가진 함수는 모두 generator 함수이다.
generator 함수는 호출되면 generator 객체를 리턴한다.
즉 generator 함수는 generator 팩토리라고 할 수 있다.

`__iter__()` 는 generator 함수이기 때문에 호출하면 generator를 리턴한다.


generator의 작동 방식은 다음 예를 통해 확인할 수 있다.

```python
def gen_123():
    print('first')
    yield 1
    print('second')
    yield 2
    print('third')
    yield 3


print(gen_123)  # <function gen_123 at 0x0000017732DC4E18>
f = gen_123()
print(f)  # <generator object gen_123 at 0x0000017732F44F10>
```

`gen_123` 은 함수다. `get_123()`은 generator 객체이다.

generator 객체는 `__next__()` 메서드와 `__iter__()`메서드를 구현하는 iterator이다.

```python
>>> g = gen_123()
>>> hasattr(g, '__next__')
True  # __next__() 를 구현한다.

>>> >>> hasattr(g, '__iter__')
True  # __iter__() 를 구현한다.

>>> iter(g) == g
True  # __iter__()는 self를 리턴한다.
```


next()를 generator 객체에 호출하면 함수 본체에 있는 다음 yield로 진행하며 생성된 값을 평가한다.
함수 본체가 리턴될 때 `StopIteration`을 발생시킨다.

```python
>>> g = gen_123()
>>> next(g)
first
1
>>> next(g)
second
2
>>> next(g)
third
3
>>> next(g)
StopIteration
```




# yield from

다른 generator에서 생성된 값을 상위 generator 함수가 생성해야 할 때는 전통적으로 중첩된 for 루프를 사용했다.

```python
def chain(*iterables):
    for it in iterables:
        print(it)
        for i in it:
            print('yield', i)
            yield i


s = 'ABC'
t = tuple(range(3))
print(list(chain(s, t)))
```

```bash
ABC
yield A
yield B
yield C
(0, 1, 2)
yield 0
yield 1
yield 2
['A', 'B', 'C', 0, 1, 2]
```


`yield from`은 파이썬 3.3에서 추가된 문법이다. ([PEP 380](https://www.python.org/dev/peps/pep-0380/) 참조)
다음과 같이 사용할 수 있다.

```python
def chain(*iterables):
    for it in iterables:
        print(it)
        yield from it

s = 'ABC'
t = tuple(range(3))

u = chain(s, t)
print(u)

print(list(u))
```

```bash
<generator object chain at 0x000002C5E48A4F10>
ABC
(0, 1, 2)
['A', 'B', 'C', 0, 1, 2]
```

`yield from`구문이 내부 for 루플르 완전히 대체했다.

`yield from`은 단순히 루프를 대체하는 일 외에도 외부 generator의 호출자와 내부 generator를 연결하는 통로를 만든다. generator를 coroutine으로 사용해서 호출자 코드에서 가져온 값을 소비하는 경우에는 이 통로가 중요해진다. coroutine을 살펴볼 때 다시 자세히 알아보도록 하자.


# callable_iterator

어떤 객체 x를 반복해야할 때 `iter(x)`를 호출한다.

그러나 `iter()`는 일반 함수나 callable 객체로부터 iterator를 생성하기 위해서 두 개의 인수를 전달해서 호출할 수도 있다. 이렇게 사용하려면 첫 번째 인자는 값을 생성할 함수 또는 callable이어야 하며(source), 두 번째 인자는 구분 표시(sentinel) - 이 값이 리턴되면 `StopIteration`이 발생함 - 이어야 한다.

```python
from random import randint

def d6():
    return randint(1, 6)

d6_iter = iter(d6, 1)
print(d6_iter)

for roll in d6_iter:
    print(roll)
```
```bash
<callable_iterator object at 0x0000019CBC177C50>
6
4
6
5
6
```

`d6_iter`는 `callable_iterator`이다.
sentinel이 1이므로 randint 결과 1이 나오면 반복이 멈춘다.


