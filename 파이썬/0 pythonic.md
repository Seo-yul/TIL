# pythonic!

# **lambda 함수**

- 익명함수 : 메모리를 아끼고 가독성을 향상시킨다. pythonic
- 일반적인 함수는 객체를 만들고, 재사용을 위해 함수 이름(메모리)를 할당한다.

```python
# lambda 인수1, 인수2, ... : 인수를 이용한 표현식
>>> sum = lambda a, b: a+b
>>> sum(3,4)
7
```

## **lambda는 왜 쓰는가?**

- 익명함수이기 때문에 한번 쓰이고 다음줄로 넘어가면 힙(heap) 메모리 영역에서 증발
- (참고) 가비지 컬렉터 (참조하는 객체가 없으면 지워버린다)
    - 파이썬에서는 모든것이 객체로 관리 되고 각 객체들은 레퍼런스 카운터를 갖게 된다. 이 카운터가 0 즉, 그 누구도 참조하지 않게 된다면 메모리를 환원 하게 된다.

---

# **map 함수**

- 내장함수
- 입력받은 자료형의 각 요소가 합수에 의해 수행된 결과를 묶어서 map iterator 객체로 리턴

```
# 문법
map(f, iterable)
# 함수(f)와 반복 가능한 (iterable) 자료형을 입력으로 받는다.
```

## **map 예시문제**

- map으로 짜면 게으른 연산을 진행해서 메모리를 크게 절약할 수 있다.
- map의 연산 결과는 map iterator 객체로 리턴한다.

```
# Input : li = [1, 2, 3]
# Output : result = [1, 4, 9]

# 풀이
result = list(map(lambda i: i ** 2 , li))
```

## **(참고) 게으른 연산**

- 필요할 때 가져다 쓴다.(예: map함수의 결과 객체)
- iterator 객체
    - next() 메소드로 데이터를 순차적으로 호출 가능한 object
    - 마지막 데이터까지 불러 오면 다음은 StopIteration exception 발생
    - iterable한 객체를 iterator 로 변환하고 싶다면, iter() 라는 built-in function 을 사용
    - for 문으로 looping 하는 동안, python 내부에서는 임시로 list를 iterator로 자동 변환

```
# ex1.
li = [1, 2, 3]
result = map(lambda i: i * i, li)
next(result) # 1
next(result) # 4
next(result) # 9
next(result) # StopIteration 발생

# ex2.
>>> x = [1,2,3]
>>> type(x)
<class 'list'>
>>> y = iter(x)
>>> type(y)
<class 'list_iterator'>
```

---

# **3항 연산자**

```
# if else문
def func(a):
    if a > 10:
        return 'a가 10보다 크다'
    else:
        return 'a가 10보다 작다'

# 3항 연산자
def func2(a):
    return 'a가 10보다 크다' if a > 10 else 'a가 10보다 작다'
```

## **3항 연산자 예시문제**

- 조건이 3개일 경우에도 3항 연산자 사용이 가능하다.

```
#Input : li = [-3, -2, 0, 6, 8]
#Output : ['음수', '음수', 0, '양수', '양수']

#풀이 1
>>> ['양수' if i > 0 else ('음수' if i < 0 else 0)  for i in li]
>>> ['음수', '음수', 0, '양수', '양수']

#풀이 2
>>> list(map(lambda i: '양수' if i > 0 else ('음수' if i < 0 else 0), li))
>>> ['음수', '음수', 0, '양수', '양수']
```

---

# **filter 함수**

- filter(함수, literable)
- 두번째 인수인 반복 가능한 자료형 요소들을 첫번째 인자 함수에 하나씩 입력하여 `리턴값이 참인 것만` 묶어서 돌려준다.
- 함수의 리턴 값은 참이나 거짓이어야 한다.

```
li = [-2, -3, 5, 6]

# 양수만 반환하는 리스트
def ft(li):
    result = []
    for e in li:
        if e > 0:
            result.append(e)
        else:
            pass
    return result

# filter 함수 사용시
def positive(x):
  return x > 0

list(filter(positive, li))

# filter + lambda 함수 사용시
list(filter(lambda x: x > 0, li))
```

---

# **reduce 함수**

- python3 부터 내장함수에서 빠짐
- reduce(function, iterable[, initializer])
- [문서](https://docs.python.org/3/library/functools.html#functools.reduce)

```
from functools import reduce
reduce(lambda x, y: x+y, [1, 2, 3, 4, 5])
# = ((((1+2)+3)+4)+5)
```

## **reduce 함수 예시문제 1**

```
from functools import reduce

# 문제1. 전통적으로 최대값 구하기
def maximum(li):
    default = 0
    for e in li:
        if default < e:
            default = e
    return default
maximum(li)

# 문제1. reduce 활용하여 최대값 구하기
reduce(lambda a,b: a if a > b else b ,li)
```

---

# **(참고) 문자별 갯수 구하기 알고리즘**

## **or, and**

```
>>> [1,2,3] and []
>>> []

>>> [1,2,3] or []
>>> [1,2,3]

>>> bool([1,2,3] or [])
>>> True
```

## **dictionary 관련함수**

- dict : dictionary 만들기
- .get() : key를 통하여 value 값 검색
- .update() : dictinary 요소 업데이트

```python
# dict
>>> dic = dict(one=1, two=2)
>>> print(dic)
>>> {'one': 1, 'two': 2}

# .get()
>>> dic['three'] # 해당 key 값이 없으면 KeyError
>>> dic.get('three') # 해당 key 값이 없으면 None
>>> dic.get('three', 3) # 해당 key 값이 없으면 & 디폴트 값이 있으면  디폴드 값 반환

# .update()
>>> dic.update({'three': 3}) # dictinary에 요소를 업데이트, 리턴 값은 None
>>> dic
>>> {'one': 1, 'three': 3, 'two': 2}

## update와 동시에 결과 값을 반환하고 싶을 때
>>> dic.update({'four': 4}) or dic
>>> {'four': 4, 'one': 1, 'three': 3, 'two': 2}
```

## **문제 (중요)**

```python
# Input : data = ['a', 'a', 'a', 'b', 'b', 'c', 'c', 'c']
# Output : {'a': 3, 'b': 2, 'c': 3}

# 풀이 1
>>> reduce(lambda a, b: a.update({b: a.get(b, 0) + 1}) or a,
        data,
        {}
      )
# reduce(lambda a, b: a.update({b: a.get(b, 0) + 1}) or a, data,{})

# 풀이 2
# 풀이 1이 풀이 2보다 더 효율적이다 (변수, itorator 객체 생성 메모리 절약)
>>>result =  { i : 0 for i in data} # {'a': 0, 'b': 0, 'c': 0}
>>>for i in data:
       result[i] += 1
>>>result
>>>{'a': 3, 'b': 2, 'c': 3}
```