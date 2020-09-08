# 2 Class 데코레이터

## Class Method

@를 사용하는 클레스 데코레이터를 이용한다. `@classmethod` 파이썬 인터프리터에게 알려주는것

클래스 메소드는 self로 인자를 받는것이 아닌 공용 변수 이기때문에 cls로 받는다. 파라미터는 일반 함수와 같다.

```python
class class_name(object):
	class_var = 0
	
	def __init__(self, arg_one, arg_two):
			self._arg_one = arg_one
			self.__arg_two = arg_two

	# 모든 인스턴스가 접근하는 클래스 변수를 조작한다.
	@classmethod
	def method_name(cls, arg):
		cls.class_var = arg    # 클래스 변수를 조작한다.
```

### class method를 이용하여 생성자를 만드는 방법

```python
@classmethod
def method_const(cls, arg_one, arg_two):
	return cls(arg_one, arg_two)
```

## Static Method

클래스 자체 혹은 인스턴스 모두에서 접근이 가능하다.

`@staticmethod`