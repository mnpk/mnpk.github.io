---
layout: post
title: (번역) Python 3 에서 주목할 만한 새로운 자료구조들
published: false
---
> 이 글은 [reddit/r/python](https://www.reddit.com/r/Python/comments/5zfsq4/new_interesting_data_structures_in_python_3/)에서 발견한 [글](https://github.com/topper-123/Articles/blob/master/New-interesting-data-types-in-Python3.rst)의 한글 번역입니다. 오역이 있을 수 있습니다.


Python 3는 더 이상 새로운 것이 아닙니다.

최근에 탄생 3000일을 [축하](https://www.reddit.com/r/Python/comments/5v0tt6/python_3_created_via_pep_3000_is_exactly_3000/)하기도 했으니까요. 꽤 오랜 기다림이 있었지만 Python 3는 드라마틱하게 성장하고 있습니다. 이제 Python 3에서 새롭게 제공되는 데이터 구조는 어떤게 있는지 한 번 알아볼 때가 된 것 같습니다.

`types.MappingProxyType`, `typing.NamedTuple` 그리고 `types.SimpleNamespace` 를 살펴보겠습니다.


# types.MappingProxyType

`types.MappingProxyType`은 읽기 전용 dict로 Python 3.3에서 추가되었습니다.
자세한 내용은 [문서](https://docs.python.org/3/library/types.html#types.MappingProxyType)를 참고하세요.

`types.MappingProxyType` 는 읽기 전용이므로 데이터를 직접 조작할 수 없습니다. 변경하려면 사본을 만들어야합니다. dict 구조의 데이터와 consumer가 존재하는 상황에서 실수로 원본 데이터를 변경되는 것을 방지하고 싶을 때 알맞은 데이터 구조입니다. 실제로 사용해보면 매우 유용한 경우가 많습니다.

다음 예에서 `read_only` 가 직접 수정되지 않는 것을 볼 수 있습니다.
```python
>>> from  types import MappingProxyType
>>> data = {'a': 1, 'b':2}
>>> read_only = MappingProxyType(data)
>>> del read_only['a']
TypeError: 'mappingproxy' object does not support item deletion

>>> read_only['a'] = 3
TypeError: 'mappingproxy' object does not support item assignment
```

데이터 dict를 다른 함수나 쓰레드에 전달할 때 데이터는 변경하지는 못하게 막고 싶다면, 원본 dict 대신 `MappingProxyType` 오브젝트를 전달하면 됩니다.

```python
>>> def my_threaded_func(in_dict):
>>>    ...
>>>    in_dict['a'] *= 10  # 앗, 전달받은 in_dict를 변경해버렸습니다.

...
# in some function/thread:
>>> my_threaded_func(data)
>>> data
data = {'a': 10, 'b':2}  # 함수 호출로 인해 data['a']가 변경되어버렸습니다.
```

이렇게 버그가 발생할 수 있는 경우에, `my_threaded_func`에 dict가 아닌 mapping proxy를 전달하면 버그 대신 오류가 발생하게 됩니다.

```python
>>> my_threaded_func(MappingProxyType(data))
TypeError: 'mappingproxy' object does not support item deletion
```

이제 `my_threaded_func`에서 먼저 `in_dict`의 사본을 만든 후 사본을 변경하는 식으로 코드를 바꾸어서 오류를 수정해야함을 발견할 수 있습니다. mappingproxy의 이런 기능은 발견하기 어려운 버그를 미리 피할 수 있게 해줍니다.


주의, **읽기 전용은 읽기 전용이지 불변(immutable)은 아닙니다.** 따라서 `data`를 변경하면 `read_only`도 같이 변경됩니다.

```python
>>> data['a'] = 3
>>> data['c'] = 4
>>> read_only  # changed!
mappingproxy({'a': 3, 'b': 2, 'c': 4})
```


# typing.NamedTuple

`typing.NamedTuple`은 유서 깊은`collections.namedtuple`가 훨씬 좋아진 버전입니다. Python 3.5에서 추가되었지만 실제로는 Python 3.6에서 나왔다고 볼 수 있습니다.

`collections.namedtuple`과 비교하면:

- 더 나은 문법
- 상속
- type annotations
- default 값 (python >= 3.6.1)

을 제공합니다.

```python
>>> from typing import NamedTuple
>>> class Student(NamedTuple):
>>>    name: str
>>>    address: str
>>>    age: int
>>>    sex: str

>>> tommy = Student(name='Tommy Johnson', address='Main street', age=22, sex='M')
>>> tommy
Student(name='Tommy Johnson', address='Main street', age=22, sex='M')
```

저는 기존의 함수 기반의 문법 보다 클래스 기반의 새로운 문법이 더 좋네요. 가독성도 좋고요.

일반적인 클래스 인스턴스가 아니라 실제로 tuple을 사용하고 있다는 점을 확인할 수 있습니다.


```python
>>> isinstance(tommy, tuple)
True
>>> tommy[0]
'Tommy Johnson'
```

Python 3.6.1부터 추가된 default 값을 이용해서 Student를 상속 받는 예제입니다.

```python
>>> class MaleStudent(Student):
>>>    sex: str = 'M'  # default value, requires Python >= 3.6.1

>>> MaleStudent(name='Tommy Johnson', address='Main street', age=22)
MaleStudent(name='Tommy Johnson', address='Main street', age=22, sex='M')  # note that sex defaults to 'M'
```

이 모던한 namedtuple은 기존 보다 훨씬 멋져졌고 향후 namedtuple의 표준으로 자리잡을 것이 분명합니다.


# types.SimpleNamespace

`types.SimpleNamespace`은 namespace 속성과 그에 대한 의미있는 repr을 제공하는 간단한 클래스입니다. Python 3.3에서 추가되었습니다.

```python
>>> from types import SimpleNamespace
>>> data = SimpleNamespace(a=1, b=2)
>>> data
namespace(a=1, b=2)
>>> data.c = 3
>>> data
namespace(a=1, b=2, c=3)
```

`types.SimpleNamespace`은 매우 단순한 클래스로 속성을 지정, 변경, 삭제할 수 있고 훌륭한 repr 출력을 제공합니다. 저는 가끔 dict 대용으로 편하게 읽고 쓰는데 사용하거나, 혹은 유연한 인스턴스화와 repr 출력을 얻기 위해서 상속해서 사용하곤 합니다.

```python
>>> import random
>>> class DataBag(SimpleNamespace):
>>>    def choice(self):
>>>        items = self.__dict__.items()
>>>        return random.choice(tuple(items))

>>> data_bag = DataBag(a=1, b=2)
>>> data_bag
DataBag(a=1, b=2)
>>> data_bag.choice()
(b, 2)
```

`types.SimpleNamespace`를 이렇게 상속해서 사용하는 것은 아주 대단한 일은 아니지만, 몇 줄의 코드를  아껴줄 수 있을겁니다. 그것도 멋진 일이죠.

