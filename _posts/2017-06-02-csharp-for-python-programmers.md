---
layout: post
title: C# for Python Programmers
published: false
---

한동안 Python을 주 언어로 사용해오다가 앞으로 C#도 필요하게 되어 Python을 기준으로 C#이 다른 점들을 정리해보았다.
Microsoft의 공식 문서인 [C# 프로그래밍 가이드](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/)를 주로 참고 하였다.


# Hello World


Python
```python
if __name__ == "__main__":
    print("Hello world")
```

C#
```cs
using System;
class Hello
{
    static void Main()
    {
        Console.WriteLine("Hello World")
    }
}
```

# 기본 문법

Python
```python
# Python은 #으로 코멘트

'''
따옴표 3개를 이용해
여러 줄을 코멘트
'''

# Python은 줄바꿈이 곧 구문 구분이고 들여쓰기로 블럭을 구분
def func():
    foo = 1
    if foo == 1:
        bar = 2
		
		
# 함수 리턴 타입, 인자 타입 지정 없음
def concat(first, second):
    return first + second


# 변수 타입 지정 없음
question = "what is the answer of the life, universe and everything?"
answer = "42"
result = concat(question, answer)


# if문
if question is not "" and answer is "42":
    return True
else:
    return False

	
# for
for i in range(0, 10):
    print(i)

# for로 컨테이너 순환
for item in items:
    print(item)

	
# exception
try:
    if not open():
        raise Exception()
except Exception as e:
    print(e)
finally:
    close()
```


C#

```cs
// C#은 //로 코멘트
/* 을 이용해 여러 줄 혹은 한 줄의 가운데 특정 부분을 코멘 */ 
// C#은 세미콜론을 사용해 명시적으로 구문을 구분하고 블럭은 중괄호
public void func()
{
    int foo = 1;
    if (foo == 1)
    {
        int bar = 2;
    }
}


// 함수 리턴 타입, 인자 타입 지정
string Concat(string first, string second)
{
    return first + second;
}

// 변수 타입 지정
string question = "what is the answer of the life, universe and everything?";
string answer = "42";
string result = Concat(question, answer);

// 선언과 동시에 값을 할당하는 경우 `var`키워드를 사용하여 타입 지정을 생략 가능
var question = "what is the answer of the life, universe and everything?";
var answer = "42";
var result = Concat(question, answer);


// if
if (question != "" && answer == "42")
{
    return true;
}
else
{
    return false;
}


// for
for (var i = 0; i< 10; i++)
{
    Console.WriteLine(i);
}

// foreach
foreach (var item in items)
{
    Console.WriteLine(item);
}


// exception
try
{
    if (!Open())
    {
        throw new Exception();
    }
}
catch (NullReferenceException e)
{
    Console.WriteLine(e);
}
finally
{
    Close();
}
```


# Array, List, Tuple, Dictionary

Python

```python
# Python은 배열을 잘 사용하지 않고 list를 사용한다.
item_list = [1, 2, 3]
first_item = item_list[0]


# Python list는 타입 종류에 상관없이 담을 수 있다.
any_list = [1, "str", Foo, Foo.bar()]

# Python은 콤마로 구분하여 나열하면 tuple이다.
sample_tuple = (1, "two", 3.0)

# Python dictionary는 별도의 타입 지정이 필요없고 타입을 섞어서 쓸 수 있다.
item_dict = {
    'first': 1,
    2: ['list'],
    'last': {'another': 'dict'}
}
first_value = item_dict['first']
second_value = item_dict[2]
```

C#

```cs
// C# Array는 크기가 고정이다.
int[] item_array = { 1, 2, 3};
int first_item = item_array[0];

// C# List는 원소 타입을 지정한다.
List<int> itemList = new List<int>[1, 2, 3];
int firstItem = itemList[0];

// C#은 Tuple은 타입을 지정한다.
var sampleTuple = new Tuple<int, string, float>(1, "two", 3.0f);

// C# Dictionary는 타입을 지정해야 한다.
Dictionary<int, string> tempDict = new Dictionary<int, string>
{
    { 1, "first" },
    { 2, "second" }
};
```


# Class, Interface

Python Class
```python
class MyClass :
    class_variable = 'my_class'


    # 생성자
    def __init__(self):
        # 인스턴스 변수에 self로 접근, 미리 선언할 필요 없음
        self.instance_variable = 'default'


    def hello(self, name):
        self.instance_variable = name
        return 'hello world'
```

C# Class
```cs
namespace SandBox
{
    public class MyClass
    {
        string instanceVariable;

        // 생성자
        public MyClass()
        {
            instanceVariable = "default";
        }


        public string Func(string name)
        {
            instanceVariable = name;
            return "hello world";
        }
    }
}
```


C#에는 `interface` 키워드를 사용해서 인터페이스를 정의할 수 있다. 인터페이스를 구현하는 모든 클래스 혹은 구조체는 인터페이스에서 지정한 서명과 일치하는 메서드에 대한 정의가 포함되어야 한다. 인터페이스 정의에서는 구현을 하지 않고 서명만 정의한다. 모든 메서드가 추상인 추상 클래스와 유사하다. C# 클래서는 다중 상속을 지원하지 않으므로 인터페이스를 활용해서 미리 정의한 여러 동작을 한 클래스에 포함시킬 수 있다.

```cs
public interface ISpeakable
{
    string Speak();
}

public class Dog : ISpeakable
{
    string name = "Dog";

    public string Speak()
    {
        return "Bark!";
    }
}
```


C#은 `abstract` 한정자를 제공하여 추상 클래스와 추상 메서드를 정의할 수 있다.

```cs
abstract class ShapesClass
{
    abstract public int Area();
}

class Square : ShapesClass
{
    int side = 0;

    public override int Area()
    {
        return side * side;
    }
}
```

C#은 `virtual` 키워드로 가상 메서드를 만들 수 있다. `virtual`로 지정된 메서드는 파생 클래스에서 `override`로 재정의할 수 있다. `virtual`로 지정되지 않은 메서드는 재정의 할 수 없다. `virtual`은 `static`, `abstract`, `private` 또는 `override` 키워드와 함께 사용할 수 없다.
```cs
class BaseClass
{
    int num;

    public virtual int Number()
    {
        return num;
    }
}

class DerivedClass : BaseClass
{
    public override int Number()
    {
        return num + 1;
    }
}
```


# Struct
C#에는 class와 구분되는 struct가 있다.
```cs
public struct CoOrds
{
    public int x, y;

    public CoOrds(int p1, p2)
    {
        x = p1;
        y = p2;
    }
}
```

구조체에 대해 매개 변수가 없는 기본 생성자를 정의하면 오류가 발생한다.구조체 본문에서 인스턴스 필드를 초기화해도 오류가 발생한다. 구조체의 멤버는 매개 변수가 있는 생성자를 사용하거나 구조체가 선언된 후 멤버에 개별적으로 액세스하는 방법으로만 초기화할 수 있다.

클래스와 달리 다른 구조체나 클래스를 상속할 수 없다.

구조체는 값 형식(value type)이다.

# Value Types vs. Reference Types

값 형식을 기반으로 한 변수에는 값이 직접 포함된다.값 형식 변수 하나를 다른 변수에 대입하면 값 자체의 복사가 일어난다. 참조 형식 변수의 경우 참조만 복사되는 것과 대비된다.

structs, enumerations, numeric types, bool 등이 값 형식이다.
class, interface, delegate, dynamic, object, string 등이 참조 형식이다.

## Boxing, Unboxing
값 형식을 `object` 형식 또는 이 값 형식에서 구현된 임의의 인터페이스 형식으로 변환하는 것을 `boxing`이라고 한다.
boxing이 일어날 때 값을 System.Object안에 wrapping하고 heap에 저장한다. unboxing은 이 object에서 값 형식을 꺼낸다. boxing은 암시적이고 unboxing은 명시적이다.

단순 할당에서는 boxing과 unboxing을 수행하는 데 많은 계산 과정이 필요하다. 값 형식을 boxing할 때는 새로운 개체를 할당하고 생성한다.


```cs
int i = 123;     // i는 value type
object o = i;    // boxing
int j = (int)o;  // unboxing
```

![](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/types/media/vcunboxingconversion.gif)

boxing된 개체는 원래 값 형식과 개별 메모리 위치를 사용하므로 서로 다른 값을 저장하게 된다.
```cs
class TestBoxing
{
    static void Main()
    {
        int i = 123;

        // Boxing copies the value of i into object o.
        object o = i;

        // Change the value of i.
        i = 456;

        // The change in i does not effect the value stored in o.
        System.Console.WriteLine("The value-type value = {0}", i);
        System.Console.WriteLine("The object-type value = {0}", o);
    }
}
/* Output:
    The value-type value = 456
    The object-type value = 123
*/
```

# Property
Python은 `property`라는 built-in function을 제공한다.
```python
class C :
    def __init__(self):
        self._x = None

    def getx(self):
        return self._x

    def setx(self, value):
        self._x = value

    def delx(self):
        del self._x

    x = property(getx, setx, delx, "I'm the x property.")
```

데코레이터 형태로 사용하는 것이 일반적이다.
```python
class C :
    def __init__(self):
        self._x = None

    @property
    def x(self):
        return self._x

    @x.setter
    def x(self, value):
        self._x = value

    @x.deleter
    def x(self):
        del self._x
```

C#은 전용 문법을 제공한다.
```cs
class C
{
    string _x;
    
    public string x
    {
        get { return _x; }
        set { _x = value; }
    }
}
```

C# 3 부터는 기본 속성 구현은 자동으로 할 수 있다.
```cs
class C
{
    public string x { get; set; }
}
```

C# 6 부터는 읽기 전용 속성에서 get 접근자를 키워드 없이 본문 멤버로 구현할 수 있다.
```cs
class C
{
    string firstName;
    string lastName;

    public string Name => $"{firstName} {lastName}";
}
```


# Lambda
Python의 Lambda는 `lambda` 키워드를 사용한다.
```python
items = [10, 15, 5]
items.sort(key=lambda x: 1 / x)
```

C#의 Lambda는 `=>` 연산자를 사용한다.
```cs
(x, y) => x == y
```


# Delegate
Python에서는 추가적인 코드 없이 함수를 인자로 넘기거나 리턴으로 넘길 수 있다.
```python
def my_adder(x, y):
    return x + y

def executor(func, arg1, arg2):
    return func(arg1, arg2)

executor(my_adder, 1, 2)
```


C#에서도 함수를 넘길 수 있으나 타입 지정이 필요하기 때문에 함수를 넘겨 받으려면 해당 함수의 시그너처를 본딴 `delegate`가 필요하다.
```cs
// delegate 선언
public delegate int Func(int first, int second);
// delegate와 시그너쳐가 동일한 함수 정의
public int MyAdder(int first, int second)
{
    return first + second;
}

public int Executor(Func func, int arg1, int arg2)
{
    return func(arg1, arg2);
}

Executor(new Func(MyAdder), 1, 2);
```

# `out`, `ref` Parameter
C#에서는 `out`, `ref` 키워드로 매개변수를 참조로 전달할 수 있다. 함수 내에서 해당 매개변수의 값을 변경 혹은 할당하고 그 내용을 호출한 쪽에서 유지할 수 있다.

```cs
void Adder(ref int value)
{
    value += 1;
}

int value = 10;
Adder(ref value);
// value == 11
```

```cs
void Setter(out int value)
{
    value = 1;
}

int value = 0;
Setter(out value);
// value == 1
```

# Named Parameter (Keyword Argument)

Python은 keyword argement를 지원한다. 이 기능을 통해 가독성을 향상시킬 수 있고, 선택적으로 매개 변수를 전달할 수 있다.

```python
def func(a, b, c, d):
    print(a, b, c, d)

func(a=1, b=2, c=3, d=4)
```

여러 개의 keyword argument를 **을 이용해서 한번에 받을 수도 있다.
```python
def func(**kwargs):
    print(kwargs['a'], kwargs['b'])


func(a=1, b=2, c=3, d=4)
```

매개 변수 이름을 지정함으로써 순서에 상관없이 선택적으로 매개변수를 넘길 수 있다.
```python
def func(a, b, c=10, d=20, e=30):
    print(a, b, c, d, e)

# c와 d는 default값을 사용하고 e만 지정하고 싶은 경우
func(0, 1, e=100)
```

C#에서도 동일한 기능을 지원하고 있다.
```cs
public int CalcBMI(int weight, int height)
{
    // ...
}

int bmi = CalcBMI(weight: 60, height: 180);
```

Python에서와 마찬가지로 매개 변수 명을 지정해줄 경우 함수 시그니처 상 매개변수 순서는 상관 없다.
```cs
// 다음 두 호출은 동일하다
int bmi = CalcBMI(weight: 60, height: 180);
int bmi = CalcBMI(height: 180, weight: 60);
```

역시 마찬가지로 선택적 인수 전달을 위해서 사용할 수 있다.
```cs
public int CalcBMI(int weight = 60, int height = 180)
{
    // ...
}

// weight는 default값을 사용하고 height만 185로 넘겨주고 싶은 경우
int bmi = CalcBMI(height: 185);
```

# `import` vs. `using`
Python은 폴더, 파일 단위가 패키지, 모듈이 되고 그대로 namespace를 구성한다. 패키지와 모듈은 `import`, `from ... import ...` 형태로 불러온다.

```python
import module1

module1.func1()
```

C#은 명시적으로 namespce를 지정한다.
```cs
namespace SampleNamespace
{
    class SampleClass
    {
        public void Method()
        {
            // ...
        }
    }
}

SampleNamespace.SampleClass.Method();
```
C#에서는 `using`을 사용해서 namespace 사용을 간소화할 수 있다.
```cs
using SampleNamespace;

SampleClass.Method();
```

# Generator, IEnumerable
Python은 `yield` 가 포함된 함수는 `generator`가 된다.
```python
def test():
    for i in range(10):
        yield i


for res in test():
    print(res)
```

C#은 `yield break` 혹은 `yield return`을 이용해서 비슷한 동작을 할 수 있고 `IEnumerable`가 된다.
```cs
public IEnumerable<int> Test()
{
    for (var i = 0; i < 10; ++i)
    {
        yield return i;    
    }
}
foreach (var i in Test())
{
    Console.WriteLine(i);
}
```

# LINQ, comprehension

C#은 LINQ(Language Integrated Query)라는 쿼리 기능을 제공한다.

```cs
using System;
using System.Linq;
using System.Collections.Generic;

class app {
  static void Main() {
    string[] names = { "Burke", "Connor", "Frank", 
                       "Everett", "Albert", "George", 
                       "Harris", "David" };

    IEnumerable<string> query = from s in names 
                               where s.Length == 5
                               orderby s
                               select s.ToUpper();

    foreach (string item in query)
      Console.WriteLine(item);
  }
}
```

이 프로그램을 실행하면, 다음과 같은 출력을 얻을 수 있다.

```
BURKE
DAVID
FRANK
```


Lambda나 delegate를 사용해서 메서드 방식으로도 쿼리할 수 있다.

```cs
IEnumerable<string> query = names
	.Where(name => name.Length == 5)
	.OrderBy(name => name)
	.Select(name => name.ToUpper());
```


Python에서는 comprehension을 사용해서 유사한 코드를 작성할 수 있다.

```python
names = ["Burke", "Connor", "Frank", "Everett", "Albert", "George", "Harris", "David"]
query = sorted([s.upper() for s in names if len(s) == 5])
for s in query:
    print(s)
```



# Reflection

C#에서는 reflection을 사용해서 어셈블리, 모듈 및 타입을 설명하는 `Type` 타입의 오브젝트를 가져올 수 있다.
타입의 인스턴스를 동적으로 만들거나, 타입을 기존 오브젝트에 바인딩하거나, 기존 오브젝트에서 형식을 가져오고 해당 메서드를 호출하거나 해당 필드와 속성에 액세스할 수 있다.
코드에서 attribute를 사용하는 경우 reflection을 통해 해당 attribute에 접근할 수 있다.

```cs
// Using GetType to obtain type information:  
int i = 42;  
System.Type type = i.GetType();  
System.Console.WriteLine(type);

// 출력:
// System.Int32
```

문서에 따르면 reflection의 일반적인 용도는 다음과 같다.

- Assembly를 사용하여 어셈블리를 정의 및 로드하고, 어셈블리 매니페스트에 나열된 모듈을 로드하고, 이 어셈블리에서 형식을 찾아 해당 인스턴스를 만듭니다.
- Module을 사용하여 모듈 및 모듈의 클래스가 포함된 어셈블리와 같은 정보를 검색합니다. 모듈에 정의된 모든 전역 메서드나 기타 특정 비전역 메서드를 가져올 수도 있습니다.
- ConstructorInfo를 사용하여 생성자의 이름, 매개 변수, 액세스 한정자(예: public 또는 private), 구현 세부 정보(예: abstract 또는 virtual)와 같은 정보를 검색합니다. Type의 GetConstructors 또는 GetConstructor 메서드를 사용하여 특정 생성자를 호출합니다.
- MethodInfo를 사용하여 메서드의 이름, 반환 형식, 매개 변수, 액세스 한정자(예: public 또는 private), 구현 세부 정보(예: abstract 또는 virtual)와 같은 정보를 검색합니다. Type의 GetMethods 또는 GetMethod 메서드를 사용하여 특정 메서드를 호출합니다.
- FieldInfo를 사용하여 필드의 이름, 액세스 한정자(예: public 또는 private), 구현 세부 정보(예: static)를 검색하고 필드 값을 가져오거나 설정합니다.
- EventInfo를 사용하여 이벤트의 이름, 이벤트 처리기 데이터 형식, 사용자 지정 특성, 선언 형식, 리플렉션 형식과 같은 정보를 검색하고 이벤트 처리기를 추가하거나 제거합니다.
- PropertyInfo를 사용하여 속성의 이름, 데이터 형식, 선언 형식, 리플렉션 형식, 읽기 전용/쓰기 가능 상태와 같은 정보를 검색하고 속성 값을 가져오거나 설정합니다.
- ParameterInfo를 사용하여 매개 변수 이름, 데이터 형식, 매개 변수가 입력 또는 출력 매개 변수인지 여부, 메서드 서명에서 매개 변수의 위치와 같은 정보를 검색합니다.
- CustomAttributeData를 사용하여 응용 프로그램 도메인의 리플렉션 전용 컨텍스트에서 작업할 때 사용자 지정 특성에 대한 정보를 검색합니다. CustomAttributeData를 사용하면 특성 인스턴스를 만들지 않고 특성을 검사할 수 있습니다.


Python은 동적 언어이기 때문에 reflection 개념이 없고 타입, 메서드, 필드 정보 등은 자연스럽게 접근 가능하다.


# Attribute

C#은 Attribuute 기능을 지원한다.`[`, `]`를 사용한다.

```cs
[Custom("I'm custom attribute of Foo!")]
class Foo
{
}
```

Custom Attribute 선언과 적용 예제는 다음과 같다.

```cs
using System;

class CustomAttribute: Attribute
{
	string name;

	public CustomAttribute(string name)
	{
		this.name = name;
	}

	public override string ToString()
	{
		return name;
	}
}

[Custom("I'm custom attribute of Foo!")]
class Foo
{
}
```
	
적용된 특성에 접근하려면 reflection을 사용해야한다.

```cs
using System.Reflection;
using System.Linq;

MemberInfo info = typeof(Foo);
Attribute attribute = info.GetCustomAttributes().First();
Console.WriteLine(attribute.GetType());
Console.WriteLine(attribute);

// 출력
// CustomAttribute
// I'm custom attribute of Foo!
```
