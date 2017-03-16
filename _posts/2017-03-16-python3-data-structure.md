---
layout: post
title: Python3의 흥미로운 데이터 구조들
published: false
---
> 이 글은 [python reddit](https://www.reddit.com/r/Python/comments/5zfsq4/new_interesting_data_structures_in_python_3/)에서 발견한 [글](https://github.com/topper-123/Articles/blob/master/New-interesting-data-types-in-Python3.rst)의 한글 번역입니다.


Python 3는 더 이상 새로운 것이 아닙니다.

실제로 최근에 탄생 3000일을 [축하](https://www.reddit.com/r/Python/comments/5v0tt6/python_3_created_via_pep_3000_is_exactly_3000/)하기도 했으니까요:) 꽤 오랜 기다림이 있었지만 Python 3는 무섭게 성장하고 있습니다. 그래서 Python 3에 새롭게 제공되는 데이터 구조는 어떤게 있는지 한 번 알아보려고 합니다.

`types.MappingProxyType`, `typing.NamedTuple` 그리고 `types.SimpleNamespace` 를 살펴보겠습니다.

모두 Python 3에서 추가되었습니다.

### types.MappingProxyType

`types.MappingProxyType`은 읽기 전용 dict로 사용되며 Python 3.3에서 추가되었습니다.
자세한 내용은 [문서](https://docs.python.org/3/library/types.html#types.MappingProxyType)를 참고하세요.

`types.MappingProxyType` 는 읽기 전용 이라 데이터를 직접 조작할 수 없습니다. 만약 데이터를 변경을 하고 싶다면 의도적으로 사본을 만들어서 사본을 변경해야합니다. dict 구조의 데이터와 그 데이터를 참조하는 consumer가 있고 consumer가 실수로 원본 데이터를 변경해버리는 것을 방지하고 싶을 때 완벽한 데이터 구조입니다. 실제로 사용해보면 매우 유용한 경우가 많습니다.

다음은 `read_only` 가 직접 수정되지 않는 것을 보여주는 예제입니다.
```python
>>> from  types import MappingProxyType
>>> data = {'a': 1, 'b':2}
>>> read_only = MappingProxyType(data)
>>> del read_only['a']
TypeError: 'mappingproxy' object does not support item deletion

>>> read_only['a'] = 3
TypeError: 'mappingproxy' object does not support item assignment
```

데이터 dict를 다른 함수나 쓰레드에 전달하는데 데이터를 변경하지는 못하게 하고 싶다면, 원본 dict 대신 `MappingProxyType` 오브젝트를 전달하면 의도적이지 않은 수정을 할 수 없게 됩니다. 이런 사용법을 보여주는 예제는 다음과 같습니다.

```python
>>> def my_threaded_func(in_dict):
>>>    ...
>>>    in_dict['a'] *= 10  # 앗, 버그가.. 전달받은 in_dict를 변경해버렸습니다.

...
# in some function/thread:
>>> my_threaded_func(data)
>>> data
data = {'a': 10, 'b':2}  # 함수 호출로 인해 data['a']가 변경되어버렸습니다.
```

이 경우에 `my_threaded_func`에 dict가 아닌 mapping proxy를 전달하면 오류가 발생하게 됩니다.

```python
>>> my_threaded_func(MappingProxyType(data))
TypeError: 'mappingproxy' object does not support item deletion
```

이제 `my_threaded_func`에서 먼저 `in_dict`의 사본을 만든 후 사본을 변경하는 식으로 코드를 바꾸어서 오류를 수정해야함을 알 수 있습니다. mappingproxy의 이런 기능은 발견하기 어려운 버그를 미리 피할 수 있게 해줍니다.


**읽기 전용은 읽기 전용이지 불변(immutable)은 아닙니다.** 따라서 `data`를 변경하면 `read_only`도 같이 변경됩니다.

```python
>>> data['a'] = 3
>>> data['c'] = 4
>>> read_only  # changed!
mappingproxy({'a': 3, 'b': 2, 'c': 4})
```


### typing.NamedTuple

`typing.NamedTuple`은 유서 깊은`collections.namedtuple`가 슈퍼차지된 버전입니다. Python 3.5에서 추가되었지만 실제로는 Python 3.6에서 나왔다고 볼 수 있습니다.

`collections.namedtuple`에 비교하면 `typing.NamedTuple`는:

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

저는 기존의 함수 기반의 문법 보다 클래스 기반의 새로운 문법이 더 좋습니다. 가독성도 좋고요.

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

요약하면, 이 현대화된 버전의 namedtuple은 훨씬 멋져졌고 앞으로 namedtuple의 표준으로 자리잡을 것이 분명합니다.


### types.SimpleNamespace

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

간단히 말해서 `types.SimpleNamespace`은 매우 단순한 클래스이며 속성을 지정, 변경, 삭제할 수 있으며 훌륭한 repr 출력을 제공합니다. 저는 가끔 dict 대용으로 편하게 읽고 쓰는데 사용하거나, 혹은 유연한 인스턴스화와 repr 출력을 얻기 위해서 상속해서 사용하곤 합니다.

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

`types.SimpleNamespace`를 이렇게 상속해서 사용하는 것은 아주 대단한 일은 아니지만, 보통 코드 몇 줄을 아껴줄 수 있을겁니다. 그것도 멋진 일이죠.

### 결론

Python 3의 새로운 데이터 구조들에 대한 간단한 연습을 여러분도 즐기셨길 바랍니다.


> 이 글은 [python reddit](https://www.reddit.com/r/Python/comments/5zfsq4/new_interesting_data_structures_in_python_3/)에서 발견한 [글](https://github.com/topper-123/Articles/blob/master/New-interesting-data-types-in-Python3.rst)의 한글 번역입니다.