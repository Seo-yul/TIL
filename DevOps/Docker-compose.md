# docker-compose

## install

```bash
# 공식 문서
# <https://docs.docker.com/compose/install/>

# 공식 github에서 다운로드
sudo curl -L "<https://github.com/docker/compose/releases/download/1.27.2/docker-compose-$>(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 실행 권한 부여
sudo chmod +x /usr/local/bin/docker-compose
# 도커 버전 확인으로 설치를 확인
docker-compose version
```

## docker-compose 명령으로 컨테이너를 실행

### 작업 디렉터리 및 docker-compose.yml 파일을 생성

mkdir ~/compose && cd ~/compose

```yaml
# docker-compose.yml
version: "3"                          # 문법 버전
services:            
  echo:                               # 컨테이너 이름
    image: docker.io/mysql       # 컨테이너 생성에 사용할 도커 이미지 
    ports:
      - 3306:3306                     # 포트 포워딩 : 호스트:컨테이너
    environment:                      # 컨테이너 내부의 환경 변수
      - MYSQL_ROOT_PASSWORD=password
	app:
	  image: docker.io/tomcat
	  ports:
	    - 9090:8080
	
	web:
	  image: docker.io/nginx
	  ports:
	    - 9000:80
```

### 컨테이너 실행

```
docker-compose up
```

### 컨테이너 실행 확인

`docker container ls` : 확인

### 컨테이너 Scaling 하기

`docker-compose scale web=3` : yml에 web으로 지정된 컨테이너를 3개 복제

### 컨테이너 중지

```
docker-compose down
```

주의 : 중지와 함께 삭제도 수행된다.

## 이미지를 만들고 compose로 컨테이너 실행

### Dockerfile을 이용해 이미지를 빌드 후 실행되도록 docker-compose.yml 수정

```bash
# Dockerfile
FROM golang:1.9
RUN mkdir /echo
COPY main.go /echo
CMD [ "go", "run", "/echo/main.go" ]

# main.go
   1 package main
   2
   3 import (
   4         "fmt"
   5         "log"
   6         "net/http"
   7 )
   8
   9 func main() {
  10         http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
  11                         log.Println("received request")
  12                         fmt.Fprintf(w, "Hello Docker!!")
  13         })
  14         log.Println("start server")
  15         server := &http.Server{ Addr: ":8080"  }
  16         if err := server.ListenAndServe(); err != nil {
  17                 log.Println(err)
  18         }
  19
  20 }
# docker-compose.yml
version: "3"
services:
  echo:
    #image: myanjini/echo:latest
    build: .
    ports:
      - 9000:8080
docker-compose up
# 결과
Creating network "compose_default" with the default driver
Creating compose_echo_1 ... done
Attaching to compose_echo_1
echo_1  | 2020/09/16 02:32:38 start server
```

## 젠킨스 컨테이너 올려보기

```yaml
# docker-compose.yml
version: "3"
services:
  master:
    container_name: master
    image: jenkinsci/jenkins
    ports:
      - 8080:8080
    volumes:
      - ./jenkins_home:/var/jenkins_home
```

`docker-compose up` 컨테이너가 실행되면 호스트에서 8080 포트포워딩