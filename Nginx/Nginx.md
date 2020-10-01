# Nginx

## 상용 기본 명령어 및 설정

`nginx` : 기동

`nginx -s stop` : 정지

`nginx -s reload` : 재기동

`nginx -t`  설정파일 체크

`/etd/nginx` : main 설정파일 위치

`/etc/nginx/conf.d/` : 각 서버용 설정파일 위치

## Nginx 명령의 옵션

`-?, -h` : nginx 명령어의 도움말 표시

`-v` : nginx Version

`-V` : nginx를 make 했을시 컴파일러 또는 configure 옵션 표시

`-t` : nginx 설정파일 오류 체크 (오류 체크시 nginx 정지 상태에서 실행, 정상메세지 출력)

`-q` : nginx 설정파일 오류 체크 (오류 체크시 nginx 정지 상태에서 실행, 에러메시지 출력)

`-s stop` : 강제종료

`-s quit` : 실행중 request 처리 종료하고 nginx 정리

`-s reload` : 설정파일 다시 읽음

`-s reopen` : nginx 재기동중 로그파일을 다시 오픈

`-c 설정파일` : 지정한 설정파일로 nginx 기동

`-g global디렉티스 설정` : 지정한 global 디렉티브의 설정으로 nginx를 기동 (부하실험 등)

`-p prefix` : default=/usr/local/nginx

## Nginx의 환경 설정 파일

환경 설정 파일은 관리자가 편집하거나 프로그램으로 분석할 수 있는 텍스트 파일로, 여러 가지 값을 지정해 프로그램의 작동 방법을 결정

Nginx의 장점 중 하나로 환경 설정이 간단하다.

## 지시어 설정

nginx의 환경 설정 파일은 논리적인 지시어 목록이다. 애플리케이션 전체가 지시어의 값에 따라 자동한다.

nginc의 conf 파일 위치는 `/usr/local/nginx/conf/nginx.conf` 가 기본값이다.

```yaml
#user nobody;
worker_processes 1;
```

`#user nobody` : 주석 줄

`worker_processes` : 지시어

첫 비트는 설정키를, 그 뒤에는 한 개 이상의 값을 붙일 수 있다. 여기서는 첫 비트는 worker_processes 뒤 값 1로 ngingx 가 단일 작업 프로세스로 작동됨을 지시한다.

기본적인 지시어는 nginx core module에 포함되어 있다.

### 구조와 인클루드

```yaml
include mimn.types;
```

이름이 의미하는것과 같이 `include` 지시어는 특정 파일을 포함하는 기능을 수행. 지시어가 있는 바로 그 위치에 해당 파일 내용이 삽입된다.

```yaml
nginx.conf:
		user nginx nginx;
		worker_processes 4;
		include other_settings.conf;

other_setting.conf:
		error_log logs/error.log;
		pid logs/nginx.pid;
```

`include` 지시어는 `파일명 글로빙(filename globbing)`을 지원한다. 즉, 파일명에 와일드 카드를 사용할 수 있다.

### 지시어 블록

지시어는 모듈에 의해 도입되므로 새 모듈을 활성화하면 그 모듈에 포함된 지시어들을 사용할 수 있다.

모듈은 환경 설정을 논리적 구조로 만들 수 있게 지시어 블록(directive block) 기능을 제공한다.

```yaml
events {
			worker_connections 1024;
}
```

기본 환경 설정 파일에서 볼 수 있는 events 블록은 이벤트 모듈에 의해 도입된다.

모듈이 제공하는 지시어들은 블록 안에서만 사용할 수 있다. `worker_connections` 는 `events` 블록 안에서만 의미를 가진다. 하지만 중요한 예외로, 일부 지시어들은 서버의 전 범위에 효력을 갖기 때문에 메인 블록이라 부르는 환경 설정 파일의 루트에 놓일 수 있다.

경우에 따라 한 블록 안에 또 다른 블록이 삽입 될 수 있다.

```yaml
http {
		server {
					listen 80;
					server_name example.com;
					access_log /var/log/nginx/example.com.log;
					location ^~ /admin/ {
							 index index.php;
          }
     }
}
```

http 블록 안에는 한 개 이상의 서버 블록을 선언할 수 있다. 하나의 `server 블록은 하나의 가상 호스트를 구성한다.` 위에서 server 블록은 [example.com](http://example.com)과 일치하는 Host HTTP 헤더를 갖는 모든 요청에 적용될 환경 설정을 포함한다.

또한, `server 블록 안에 한 개 이상의 location 블록을 삽입할 수 있다.` 로케이션 블록은 요청 URI가 지정 경로와 일치할 경우에만 설정을 적용한다.

환경 설정은 자식 블록(children block)에 상속된다. 위에서 server 블록 레벨에 정의된 `access_log 지시어는 서버가 수신하는 모든 HTTP 요청을 텍스트 파일에 기록한다.`

다시 access_log 지시어를 사용해 로그를 중지시키지 않는 한 자식 블록안에서도 유효하다.

```yaml
[...]
location ^~ /admin/ {
			index index.php;
			access_log off;
}
[...]
```

위의 경우 /admin/ 위치 경로를 제외한 웹사이트의 모든 곳에 적용된다. server 블록 레벨에서 적용한 access_log 지시어의 설정 값은 location 블록 레벨에서 설정한 값에 의해 변경됨

### 지시어 값에서 사용되는 약자

지시어 값에서 파일 크기를 지정할 때 약자를 사용할 수 있다.

- k 또는 k 킬로바이트
- m 또는 M 메가바이트

```yaml
# 아래 두 의미는 같다.
client_max_body_size 2M;
client_max_body_size 2048k;
```

시간 값을 지정하는 단축 문자

- ms 밀리초
- s 초
- m 분
- h 시
- d 일
- w 주
- M 월(30일)
- y 연(365일)

기본 시간단위는 초이다.

```yaml
# 아래 세 의미는 같다.
client_body_timeout 3m;
client_body_timeout 180s;
client_body_timeout 180;
```

### 변수

모듈은 지시어 값을 정의할 때 활용할 수 있는 변수를 제공한다. 예를들어 nginx HTTP core module에서는 $nginx_version 변수를 정의한다. log_format 지시어를 설정할 때 포맷 문자열 안에 모든 종류의 변수를 포함할 수 있다.

```yaml
[...]
		 location ^~ /admin/ {
          access_log logs/main.log;
          log_format main '$pid - $nginx_version - $remote_addr';
     }
[...]
단, 일부 지시어에서 변수 사용을 허용하지 않는 경우가 있다.
```

### 문자열

지시어 값으로 사용되는 문자열은 세가지 형태로 나뉜다.

- 따옴표를 사용하지 않는 경우
- 큰따옴표를 사용하는 경우
- 작은따옴표를 사용하는 경우
- 단, nginx 는 큰따옴표와 작은따옴표를 구분하지 않는다!

```yaml
root /home/example.com/www;
root '/home/example.com/my web pages';
# 공백이나, 세미콜론, 중괄호를 사용하기 위해서는 따옴표 안에 지시어 값을 넣어야 한다.
```

## 기본 모듈 지시어

기본 모듈은 엔진엑스의 기본 기능의 매개변수를 정의하는 지시어를 제공한다.

기본 모듈은 컴파일 시 제외할 수 없기 때문에 기본 모듈이 제공하는 지시어와 블록은 항상 사용 가능하다.

- 코어 모듈(Core module) : 프로세스 관리나 보안과 같은 필수적인 기능과 지시어
- 이벤트 모듈(Events module) : 네트워크 기능의 내부 작동 방식을 구성
- 환경 설정 모듈(Configuration module) : 인클루드 체계를 사용할 수 있게 한다.

### 엔진엑스 프로세스 구조

엔진엑스의 실행이 막 시작된 순간에는 마스터 프로세스 한개만 존재한다. 마스터 프로세스 자신은 어떤 클라이언트 요청도 처리하지 않으며 단지 작업(worker) 프로세스를 증식(invoke) 시킨다. 이때, 워커 프로세스의 사용자/그룹은 바뀐다. 환경 설정에서 worker 프로세스의 수, worker 프로세스 당 최대 접속 수 등을 정할 수 있다. (마치 docker swarm과 같이)

## 코어 모듈 지시어

코어 모듈 지시어는 환경 설정 파일 루트에 넣어야 하며 오직 한번만 사용한다. 일부는 여러가지 문맥으로 유효하게 사용할 수 있는데, 그 경우 환경 설정 파일의 루트에 지시어명과 함께 유효 문맥의 목록이 표시되며, 이때도 한번만 사용할 수 있다.

### 1. error_log (main, http, server, location)

```
error_log /file/path level;
```

상세 로그 레벨부터 차례로 `debug, info, notice, warn, error, crit`

애플리케이션, HTTP 서버, 가상 호스트, 가상 호스트 디렉토리에 대해 각각 다른 레벨로 로그를 설정할 수 있다.

로그 출력 방향을 /dev/null로 전환하면 에러 로그를 해제하는 것과 같다. 환경 설정 파일의 루트에 다음과 같이 사용

```
error_log /dev/null crit;
```

### 2. master_process

```
master_process on;
```

on으로 설정하면 엔진엑스는 다수의 프로세스(하나의 마스터 프로세스와 여러개의 워커 프로세스)를 시작할 수 있다. off로 해제하면 단일 프로세스만 작동

### 3. thread_stack_size

```
thread_stack_size 1m;
```

스레드 스택의 크기를 정의

### 4. timer_resolution

```
timer_resolution 100ms;
```

내부 클럭 동기화를 위해 gettimeofday() 시스템 호출 사이의 간격을 제어한다. 이 값을 지정하지 않으면 커널 이벤트 알림이 있을 때마다 클럭이 동기화된다.

### 5. worker_threads

```
worker_threads 8;
```

워커 프로세스당 스레드 수를 정의한다. 스레드는 기본적으로 사용이 해제되어있다. 이유_엔진엑스는 경량 프로세스로 구현되었으며, 멀티스레드를 완전히 지원하지 않음

### 6. worker_cpu_affinty

```
worker_cpu_affinitty 1000 0100 0010 0001;
worker_cpu_affinitty 10 10 01 01;
worker_cpu_affinitty;
```

worker_process와 결합하여 작동하는 지시어로 CPU 코어와 워커 프로세스의 관계에 영향을 준다. 워커 프로세스 수만큼 숫자가 나열되며, 각 숫자의 자리 수는 CPU 코어 수와 같다.

CPU affinty(친화성)은 멀티코어 CPU를 위해 권한하는 것일 뿐 하이퍼 스레딩과 같은 기술을 사용하는 CPU를 대상이 아님

### 7. worker_priority

```
worker_priority 0;
```

일반적인 프로세스 우선순위 설정과 같다. -20 ~ 19 까지의 숫자로 워커 프로세스의 우선 순위를 정한다. 기본 값은 0. 커널 프로세스는 우선순위 -5 수준에서 실행되므로 우선순위를 -6 이상으로 설정 할 것을 권장

### 8. worker_processes

기본값 1로 워커 프로세스 수를 정의한다. 엔진엑스는 클라이언트 요청을 여러 개의 프로세스로 나눠 처리한다. auto 값 설정을 권장 코어만큼 증가.

I/O 동작이 느려 프로세스가 기다리는 상태가 되면 이때 들어오는 요청은 다른 워커 프로세스에 위임될 수 있음

### 9. worker_rlimit_core

```
worker_rlimit_core 100m;
```

워커 프로세스당 코어 파일의 크기 정의

### 10. worker_rlimit_nofile

```
worker_rlimit_nofile 10000;
```

워커 프로세스가 동시에 사용할 수 있는 파일의 수 정의

### 11. worker_rlimit_sigpending

```
worker_rlimit_sigpending 10000;
```

호출 프로세스의 사용자ID 당 대기할 수 있는 시그널 수 정의. queue(대기열)가 꽉 차면 한계치를 넘은 시그널은 버려진다.

## 이벤트 모듈 지시어