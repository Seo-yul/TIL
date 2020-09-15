파이썬은 콜바이오브젝트 형식을 따르지만 다른 언어와 같이 참조형의 특징을 가지는 경우가 있다.

## __eq__와 id 비교의 경우

```python
x = {'name':'kim', 'age':20}
y = x

print(id(x), id(y)) # 3186614069352 3186614069352 로 똑같이 출력 된다.
print(x == y) # True
print(x is y) # True

x['city': 'seoul'] # 둘 다 추가됨
print(x, y)
# {'name': 'kim', 'age': 20, 'city': 'seoul'} {'name': 'kim', 'age': 20, 'city': 'seoul'}
```

## tuple 자료형 불변형 비교

```python
# 일반적인 튜플은 불변형으로 같은 내용이면 같은 주소(id), 변경이 생기면 새로운 객체를 할당 받음
# 리스트가 포함된 튜플은 다른 아이디를 할당 받는다.
tuple1 = (10, [100, 1000])
tuple2 = (10, [100, 1000])

print(id(tuple1), id(tuple2)) # 1875953386312 1875953386248
print(tuple1 is tuple2)       # False
print(tuple1 == tuple2)       # True
print(tuple1.__eq__(tuple2))  # True
```

## Copy, Deepcopy (얕은 복사, 깊은 복사)

파이썬에서 변수의 객체 할당은 링크를 통한 얕은 복사를 의미한다.

```python
 # 장바구니
class Basket:
    def __init__(self, products=None):
        if products is None:
            self._products = []
        else:
            self._products = list(products)

    def put_prod(self, prod_name):
        self._products.append(prod_name)

    def del_prod(self, prod_name):
        self._products.remove(prod_name)

import copy

basket1 = Basket(['Apple', 'Bag', 'TV', 'Snack', 'Water'])
basket2 = copy.copy(basket1)
basket3 = copy.deepcopy(basket1)

print(id(basket1), id(basket2), id(basket3))
# 1838998816328 1838998816520 1838998829896
print(id(basket1._products), id(basket2._products), id(basket3._products))
# 1838998816392 1838998816392 1838998778952

basket1.put_prod('Orange')
basket2.del_prod('Snack')

print(basket1._products)
# ['Apple', 'Bag', 'TV', 'Water', 'Orange']
print(basket2._products)
# ['Apple', 'Bag', 'TV', 'Water', 'Orange']
print(basket3._products)
# ['Apple', 'Bag', 'TV', 'Snack', 'Water']
```

## 파이썬 불변형 예외

```
str, bytes, frozenset, Tuple : 사본 생성 X -> 참조 반환
```

메모리 낭비를 막아주는 방법으로 불변형이 완전히 같은 내용을 참조하는 경우 같은 id를 가진다.