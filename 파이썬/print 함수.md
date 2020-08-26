# print 함수

## 문법

print에서는 `''`작은 따옴표와 `""`큰 따옴표를 모두 사용해서 표현할 수 있다.

`print("""Hello Python""")` 과 같이 세번써 감싼 문자열도 출력할 수 있다.

### Separator 옵션 사용

`print('T', 'E', 'S', 'T', sep='')` 출력 `TEST`

`print('2020', '08', '21' sep='-')` 출력 `2020-08-21`

sep 옵션은 옵션 값으로 요소를 연결해준다.

### end 옵션 사용

`print('Welcome To', end=' ')`

`print('the black parade, end=' ')`

출력 `Welcome To the black parade`

end 옵션은 마지막 문자를 옵션 값으로 한다. 따라서 기본 줄바꿈이 사라진다.

### format 사용 [ ], { }, ( )

`print('{ } and { }'.format('You', 'Me'))` 출력 `You and Me`

`print('{0} and {1} and {0}'.format('You', 'Me'))` 출력 `You and Me and You`

`print('{a} and {b}'.format(a='You', b='Me'))` 출력 `You and Me`

%s : 문자, %d : 정수, %f : 실수

`print("%s's favorite number is %d" % ('reso', 1))` 출력 `reso's favorite number is 1`

`print("Test1: %5d, price: %4.2f" % (776, 6534.123))` 출력 `Test1: 776, price: 6534.12`

`print("Test1: {0: 5d}, price: {1: 4.2f}".format(776, 6534.123))` 출력 `Test1: 776, price: 6534.12`

`print("Test1: {a: 5d}, price: {b: 4.2f}".format(a=776, b=6534.123))` 출력 `Test1: 776, price: 6534.12`

## Escape 코드

```python
"""
\n : 개행
\t : 탭
\\ : 문자
\' : 문자
\" : 문자
\r : 캐리지 리턴
\f : 폼 피드
\a : 벨 소리
\b : 백 스페이스
\000 : 널 문자
"""
```