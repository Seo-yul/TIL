# Docker

- centos7 환경에서 실습 진행

## install

```
yum install epel-release
yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-selinux docker-engine-selinux docker-engine
```

도커 이미 있다면 다 지우고 밑 설치

```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

`yum-config-manager --add-repo <https://download.docker.com/linux/centos/docker-ce.repo`>

`yum install -y docker-ce` 최신버전 도커 설치

`sudo useradd docker` : 사용자를 만들고

`sudo usermod -a -G docker $USER` : 도커 사용자 를 추가 sudo 없이 명령 사용가능하게

`sudo systemctl start docker.service` : 서비스 시작

`sudo systemctl enable docker.service` : 서비스 서비스 등록

`sudo systemctl reload docker.service` : 서비스 리로드

`docker version` : 도커 버전확인

## docker hub login

`docker login` : 도커 허브의 계정으로 로그인하여 허브에 이미지를 올리고 내리는데 사용한다.

## docker hub search image

`docker search <imgname>` : <imaname>으로 허브에서 이미지를 검색할 수 있다.

## docker image download

`docker pull <imgname>` : 도커 이미지 다운로드

```bash
# centos 최신버전을 도커 공식에서 다운받는다.
# docker pull docker.io/centos
[vagrant@ohmygirl ~]$ docker pull docker.io/centos
Using default tag: latest                                   # 최신버전
Trying to pull repository docker.io/library/centos ...      # 레포지토리에서 pull
latest: Pulling from docker.io/library/centos               
3c72a8ed6814: Pull complete
Digest: sha256:76d24f3ba3317fa945743bb3746fbaf3a0b752f10b10376960de01da70685fbd
Status: Downloaded newer image for docker.io/centos:latest
```

로그를 보면 `:latest` 라고 태그가 붙어있다. 이 태그는 이미지의 버전을 관리하는 이름이다.

명시하지 않으면 latest 가 기본으로 설정된다. 이미지의 형식은 `계정명/이미지명:태그`

## image list check

`docker image list[또는 ls]` : 내려받은 image 목록을 확인한다.

## container run

```
docker run -ti -d --privileged -p 8888:80 --name centos7 docker.io/centos:centos7
```

컨테이너를 생성한다. 현재 centos 최신버전은 8로 7과 달라 태그를 설정해줬다.

`-d` 옵션으로 백그라운드 실행 `-ti` pseudo-TTY 를 할당해 상호작용

`--name` 으로 이름 컨테이너 이름 부여

`--privileged` 도커는 기본적으로 unprivileged 모드이지만 시스템 콜을 사용하기 위해

`-p` 포트 포워딩 옵션이다 `호스트포트:컨테이너포트` 를 의미한다.

현재 로컬에 docker image가 존재하지 않으면 자동으로 docker hub에서 찾는다.

## container run check

현재 실행중인 컨테이너 목록을 확인한다. `docker container ls` 또는 `docker container ps`

`-a` 옵션으로 종료된 컨테이너까지 확인

## container remove

`docker container rm <container_id_or_name>` : 컨테이너의 id 또는 name 을 파라미터로 지정하여 지울 수 있다. rm의 옵션으로 `-f` 를 주어 강제 삭제도 가능

## container stop / start

`docker container stop <container_id_or_name>` : 컨테이너 정지

`docker container start <container_id_or_name>` : 컨테이너 시작

## container log

```
docker container logs -f <container_name>
```

## 명령행 필터

`docker container ls --filter "ancestor=example/echo"` : example/echo 로 만들어진 컨테이너

필터링에 사용하는 내용은 "사용자명/이미지명" 까지만 식별한다. tag식별 불가

### 컨테이너 조회 결과에서 ID 만 추출

```
docker container ls --filter "ancestor=example/echo" -q
```

### 동일 이미지로 생성된 컨테이너를 일괄 삭제하기

```
docker container stop $(docker container ls --filter "ancestor=example/echo" -q)
```

### 태깅되지 않은 이미지를 검색

```
docker image ls -f "dangling=true"
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              fefad6ab4ef6        11 minutes ago      1.23MB
```

### 이미지 태그 변경

```
docker image tag --help
Usage:  docker image tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
```

### 리눅스에서 json 을 가공해주는 jq 모듈

```
json 형식을 출력하는 output 명령 | jq
```

# 도커 이미지

## Dockerfile 이란?

### centos 의 경우

```bash
CentOS Dockerfile
<https://github.com/CentOS/sig-cloud-instance-images/blob/f2788ce41161326a18420913b0195d1c6cfa1581/docker/Dockerfile>

FROM scratch					                # 컨테이너 이미지를 처음부터 생성
ADD centos-7-x86_64-docker.tar.xz /		# 이미지에 파일을 추가 (ADD: 압축 파일을 풀어서 배치, COPY: 단순 복사)

LABEL \\
    org.label-schema.schema-version="1.0" \\
    org.label-schema.name="CentOS Base Image" \\
    org.label-schema.vendor="CentOS" \\
    org.label-schema.license="GPLv2" \\
    org.label-schema.build-date="20200809" \\
    org.opencontainers.image.title="CentOS Base Image" \\
    org.opencontainers.image.vendor="CentOS" \\
    org.opencontainers.image.licenses="GPL-2.0-only" \\
    org.opencontainers.image.created="2020-08-09 00:00:00+01:00"

CMD ["/bin/bash"]				               # 컨테이너가 기동될 때 실행할 default 프로세스를 지정
```

## Dockerfile 작성

1. 컨테이너 이미지 내부로 전달할 파일을 생성

```bash
echo "Hello, Docker." > hello-docker.txt
```

1. Dockerfile을 생성 ⇒ 이미지 생성에 사용

```bash
FROM docker.io/centos:latest        # 베이스 이미지를 지정
ADD  hello-docker.txt /tmp          # 호스트에 있는 hello-docker.txt 파일을 컨테이너 이미지의 /tmp 아래로 복사
RUN  yum install -y epel-release    # 컨테이너 이미지를 만들 때 실행
CMD  [ "/bin/bash" ]                # 컨테이너가 실행될 때 실행할 명령어
```

## Dockerfile을 이용해서 이미지를 빌드

이미지 빌드가 완료되면 Dockerfile의 명령어 줄 수 만큼 레이어가 존재

실제 컨테이너에서 사용하는 파일(디렉토리)이 이미지 레이어에 존재하면 공간만 차지하게 됨

Dockerfile을 작성할 때 &&을 이용하여 각 RUN 명령어를 하나로 묶어 실행하는것이 좋다.

```bash
docker image build -t 계정명/이미지:버전 .
~~~~~~~~~~~~~~~~~~ ~~~~~~~~~~~~~~~~~~~~~~ ~
|                  |                     Dockerfile 위치 ( . ⇒ 현재 위치)
|                  +-- 이미지 이름 ⇒ DOCKER_HUB_ID/IMAGE_NAME:TAG_NAME
+-- Dockerfile을 이용해서 이미지를 생성
```

## 컨테이너를 이용한 이미지 생성

```bash
docker container commit devops-book-1.0 myanjini/centos:1.1
~~~~~~~~~~~~~~~~~~~~~~~ ~~~~~~~~~~~~~~~ ~~~~~~~~~~~~~~~~~~~
|                        컨테이너 이름   이미지 이름
+-- 컨테이너의 현재 상태를 이미지로 기록(생성)
```

## 이미지를 도커 허브에 등록

```
docker image push <dockhub_id/image_name:tag>
docker image push riceseed/centos:1.1
```

### 이미지의 이름이 겹친다면 이전 이미지의 이름이 none 처리되어 남는다.

```bash
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
example/echo        latest              10328df67a53        20 minutes ago      750MB
<none>              <none>              ea3a770bcc3b        22 minutes ago      750MB
```

## 출력 형식 지정 (formatting) - 중요

https://docs.docker.com/engine/reference/commandline/ps/

```
docker container ls -a --format "table {{.ID}} : {{.Names}} \\t {{.Command}}"
```

centos 에서 deamon 실행 권한 문제 시 실행법

```bash
sudo docker run -d —-privileged —-name centos /sbin/init
```

기본적으로 보안의 이유로 도커는 unprivileged 모드로 작동한다!

# 컨테이너의 데이터를 영속적(persistent)인 데이터로 활용하기

## 방법1. 호스트 볼륨을 공유한다.

-v 옵션을 이용해서 호스트 볼륨을 공유한다.

호스트의 디렉토리를 컨테이너의 디렉토리에 마운트

원래 존재하는 호스트의 디렉토리를 볼륨으로 공유하면 컨테이너의 디렉토리 자체가 덮어씌워져진다.

### 실습하기

1. 모든 컨테이너, 이미지, 볼륨을 삭제

```bash
vagrant@xenial64:~/blog$ docker container rm -f $(docker container ls -aq)
vagrant@xenial64:~/blog$ docker image rm -f $(docker image ls -aq)
vagrant@xenial64:~/blog$ docker volume rm -f $(docker volume ls -q)
```

1. MySQL 이미지를 이용한 데이터베이스 컨테이너를 생성

```bash
vagrant@xenial64:~/blog$ docker run -d --name wordpressdb_hostvolume -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=wordpress -v **/home/vagrant/wordpress_db**:/var/lib/mysql mysql:5.7
6b7848cca9068f3af8b1cf56acbb9d5fb0160f0500e35cd4498ba13fbe14757a
```

`/home/wordpress_db` : 도커가 자동으로 생성

`/var/lib/mysql` : mysql 데이터베이스의 데이터를 저장하는 기본 디렉터리

1. 워드프레스 이미지를 이용해 웹 서버 컨테이너를 생성

```bash
vagrant@xenial64:~/blog$ docker run -d -e WORDPRESS_DB_PASSWORD=password --name wordpress_hostvolume --link wordpressdb_hostvolume:mysql -p 80 wordpress
```

1. 호스트 볼륨 공유를 확인

```bash
vagrant@xenial64:~/blog$ ls /home/vagrant/wordpress_db
auto.cnf ca.pem client-key.pem ibdata1 ib_logfile1 private_key.pem server-cert.pem
ca-key.pem client-cert.pem ib_buffer_pool ib_logfile0 mysql public_key.pem server-key.pem
```

1. 컨테이너 내부의 디렉터리 확인

```bash
vagrant@xenial64:/home$ docker container exec wordpressdb_hostvolume ls /var/lib/mysql
auto.cnf ca.pem client-key.pem ibdata1 ib_logfile1 private_key.pem server-cert.pem
ca-key.pem client-cert.pem ib_buffer_pool ib_logfile0 mysql public_key.pem server-key.pem
```

호스트에 공유된 볼륨 디렉토리와 같은것을 확인할 수 있다.

1. wordpressdb_hostvolume 컨테이너를 삭제한 후 호스트 볼륨을 확인

```bash
vagrant@xenial64:/home$ docker container rm -f wordpressdb_hostvolume
wordpressdb_hostvolume

vagrant@xenial64:/home$ docker container ls -a
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
54f4aa9f0aa1 wordpress "docker-entrypoint.s…" 3 minutes ago Up 3 minutes 0.0.0.0:32777->80/tcp wordpress_hostvolume

vagrant@xenial64:/home$ ls /home/vagrant/wordpress_db/
auto.cnf ca.pem client-key.pem ibdata1 ib_logfile1 mysql private_key.pem server-cert.pem sys
ca-key.pem client-cert.pem ib_buffer_pool ib_logfile0 ibtmp1 performance_schema public_key.pem server-key.pem wordpress
```

컨테이너는 삭제되었지만 공유되고 있던 디렉토리는 그대로 남아 파일 보존됨을 확인

데이터(파일)에 영속성을 부여

## 방법2. 볼륨 컨테이너

-v 옵션으로 볼륨을 사용하는 컨테이너를 다른 컨테이너와 공유하는 것

컨테이너를 생성할 때 `--volumes-from` 옵션을 설정하면 -v 또는 --volume 옵션을 적용한 컨테이너의 볼륨 디렉터리 공유가 가능하다.

## 방법3. 도커 볼륨

도커 자체가 제공하는 볼륨 기능을 활용

1. 볼륨 생성

```bash
vagrant@xenial64:~$ docker volume create --name myvolume
myvolume

vagrant@xenial64:~$ docker volume ls
DRIVER              VOLUME NAME
local               30cc85d3457a1dc55d7f6059f14c27d662d1848a2bd50d4eb95d362f9f43b3a5
local               375b5e63b4ee4dbf08ce79d11cbae985b58a0a2131acb0576aa19cb3bc94bf9e
local               eec00043797c8d49331faab9e05f427b787fa8519f1113ac5cfe4ceeec1fac37
local               myvolume
```

1. 생성한 볼륨을 이용해서 컨테이너를 생성

```bash
-v 볼륨이름:컨테이너내부디렉터리

vagrant@xenial64:~$ docker run -it --name myvolume1 -v myvolume:/root/ ubuntu:14.04
root@0bf6ac067a56:/# cd root
root@0bf6ac067a56:~# echo hello, volume >> /root/volume
root@0bf6ac067a56:~# exit
exit
```

1. 동일 볼륨을 사용하는 컨테를 생성해서 공유 여부를 확인

```bash
vagrant@xenial64:~$ docker run -it --name myvolume2 -v myvolume:/temp/ ubuntu:14.04
root@bc6bc0c35bfc:/# cat /temp/volume
hello, volume
root@bc6bc0c35bfc:/#
```

1. docker inspect 명령으로 볼륨의 저장 위치를 확인

```bash
vagrant@xenial64:~$ docker inspect --type volume myvolume
[
    {
        "CreatedAt": "2020-09-16T00:42:36Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/myvolume/_data",
        "Name": "myvolume",
        "Options": {},
        "Scope": "local"
    }
]
```