#  11 Concurrency - Future

- 동시성 개념
- 비동기 작업
- 쓰레드풀 실습
- 프로세스풀 실습
- 블록 / 논블록

서로 다른 스레드 또는 프로세스에서 실행가능하다.

내부 과정 알 필요 없으며, 고수준으로 인터페이스를 제공

```
from concurrent import future
```

**ProcessPoolExecutor : GIL 우회, 내부적으로 os.cpu_count() 를 사용**

**ThreadPoolExecutor: GIL 종속**

## Python GIL (Global Interpreter Lock)

GIL 은 한 번에 하나의 스레드만 수행 할 수 있게 인터프리터 자체에서 락을 거는 것.

예를들어 하나의 변수에 여러개의 스레드가 접근 하는 경우 데드락 같은게 발생할 수 있음

```python
from concurrent import future
worker =  10 #스레드수

# map 을 사용해 스레드로 함수를 실행한다.
with futures.ThreadPoolExecutor(worker) as excutor:
	# map -> 작업 순서를 유지하고 즉시 실행
	result_cnt = excutor.map(작업함수, 함수의 파라미터obj)
```

하나의 오브젝트에 접근해서 GIL 때문에 생기는 컨텍스트 스위칭 오버헤드를 우회하는 방법은 멀티 프로세싱 모듈을 사용하면 된다.

```python
from concurrent import future

# map 을 사용해 프로세스로 함수를 실행한다.
with futures.**ProcessPoolExecutor** () as excutor:
	# map -> 작업 순서를 유지하고 즉시 실행
	result_cnt = excutor.map(작업함수, 함수의 파라미터obj)
```

map 을 사용하지 않고 프로세스를 스케줄링 할 수 있다.

```python
futures_list = []
with futures.ProcessPoolExecutor() as excutor:
        # Submit -> Callable 객체 스케쥴링(실행 예약) -> Future
        # Future -> result(), done(), as_completed() 주로 사용
        for nt in sorted(NATION_LS):
            # future 반환
            future = excutor.submit(작업함수, 함수의 파라미터obj)
            # 스케쥴링
            futures_list.append(future)

				for future in futures.as_completed(futures_list):
						# 스케줄링을 통해 프로세스의 진행을 확인할 수 있다.
            result = future.result() # 실행 결과
            done = future.done() # 실행 성공? True 또는 False
            cancelled = future.cancelled # True
            # future 결과 확인
            print('Future Result : {}, Done : {}'.format(result, done))
            print('Future Cancelled : {}'.format(cancelled))
```