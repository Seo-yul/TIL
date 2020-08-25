# 함수 및 람다(lambda)

# 람다

## 함수 정의 방법

`def 함수명(parameter):`

## 함수 호출

`함수명(parameter)`

## 람다의 특징

- 람다식 : 메모리 절약, 가독성 향상, 코드 간결
- 함수는 객체 생성 → 리소스(메모리) 할당
- 람다는 즉시 실행(Heap 초기화) → 메모리 초기화

## 함수 정의와 호출 예제

```python
def hello(world):
	print('Hello ', world)

hello('python')

def hello_return(world):
	val = "Hello " + str(world)
	return val
```

# 함수의 다중 리턴

```python
def func_mul(x):
	y1 = x * 100
	y2 = x * 200
	y3 = x * 300
	return y1, y2, y3

val1, val2, val3 = func_mul(100)
# 10000 20000 30000
```

# *args, **kwargs

parameter가 몇개인지 모르는 상황에서 받는 방법이다.

```python
def args_func(*args):
	# 다중 파라미터는 인덱스를 가진 튜플로 넘어오게 된다.
	# 순회 할 수 있기 때문에 for나 enumerate등 시퀀스를 사용가능
	for i in enumerate(args):
			print(i)

# 함수 호출
args_func('kim', 'lee', 'park')
'''
결과
kim
lee
park
'''

# kwargs(키워드) 사용의 경우 딕셔너리 형태로 받는다.
def kwargs_func(**kwargs):
		for k, v in kwargs.items():
				print(k, v)

# 함수 호출
kwargs_func(name1='kim', name2='park', name3='lee')
```

# 중첩함수(클로저)

함수안에 함수가 있는 상태

변수를 안에서만 사용함. 메모리 관리가 더 효율적이다.

```python
def nested_func(num):
		def func_in_func(num):
				print(num)
		print('in func')
		func_in_func(num+10000)

nested_func(10000)
```

# 데코레이터

파이썬 공부 더 한후 구글링해서 공부 할 것!

# 파이썬 함수의 힌트

함수의 파라미터와 리턴값을 알려줄 수 있다.

```python
def func_mul(x : int) -> list:
	y1 = x * 100
	y2 = x * 200
	y3 = x * 300
	return [y1, y2, y3]
```