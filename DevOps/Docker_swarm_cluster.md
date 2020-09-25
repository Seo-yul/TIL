# docker swarm cluster

아래 실습은 3개의 머신에서 합니다.

```yaml
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
# config.vm.box = "centos/7"
  config.vm.box = "generic/centos7"
  config.vm.hostname = "swarm-manager"
  config.vm.network "private_network", ip: "192.168.111.100"
  config.vm.synced_folder ".", "/home/vagrant/sync", disabled: true
end
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
# config.vm.box = "centos/7"
  config.vm.box = "generic/centos7"
  config.vm.hostname = "swarm-worker"
  config.vm.network "private_network", ip: "192.168.111.101"
  config.vm.synced_folder ".", "/home/vagrant/sync", disabled: true
end
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
# config.vm.box = "centos/7"
  config.vm.box = "generic/centos7"
  config.vm.hostname = "swarm-worker2"
  config.vm.network "private_network", ip: "192.168.111.102"
  config.vm.synced_folder ".", "/home/vagrant/sync", disabled: true
end
```

Host Name IP ============= =============== swarm-manager 192.168.111.100 swarm-worker 192.168.111.101 swarm-worker2 192.168.111.102

------

## **도커 스웜 모드의 구조**

port 2377 사용

- 매니저(manager) 노드와 워커(worker) 노드로 구성
- 워커 노드 ⇒ 실제 컨테이너가 생성되고 관리되는 도커 서버
- 매니저 노드 ⇒ 워커 노드를 관리하기 위한 도커 서버
- 매니저 노드는 워커 노드의 역할을 포함
- 클러스터를 구성하기 위해서는 최소 1개 이상의 매니저 노드가 존재해야 함

## **매니저 역할의 서버에서 스웜 클러스터를 시작**

```bash
# 매니저에서
docker swarm init --advertise-addr 192.168.111.100
#                                  ~~~~~~~~~~~~~~~~~ **⇒ 매니저 노드의 주소** 

Swarm initialized: current node (ym6n02vuulxa9izewh25vdv57) is now a manager.
To add a worker to this swarm, run the following command:

docker swarm join \\
--token SWMTKN-1-0y1pdbtlgh4zuag7nq8zjph287wih5gh8pkizfcuxsrkvj4z9o-bb0n5qy7r1umig6lwf6cq91ys \\
192.168.111.100:2377

# 새로운 워커 노드를 클러스터에 추가할 때 사용하는 비밀키(토큰)
```

## **워커 노드를 추가**

매니저의 방화벽 주의

```bash
firewall-cmd --permanent --zone=public --add-port=2377/tcp
# 워커에서
docker swarm join \\
--token SWMTKN-1-51gsqhe3wqo7nj7tffuexh2jphwfjffy0a5anzgbx6ljn0m4as-24zl1npz454ep1olarxsddu0o \\
192.168.111.100:2377

This node joined a swarm as a worker.
```

## **스웜 클러스터에 정상적으로 추가되었는지 확인**

```
docker node ls
ID HOSTNAME STATUS AVAILABILITY MANAGER STATUS
u0aezhea44cx9z5faipmntbpl swarm-worker2 Ready Active
wdnqgalwfd9828npcr516uw2s swarm-worker1 Ready Active
ym6n02vuulxa9izewh25vdv57 * swarm-manager Ready Active Leader
```

## **토큰 확인 및 변경 방법**

```
docker swarm join-token manager
To add a manager to this swarm, run the following command:

docker swarm join \\
--token SWMTKN-1-51gsqhe3wqo7nj7tffuexh2jphwfjffy0a5anzgbx6ljn0m4as-**3m4lmy22pfapenpxonqvcpobj** \\
192.168.111.100:2377
docker swarm join-token worker
To add a worker to this swarm, run the following command:

docker swarm join \\
--token SWMTKN-1-51gsqhe3wqo7nj7tffuexh2jphwfjffy0a5anzgbx6ljn0m4as-24zl1npz454ep1olarxsddu0o \\
192.168.111.100:2377
docker swarm join-token --rotate manager
Successfully rotated manager join token.
To add a manager to this swarm, run the following command:

docker swarm join \\
--token SWMTKN-1-51gsqhe3wqo7nj7tffuexh2jphwfjffy0a5anzgbx6ljn0m4as-**2x9uqu2xqgrobck2ho7z10rgh** \\
192.168.111.100:2377
```

## 워커 노드 삭제

```
docker swarm leave
Node left the swarm.

# 매니저에서 확인
docker node ls

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
wn5j8nymx704prelscdq8fw54 *   swarm-manager       Ready               Active              Leader              19.03.6
i2boml3wjv87wjv35mpi0okgt     swarm-worker1       Ready               Active                                  19.03.6
m110afvl76ppbomnlmh6zzwke     swarm-worker2       Down                Active                                  19.03.6

# 매니저에서 워커 삭제
docker node rm swarm-worker2

swarm-worker2

# 매니저에서 다시 확인
docker node ls

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
wn5j8nymx704prelscdq8fw54 *   swarm-manager       Ready               Active              Leader              19.03.6
i2boml3wjv87wjv35mpi0okgt     swarm-worker1       Ready               Active                                  19.03.6
```

## 워커 노드를 매니저 노드로 변경

`promote` 명령으로 매니저 노드에서 워크 노드를 매니저 노드로 승격시킨다.

```bash
# 매니저에서
docker node promote swarm-worker1
Node swarm-worker1 promoted to a manager in the swarm.

docker node ls
```

## 매니저 노드를 워커 노드로 변경

`demote` 명령으로 매니저 노드를 워커 노드로 변경한다.

```bash
# 매니저에서
docker node demote swarm-worker1
Manager swarm-worker1 demoted in the swarm.

docker node ls
```

# 도커 스웜 모드

도커 명령어의 제어 단위 : 컨테이너

스웜 모드의 제어 단위 : 서비스

서비스 : 같은 이미지에서 생성된 컨테이너의 집합으로 서비스를 제어하면 해당 서비스 내의 컨테이너에 같은 명령이 실행된다. 서비스 내에 컨테이너는 한 개 이상 존재할 수 있으며, 컨테이너들은 각 워커 노드와 매니저 노드에 할당된다. 각 노드에 할당된 컨테이너들을 task 라고 한다.

## 서비스 생성

```
docker service create <image:tage > <명령>
docker service create ubuntu:14.04 bin/sh -c "while true; do echo hello world; sleep 1; done"
mak6jjcxo2ayknbczbs0ru8pg
```

## 서비스 조회

```
docker service ls
ID            NAME           MODE        REPLICAS  IMAGE
mak6jjcxo2ay  clever_mclean  replicated  1/1       ubuntu:14.04
```

## 서비스 상세 조회

```
docker service ps <name>
ID            NAME             IMAGE         NODE           DESIRED STATE  CURRENT STATE          ERROR  PORTS
jirp2iwkktd0  clever_mclean.1  ubuntu:14.04  swarm-worker1  Running        Running 7 minutes ago
```

## 서비스 삭제

docker rm 과 달리 서비스 상태에 관계없이 서비스 중 컨테이너를 바로 삭제 가능

```
docker service rm <name>
```

## 웹 서버 서비스 생성해보기

`--replicas` 옵션을 이용해서 항상 유지하는 서비스 개수를 복제하여 지정한다.

```bash
# 매니저에서
docker service create --name myweb --replicas 2 -p 80:80 nginx
```

### 레플리카셋 추가

컨테이너 수를 늘린 뒤 서비스 내의 컨테이너 목록을 확인해본다.

```bash
# 매니저에서
docker service scale myweb=4
myweb scaled to 4

docker service ps myweb
ID            NAME     IMAGE         NODE           DESIRED STATE  CURRENT STATE                    ERROR  PORTS
hvx4iwhma1rg  myweb.1  nginx:latest  swarm-manager  Running        Running 8 minutes ago
xx00pyjbdfi7  myweb.2  nginx:latest  swarm-worker2  Running        Running 8 minutes ago
27rfxfyj82xk  myweb.3  nginx:latest  swarm-worker1  Running        Running less than a second ago
2le2o9cqyh00  myweb.4  nginx:latest  swarm-worker1  Running        Starting less than a second ago
```

## 서비스 모드

### 복제(replicated) 모드

레플리카 셋의 수를 정의해 그 만큼의 같은 컨테이너를 생성

실제 서비스 제공에 일반적으로 사용하는 모드

### 글로벌(global)모드

스웜 클러스터 내에서 사용할 수 있는 모든 노드에 컨테이너를 반드시 하나씩 생성

레플리카 셋의 수를 별도로 지정하지 않는다.

스웜 클러스터를 모니터링하기 위한 에이전트 컨테이너 등을 생성해야 할 때 유용

`docker service create --mode global` 로 생성

```bash
# 글로벌 모드 실습 예제
docker service create --name global_web --mode global nginx
docker service ls
docker service ps global_web
```

## 서비스 장애 복구

복제 모드로 설정된 서비스의 컨테이너가 정지하거나 특정 노드가 다운되면 스웜 매니저는 새로운 컨테이너를 생성해서 자동으로 복구한다.

### 특정 노드에서 myweb 서비스에 속한 컨테이너를 삭제하면 자동으로 다시 생성되는 것을 확인

swarm-manager 노드에서 실행 중인 컨테이너 목록 확인

```bash
docker ps

CONTAINER ID        IMAGE                                                                           COMMAND                  CREATED             STATUS              PORTS               NAMES
1a873783c5a9        nginx@sha256:c628b67d21744fce822d22fdcc0389f6bd763daac23a6b77147d0712ea7102d0   "/docker-entrypoin..."   56 minutes ago      Up 56 minutes       80/tcp              myweb.4.50lda66kfxdxhhbp58mbeq8co
96d8aca3d877        nginx@sha256:c628b67d21744fce822d22fdcc0389f6bd763daac23a6b77147d0712ea7102d0   "/docker-entrypoin..."   About an hour ago   Up About an hour    80/tcp              myweb.1.j3hzz2g1ldhm5f6e79h9gy08f
# 컨테이너의 NAME 은 서비스 이름, 레플리카 번호, 자동으로 생성된 고유 id 로 구성된다.
```

매니저 노드에서 실행 중인 컨테이너 강제 삭제

```bash
[vagrant@swarm-manager ~]$ docker rm -f myweb.1.j3hzz2g1ldhm5f6e79h9gy08f
myweb.1.j3hzz2g1ldhm5f6e79h9gy08f
```

myservice 서비스의 task 확인 _> 새로운 task 생성 확인

```bash
[vagrant@swarm-manager ~]$ docker service ps myweb
ID            NAME         IMAGE         NODE           DESIRED STATE  CURRENT STATE               ERROR                             PORTS
3ge74717tznl  myweb.1      nginx:latest  swarm-worker1  Running        Running about a minute ago
j3hzz2g1ldhm   \\_ myweb.1  nginx:latest  swarm-manager  Shutdown       Failed about a minute ago   "task: non-zero exit (137)"
hvx4iwhma1rg   \\_ myweb.1  nginx:latest  swarm-manager  Shutdown       Failed about an hour ago    "task: non-zero exit (0): No s…"
mcvytgn63xsg  myweb.2      nginx:latest  swarm-worker1  Running        Running about an hour ago
xx00pyjbdfi7   \\_ myweb.2  nginx:latest  swarm-worker2  Shutdown       Failed about an hour ago    "task: non-zero exit (0): No s…"
hodibbd262mb  myweb.3      nginx:latest  swarm-worker2  Running        Running about an hour ago
50lda66kfxdx  myweb.4      nginx:latest  swarm-manager  Running        Running 58 minutes ago
```

## 특정 노드가 다운되면 해당 노드의 컨테이너가 종료되고 다른 노드에 컨테이너가 생성되는 것을 확인

실행중인 swarm-worker1 노드의 도커 데몬 프로세스를 종료

```bash
# swarm-worker1 에서
service docker stop
```

매니저 노드에서 노드 상태 확인 _> swarm-worker1 노드의 다운 확인

```bash
[vagrant@swarm-manager ~]$ docker node ls
ID                           HOSTNAME       STATUS  AVAILABILITY  MANAGER STATUS
kh73efg2adhk6ia78djx0tc54    swarm-worker1  Down    Active
oru8ev52mtl7d9zh2ec34dryu    swarm-worker2  Ready   Active
zt5pxwu0iep9e6qx6r8y0mwxr *  swarm-manager  Ready   Active        Leader
```

매니저 노드에서 myservice 서비스 task 상태 확인 _> 실행 중인 다른 노드에서 task 생성 확인

```bash
[vagrant@swarm-manager ~]$ docker service ps myweb
ID            NAME         IMAGE         NODE           DESIRED STATE  CURRENT STATE               ERROR                             PORTS
0u6a5048gn82  myweb.1      nginx:latest  swarm-manager  Running        Running about a minute ago
3ge74717tznl   \\_ myweb.1  nginx:latest  swarm-worker1  Shutdown       Running about a minute ago
j3hzz2g1ldhm   \\_ myweb.1  nginx:latest  swarm-manager  Shutdown       Failed 5 minutes ago        "task: non-zero exit (137)"
hvx4iwhma1rg   \\_ myweb.1  nginx:latest  swarm-manager  Shutdown       Failed about an hour ago    "task: non-zero exit (0): No s…"
ibdsd6yq9yut  myweb.2      nginx:latest  swarm-manager  Running        Running about a minute ago   # 매니저에 태스크 생성됨
mcvytgn63xsg   \\_ myweb.2  nginx:latest  swarm-worker1  Shutdown       Running about a minute ago
xx00pyjbdfi7   \\_ myweb.2  nginx:latest  swarm-worker2  Shutdown       Failed about an hour ago    "task: non-zero exit (0): No s…"
hodibbd262mb  myweb.3      nginx:latest  swarm-worker2  Running        Running about an hour ago
50lda66kfxdx  myweb.4      nginx:latest  swarm-manager  Running        Running about an hour ago
```

## 다운되었던 노드를 재시작해도 재균형(rebalance) 작업은 자동으로 일어나지 않는다.

다운되었던 swarm-worker1 노드 재시작

```bash
# swarm-worker1 에서
service docker start
```

매니저 노드에서 노드 상태 확인 _> swarm-work1 노드 active 확인

```bash
[vagrant@swarm-manager ~]$ docker node ls
ID                           HOSTNAME       STATUS  AVAILABILITY  MANAGER STATUS
kh73efg2adhk6ia78djx0tc54    swarm-worker1  Ready   Active
oru8ev52mtl7d9zh2ec34dryu    swarm-worker2  Ready   Active
zt5pxwu0iep9e6qx6r8y0mwxr *  swarm-manager  Ready   Active        Leader
```

매니저 노드에서 myservice 서비스의 task가 여전함을 확인

```bash
[vagrant@swarm-manager ~]$ docker service ps myweb
ID            NAME         IMAGE         NODE           DESIRED STATE  CURRENT STATE              ERROR                             PORTS
0u6a5048gn82  myweb.1      nginx:latest  swarm-manager  Running        Running 4 minutes ago
3ge74717tznl   \\_ myweb.1  nginx:latest  swarm-worker1  Shutdown       Failed about a minute ago  "No such container: myweb.1.3g…"
j3hzz2g1ldhm   \\_ myweb.1  nginx:latest  swarm-manager  Shutdown       Failed 8 minutes ago       "task: non-zero exit (137)"
hvx4iwhma1rg   \\_ myweb.1  nginx:latest  swarm-manager  Shutdown       Failed about an hour ago   "task: non-zero exit (0): No s…"
ibdsd6yq9yut  myweb.2      nginx:latest  swarm-manager  Running        Running 4 minutes ago      # 여전히 매니저가 담당중
mcvytgn63xsg   \\_ myweb.2  nginx:latest  swarm-worker1  Shutdown       Failed about a minute ago  "No such container: myweb.2.mc…"
xx00pyjbdfi7   \\_ myweb.2  nginx:latest  swarm-worker2  Shutdown       Failed about an hour ago   "task: non-zero exit (0): No s…"
hodibbd262mb  myweb.3      nginx:latest  swarm-worker2  Running        Running about an hour ago
50lda66kfxdx  myweb.4      nginx:latest  swarm-manager  Running        Running about an hour ago
```

## docker 컨테이너 할당 균형 맞추기

`docker service scale` 명령으로 컨테이너 수를 줄이고 다시 늘리는 방식으로 할당 균형을 맞춤

```bash
# 매니저에서
docker service scale myweb=1
myweb scaled to 1

docker service scale myweb=4
myweb scaled to 4

docker service ps myweb
ID            NAME         IMAGE         NODE           DESIRED STATE  CURRENT STATE             ERROR                             PORTS
0u6a5048gn82  myweb.1      nginx:latest  swarm-manager  Running        Running 6 minutes ago
3ge74717tznl   \\_ myweb.1  nginx:latest  swarm-worker1  Shutdown       Failed 3 minutes ago      "No such container: myweb.1.3g…"
j3hzz2g1ldhm   \\_ myweb.1  nginx:latest  swarm-manager  Shutdown       Failed 11 minutes ago     "task: non-zero exit (137)"
hvx4iwhma1rg   \\_ myweb.1  nginx:latest  swarm-manager  Shutdown       Failed about an hour ago  "task: non-zero exit (0): No s…"
g5qbuloicoyd  myweb.2      nginx:latest  swarm-worker2  Running        Running 15 seconds ago
dhlpohkd3kpn  myweb.3      nginx:latest  swarm-worker1  Running        Running 15 seconds ago
z81hxl03czqa  myweb.4      nginx:latest  swarm-worker1  Running        Running 15 seconds ago
```

# 서비스 롤링 업데이트

스웜모드에서 자체적으로 지원하는 기능

### 서비스 생성

```bash
[vagrant@swarm-manager ~]$ docker service create --name myweb2 --replicas 3 nginx:1.10
81atz3gy57ya8v93a2uhy6jol
[vagrant@swarm-manager ~]$ docker service ps myweb2
ID            NAME      IMAGE       NODE           DESIRED STATE  CURRENT STATE           ERROR  PORTS
e0j5szd32bzp  myweb2.1  nginx:1.10  swarm-manager  Running        Running 16 seconds ago
jh2woxo27cl2  myweb2.2  nginx:1.10  swarm-worker2  Running        Running 16 seconds ago
yxftzi1xawpb  myweb2.3  nginx:1.10  swarm-worker1  Running        Running 16 seconds ago
```

### 서비스 설정 변경

`docker service update` 명령을 이용

```bash
# nginx 의 버전을 업데이트
[vagrant@swarm-manager ~]$ docker service update --image nginx:1.11 myweb2
myweb2
[vagrant@swarm-manager ~]$ docker service ps myweb2
ID            NAME          IMAGE       NODE           DESIRED STATE  CURRENT STATE            ERROR  PORTS
qw7fwymzgqek  myweb2.1      nginx:1.11  swarm-manager  Running        Running 11 seconds ago
e0j5szd32bzp   \\_ myweb2.1  nginx:1.10  swarm-manager  Shutdown       Shutdown 16 seconds ago
hwird9p11o4g  myweb2.2      nginx:1.11  swarm-worker2  Running        Running 4 seconds ago
jh2woxo27cl2   \\_ myweb2.2  nginx:1.10  swarm-worker2  Shutdown       Shutdown 10 seconds ago
ued1v8qn54hu  myweb2.3      nginx:1.11  swarm-worker1  Running        Preparing 4 seconds ago
yxftzi1xawpb   \\_ myweb2.3  nginx:1.10  swarm-worker1  Shutdown       Shutdown 3 seconds ago
```

### 서비스를 생성할 롤링 업데이트 설정

롤링 업데이트 주기, 업데이트를 동시에 진행할 컨테이너의 개수, 업데이트 실패 시 대처 설정

```bash
                                                                            # 10초 단위 업데이트, 업데이트 작업을 한 번에 2개의 컨테이너에서 수행
[vagrant@swarm-manager ~]$ docker service create --replicas 4 --name myweb3 --update-delay 10s --update-parallelism 2 nginx:1.10
k0rk0fprnjtrd6v50wtxnv7fs
```

롤링 업데이트 설정 확인 `docker service inspect --pretty <name>` , `--type` 옵션으로 상세보기 가능

```bash
[vagrant@swarm-manager ~]$ docker service inspect --pretty myweb3

ID:             k0rk0fprnjtrd6v50wtxnv7fs
Name:           myweb3
Service Mode:   Replicated
 Replicas:      4
Placement:
UpdateConfig:
 Parallelism:   2
 Delay:         10s
 On failure:    pause   # 업데이트 중 오류 시 중지
 Max failure ratio: 0
ContainerSpec:
 Image:         nginx:1.10@sha256:6202beb06ea61f44179e02ca965e8e13b961d12640101fca213efbfd145d7575
Resources:
Endpoint Mode:  vip
```

업데이트 중 오류가 있어도 진행하도록 설정 `--update-failure-action continue` 옵션의 추가

```bash
[vagrant@swarm-manager ~]$ docker service create --name myweb4 --replicas 4 --update-failure-action continue nginx:1.10
m31abx0c572f34x9kuc2b8uct
[vagrant@swarm-manager ~]$ docker service inspect --pretty myweb4

ID:             m31abx0c572f34x9kuc2b8uct
Name:           myweb4
Service Mode:   Replicated
 Replicas:      4
Placement:
UpdateConfig:
 Parallelism:   1
 On failure:    continue
 Max failure ratio: 0
ContainerSpec:
 Image:         nginx:1.10@sha256:6202beb06ea61f44179e02ca965e8e13b961d12640101fca213efbfd145d7575
Resources:
Endpoint Mode:  vip
```

## 서비스를 롤링 업데이트 이전으로 돌리는 롤백

참고 : https://docs.docker.com/engine/reference/commandline/service_update/

```
docker service update --rollback <service_name>
[vagrant@swarm-manager ~]$ docker service ps myweb2
ID            NAME          IMAGE       NODE           DESIRED STATE  CURRENT STATE           ERROR  PORTS
qw7fwymzgqek  myweb2.1      nginx:1.11  swarm-manager  Running        Running 9 minutes ago
e0j5szd32bzp   \\_ myweb2.1  nginx:1.10  swarm-manager  Shutdown       Shutdown 9 minutes ago
hwird9p11o4g  myweb2.2      nginx:1.11  swarm-worker2  Running        Running 9 minutes ago
jh2woxo27cl2   \\_ myweb2.2  nginx:1.10  swarm-worker2  Shutdown       Shutdown 9 minutes ago
ued1v8qn54hu  myweb2.3      nginx:1.11  swarm-worker1  Running        Running 9 minutes ago
yxftzi1xawpb   \\_ myweb2.3  nginx:1.10  swarm-worker1  Shutdown       Shutdown 9 minutes ago

[vagrant@swarm-manager ~]$ docker service update --rollback myweb2
myweb2

[vagrant@swarm-manager ~]$ docker service ps myweb2
ID            NAME          IMAGE       NODE           DESIRED STATE  CURRENT STATE                   ERROR  PORTS
lq38k7l3dr2z  myweb2.1      nginx:1.10  swarm-manager  Running        Running less than a second ago
qw7fwymzgqek   \\_ myweb2.1  nginx:1.11  swarm-manager  Shutdown       Shutdown 1 second ago
e0j5szd32bzp   \\_ myweb2.1  nginx:1.10  swarm-manager  Shutdown       Shutdown 12 minutes ago
zz7nq31392g9  myweb2.2      nginx:1.10  swarm-worker2  Running        Running 2 seconds ago
hwird9p11o4g   \\_ myweb2.2  nginx:1.11  swarm-worker2  Shutdown       Shutdown 2 seconds ago
jh2woxo27cl2   \\_ myweb2.2  nginx:1.10  swarm-worker2  Shutdown       Shutdown 12 minutes ago
wgnaod03otso  myweb2.3      nginx:1.10  swarm-worker1  Ready          Ready less than a second ago
ued1v8qn54hu   \\_ myweb2.3  nginx:1.11  swarm-worker1  Shutdown       Running less than a second ago
yxftzi1xawpb   \\_ myweb2.3  nginx:1.10  swarm-worker1  Shutdown       Shutdown 12 minutes ago
```

## Visualizer 를 이용한 swarm cluster 컨테이너 배치 시각화

https://hub.docker.com/r/dockersamples/visualizer

```bash
[vagrant@swarm-manager ~]$ docker service create --name=viz --publish=8080:8080/tcp --constraint=node.role==manage --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock dockersamples/visualizer
m5nxmqutywjdv4bi4ogkdakt8
```

# 서비스 컨테이너에 config 전달

### 방법1. -v 옵션을 이용 호스트에 위치한 설정 파일이나 값을 볼륨으로 공유

```bash
docker run -d --name yml_registry -p 5002:5000 --restart=always \\
-v $(pwd)/config.yml:/etc/docker/registry/config.yml \\
registry:2.6
```

### 방법2. -e 옵션을 통해 환경 변수로 전달

```bash
docker run -d --name wordpressdb_hostvolume \\
-e MYSQL_ROOT_PASSWORD=password \\
-e MYSQL_DATABASE=wordpress \\
-v /home/wordpress_db:/var/lib/mysql \\
mysql:5.7
```

## Swarm mode 는 secret 과 config 기능을 제공한다.

스웜 모드와 같은 서버 클러스터에서 파일 공유를 위해 설정 파일을 호스트마타 두는 것은 비효율적

비밀번호와 같은 민감 정보를 환경 변수로 설정하는 것은 보안상 좋지 못함

스웜 모드에서 사용할 수 있는 secret, config 기능을 제공

- secret : 비밀번호, SSH 키, 인증서 키 와 같은 보안에 민감한 데이터 전송 용도
- config : nginx 나 레지스트리 설정 파일과 같이 암호화할 필요가 없는 설정값에 사용

## secret 사용하기

secret 생성 : 생성된 secret 은 조회에도 실제 값을 확인할 수 없다. secret 값은 매니저 노드 간에 암호화 된 상태로 저장되어있다. 컨테이너 배포 후에도 메모리에 저장되며 서비스 컨테이너가 삭제될 때 함께 삭제

```bash
# 매니저에서
echo 1q2w3e4r | docker secret create my_mysql_password -
```

`docker secret ls` : secret 조회

`docker secret inspect <secret_name>` : 상세 조회

생성한 secret 을 이용해 MySQL 컨테이너 생성해보기 예제

`--secret` 옵션을 이용 컨테이너로 공유된 값은 기본적으로 컨테이너 내부의 /run/secret/ 디렉터리 마운트

`--secret source=my_mysql_password,target=/home/mysql_root_password` 처럼 디렉터리 지정 가능

```bash
# 매니저에서
docker service create --name mysql --replicas 1 \\
--secret source=my_sql_password,tartget=mysql_root_password \\
--secret source=my_sql_password,tartget=mysql_passwd \\
-e MYSQL_ROOT_PASSWORD_FILE="/run/secrets/mysql_root_password" \\
-e MYSQL_PASSWORD_FILE="/run/secrets/mysql_password" \\
-e MYSQL_DATABASE="wordpress" \\
mysql:5.7
```

결과

```bash
[root@swarm-manager tmp]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                 NAMES
6fd474eb7479        nginx:latest        "/docker-entrypoint.…"   35 minutes ago      Up 35 minutes       80/tcp                myweb.2.kzh2ocvb81dhy7u75cs5nh6vk
[root@swarm-manager tmp]# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
xh56mm3s2qwf        mysql               replicated          1/1                 mysql:5.7
rkdbvv6hbq84        myweb               replicated          4/4                 nginx:latest        *:80->80/tcp
[root@swarm-manager tmp]# docker service ps mysql
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR                              PORTS
nni283oxfzoo        mysql.1             mysql:5.7           swarm-manager       Running             Running 2 hours ago
z4g8qmv8z7rb         \\_ mysql.1         mysql:5.7           swarm-manager       Shutdown            Failed 2 hours ago      "No such container: mysql.1.z4…"
u13idq834cmn         \\_ mysql.1         mysql:5.7           swarm-worker1       Shutdown            Failed 24 minutes ago   "starting container failed: Re…"
r2du1u2zjslp         \\_ mysql.1         mysql:5.7           swarm-manager       Shutdown            Failed 2 hours ago      "starting container failed: Re…"
z3kylwczvgxx         \\_ mysql.1         mysql:5.7           swarm-worker2       Shutdown            Failed 24 minutes ago   "starting container failed: Re…"
[root@swarm-manager tmp]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                 NAMES
6fd474eb7479        nginx:latest        "/docker-entrypoint.…"   37 minutes ago      Up 37 minutes       80/tcp                myweb.2.kzh2ocvb81dhy7u75cs5nh6vk
ea9cac0d3b8e        nginx:latest        "/docker-entrypoint.…"   37 minutes ago      Up 37 minutes       80/tcp                myweb.3.rq8bk4qmfqi8o2bn2ki6uohjz
9de652dc2e01        nginx:latest        "/docker-entrypoint.…"   38 minutes ago      Up 37 minutes       80/tcp                myweb.4.tescfb62z9tpppczs5j9f5fcz
68781f8fbb55        mysql:5.7           "docker-entrypoint.s…"   2 hours ago         Up 2 hours          3306/tcp, 33060/tcp   mysql.1.nni283oxfzoo6nbq2buso3k5a
f9149ffd1053        nginx:latest        "/docker-entrypoint.…"   2 hours ago         Up 2 hours          80/tcp                myweb.1.s18tz2aybxvi2ojhz629ueepv
[root@swarm-manager tmp]# sudo docker exec mysql.1.nni283oxfzoo6nbq2buso3k5a ls /run/secrets
mysql_password
mysql_root_password
```

## config 사용하기

사설 레지스트리 설정 파일 생성

```bash
# vi config.yml

version: 0.1
log:
 level: info
storage:
 filesystem:
   rootdirectory: /registry_data
 delete:
   enabled: true
http:
  addr: 0.0.0.0:5000
```

설정 파일로 config 생성

```bash
docker config create registry-config config.yml
```

config 생성 확인

```bash
[root@swarm-manager ~]# docker config ls
ID                          NAME                CREATED             UPDATED
vgysy0vx469jlq64smndxo69a   registry-config     33 seconds ago      33 seconds ago
```

config는 입력된 값을 BASE64로 인코딩해서 저장

```bash
[root@swarm-manager ~]# docker config inspect registry-config
[
    {
        "ID": "vgysy0vx469jlq64smndxo69a",
        "Version": {
            "Index": 8082
        },
        "CreatedAt": "2020-09-17T08:33:42.752632893Z",
        "UpdatedAt": "2020-09-17T08:33:42.752632893Z",
        "Spec": {
            "Name": "registry-config",
            "Labels": {},
            "Data": "dmVyc2lvbjogMC4xCmxvZzoKIGxldmVsOiBpbmZvCnN0b3JhZ2U6CiBmaWxlc3lzdGVtOgogICByb290ZGlyZWN0b3J5OiAvcmVnaXN0cnlfZGF0YQogZGVsZXRlOgogICBlbmFibGVkOiB0cnVlCmh0dHA6CiAgYWRkcjogMC4wLjAuMDo1MDAwCg=="
        }
    }
]
```

BASE64 디코딩

```bash
echo dmVyc2lvbjogMC4xCmxvZzoKIGxldmVsOiBpbmZvCnN0b3JhZ2U6CiBmaWxlc3lzdGVtOgogICByb290ZGlyZWN\\
0b3J5OiAvcmVnaXN0cnlfZGF0YQogZGVsZXRlOgogICBlbmFibGVkOiB0cnVl\\
Cmh0dHA6CiAgYWRkcjogMC4wLjAuMDo1MDAwCg== | base64 -d

version: 0.1
log:
 level: info
storage:
 filesystem:
   rootdirectory: /registry_data
 delete:
   enabled: true
http:
  addr: 0.0.0.0:5000
```

config로 사설 레지스트리 생성

```bash
docker service create --name yml_registry -p 5000:5000 --config \\
source=registry-config,target=/etc/docker/registry/config.yml registry:2.6
```

# 도커 스웜 네트워크

## 매니저 노드에서 네트워크 목록 확인

```bash
[root@swarm-manager ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
44cb65b8699a        bridge              bridge              local
097e998b1988        docker_gwbridge     bridge              local
9c83a0b1501f        host                host                local
2jdgw8i2ktht        ingress             overlay             swarm
ff2e7e9aab2d        none                null                local
```

## Ingress 네트워크

스웜 클러스터를 생성하면 자동으로 등록되는 네트워크

스웜 모드를 사용할 때만 유효

스웜 클러스터에 등록된 모든 노드에 ingress 네트워크가 생성

```bash
[root@swarm-manager ~]# docker network ls | grep ingress
2jdgw8i2ktht        ingress             overlay             swarm
```

ingress 네트워크는 어떤 스웜 노드에 접근하더라도 서비스 내의 컨테이너에 접근할 수 있게 설정하는 라우팅 메시를 구성하고, 서비스 내의 컨테이너에 대한 접근을 라운드 로빈 방식으로 분산하는 로드 밸런싱 담당

### 임의로 할당된 16진수를 출력하는 PHP 파일이 들어있는 웹 서버 이미지를 실행해보는 예제

```bash
# 매니저에서 진행함
docker service create --name hostname -p 8000:80 --replicas=4 alicek106/book:hostname

tav60h70zhufz6nlyes30lxul
overall progress: 4 out of 4 tasks
1/4: running   [==================================================>]
2/4: running   [==================================================>]
3/4: running   [==================================================>]
4/4: running   [==================================================>]
verify: Service converged
```

생성된 컨테이너 목록 확인 _> replicas 를 4로 설정하였으므로 컨테이너 4개 생성

```bash
docker ps --format "table {{.ID}}\\t{{.Status}}\\t{{.Image}}"
[root@swarm-manager ~]# docker ps --format "table {{.ID}}\\t{{.Status}}\\t{{.Image}}"
CONTAINER ID        STATUS              IMAGE
e6d63b4a579d        Up 2 minutes        alicek106/book:hostname
6fd474eb7479        Up About an hour    nginx:latest
ea9cac0d3b8e        Up About an hour    nginx:latest
9de652dc2e01        Up About an hour    nginx:latest
68781f8fbb55        Up 2 hours          mysql:5.7
f9149ffd1053        Up 2 hours          nginx:latest
[root@swarm-worker1 tmp]# docker ps --format "table {{.ID}}\\t{{.Status}}\\t{{.Image}}"
CONTAINER ID        STATUS              IMAGE
9753e8204809        Up 2 minutes        alicek106/book:hostname
fd9cd69a0934        Up 2 minutes        alicek106/book:hostname
[root@swarm-worker2 vagrant]# docker ps --format "table {{.ID}}\\t{{.Status}}\\t{{.Image}}"
CONTAINER ID        STATUS              IMAGE
7c01a11f144b        Up 2 minutes        alicek106/book:hostname
```

### ingress 네트워크를 사용하지 않고 호스트의 8080 포트를 직접 컨테이너의 80 포트로 연결하는 방법

어느 호스트에서 컨테이너가 생성될지 알 수 없어 포트 및 서비스 관리가 어렵다는 단점이 있다.

가급적이면 ingress 네트워크를 사용해 외부로 서비스를 노출하는것을 지향

```bash
[root@swarm-manager ~]# docker service create --publish mode=host,target=80,published=8080,protocol=tcp --name webnginx nginx
hzs1m0ihikjhibsobrdmotz57
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged
```

## 오버레이 네트워크

오버레이 네트워크는 여러 개의 도커 데몬을 하나의 네트워크 풀로 만드는 네트워크 가상화 기술의 하나로, 도커에 오버레이 네트워크를 적용하면 여러 도커 데몬에 존재하는 컨테이너가 서로 통신할 수 있다.

여러 개의 스웜 노드에 할당된 컨테이너는 오버레이 네트워크의 서브넷에 해당하는 IP 대역을 할당받고 이 IP를 통해 서로 통신할 수 있다.

### 스웜 클러스터 내의 컨테이너가 할당받은 IP 주소 확인

eth0 : ingress 네트워크와 연결된 NIC

ingress 네트워크는 오버레이 네트워크 드라이버를 사용한다.

```bash
[root@swarm-manager ~]# docker ps --format "table {{.ID}}\\t{{.Status}}\\t{{.Image}}"
CONTAINER ID        STATUS              IMAGE
e6d63b4a579d        Up 21 minutes       alicek106/book:hostname
68781f8fbb55        Up 3 hours          mysql:5.7
[root@swarm-manager ~]# docker exec e6d ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:0a:ff:00:a0
          inet addr:10.255.0.160  Bcast:10.255.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:129 errors:0 dropped:0 overruns:0 frame:0
          TX packets:77 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:28855 (28.8 KB)  TX bytes:28481 (28.4 KB)

eth1      Link encap:Ethernet  HWaddr 02:42:ac:12:00:07
          inet addr:172.18.0.7  Bcast:172.18.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:780 (780.0 B)  TX bytes:372 (372.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:764 (764.0 B)  TX bytes:764 (764.0 B)
[root@swarm-worker1 tmp]# docker ps --format "table {{.ID}}\\t{{.Status}}\\t{{.Image}}"
CONTAINER ID        STATUS              IMAGE
9753e8204809        Up 25 minutes       alicek106/book:hostname
fd9cd69a0934        Up 25 minutes       alicek106/book:hostname
[root@swarm-worker1 tmp]# docker exec 975 ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:0a:ff:00:a1
          inet addr:10.255.0.161  Bcast:10.255.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:26 errors:0 dropped:0 overruns:0 frame:0
          TX packets:16 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:6149 (6.1 KB)  TX bytes:6482 (6.4 KB)

eth1      Link encap:Ethernet  HWaddr 02:42:ac:12:00:03
          inet addr:172.18.0.3  Bcast:172.18.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:9 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:822 (822.0 B)  TX bytes:372 (372.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:764 (764.0 B)  TX bytes:764 (764.0 B)
[root@swarm-worker2 vagrant]# docker exec 7c01 ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:0a:ff:00:a3
          inet addr:10.255.0.163  Bcast:10.255.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

eth1      Link encap:Ethernet  HWaddr 02:42:ac:12:00:03
          inet addr:172.18.0.3  Bcast:172.18.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:6 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:672 (672.0 B)  TX bytes:372 (372.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:764 (764.0 B)  TX bytes:764 (764.0 B)
```

### 별도의 포트 포워딩을 하지 않아도 컨테이너 간 전송이 가능하다.

```bash
# 매니저 노드 컨테이너에서 swarm-worker2 노드의 컨테이너로 ping
```

### docker_gwbridge 네트워크

오버레이 네트워크를 사용하지 않는 컨테이너는 기본적으로 존재하는 브리지(bridge) 네트워크를 사용해 외부와 연결한다. 그러나 ingress 를 포함한 모든 오버레이 네트워크는 브리지 네트워크와 다른 docker_gwbridge 네트워크와 함께 사용된다.

docker_gwbridge 네트워크는 외부로 나가는 통신 및 오버레이 네트워크의 트랙빅 종단점(VTEP) 역할을 담당한다. 컨테이너 내부의 네트워크 인터페이스 카드 중 eth1 과 연결됨

## 사용자 정의 오버레이 네트워크

```bash
docker network create --subnet 10.0.9.0/24 -d overlay myoverlay
																											# 오버레이 네트워크 이름 설정
																           # 네트워크 드라이버를 overlay로 설정한다.
                      # 오버레이 네트워크의 서브넷 설정

# 오버레이 네트워크 확인
docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
44cb65b8699a        bridge              bridge              local
097e998b1988        docker_gwbridge     bridge              local
9c83a0b1501f        host                host                local
2jdgw8i2ktht        ingress             overlay             swarm    # swarm 클러스터에만 적용
m6co5qw6r0vn        myoverlay           overlay             swarm    # 된 것을 확인할 수 있다.
ff2e7e9aab2d        none                null                local
```

### 오버레이 네트워크를 서비스에 적용(--network 옵션 이용) 하여 컨테이너 생성

```bash
docker service create --name overlay_service --network myoverlay --replicas 2 alicek106/book:hostname
```

### 생성된 컨테이너에 할당된 오버레이 네트워크 IP 주소 확인 (eh0)

```bash
[root@swarm-manager ~]# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                     PORTS

[root@swarm-manager ~]# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                     PORTS
tav60h70zhuf        hostname            replicated          4/4                 alicek106/book:hostname   *:8000->80/tcp
xh56mm3s2qwf        mysql               replicated          1/1                 mysql:5.7
pyspvrzwya8k        overlay_service     replicated          2/2                 alicek106/book:hostname
hzs1m0ihikjh        webnginx            replicated          1/1                 nginx:latest
cjk7q8uhi7i7        yml_registry        replicated          0/1                 config:latest             *:5000->5000/tcp

[root@swarm-manager ~]# docker service ps overlay_service
ID                  NAME                IMAGE                     NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
wa0b3bljyzp1        overlay_service.1   alicek106/book:hostname   swarm-worker2       Running             Running 3 minutes ago
pdzmkuyk3azq        overlay_service.2   alicek106/book:hostname   swarm-manager       Running             Running 3 minutes ago

[root@swarm-manager ~]# docker ps | grep pdzmk
e60e3be867e3        alicek106/book:hostname   "apachectl -DFOREGRO…"   4 minutes ago       Up 4 minutes        80/tcp                overlay_service.2.pdzmkuyk3azqimmfzqip4f9t8

[root@swarm-manager ~]# docker exec overlay_service.2.pdzmkuyk3azqimmfzqip4f9t8 ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:0a:00:09:04
          inet addr:10.0.9.4  Bcast:10.0.9.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

eth1      Link encap:Ethernet  HWaddr 02:42:ac:12:00:03
          inet addr:172.18.0.3  Bcast:172.18.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

# 서비스 디스커버리

## server 서비스 생성과 client 서비스 생성

```bash
# 예제 서비스에서 사용할 오버레이 네트워크 생성
[root@swarm-manager ~]# docker network create -d overlay discovery
6z8eqhxwodwq6dsjomnqspye4

# server 서비스 생성 - 컨테이너 호스트 이름을 출력하는 웹 서버 컨테이너 2개 생성
[root@swarm-manager ~]# docker service create --name server --replicas 2 --network discovery alicek106/book:hostname
x4x0o2k052h9krpjrsd5y8h48
overall progress: 2 out of 2 tasks
1/2: running   [==================================================>]
2/2: running   [==================================================>]
verify: Service converged

# client 서비스 생성 - server 서비스에 http 요청을 보낼 컨테이너 생성
[root@swarm-manager ~]# docker service create --name client --network discovery alicek106/book:curl ping docker.com
ic3yaescdp9pcadvzpzdxr1be
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged

# client 서비스 컨테이너 실행 노드 확인
[root@swarm-manager ~]# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                     PORTS
ic3yaescdp9p        client              replicated          1/1                 alicek106/book:curl

[root@swarm-manager ~]# docker service ps client
ID                  NAME                IMAGE                 NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
uary4kcweihr        client.1            alicek106/book:curl   swarm-worker1       Running             Running about a minute ago
# swarm-worker1 노드에 할당되어 작동을 확인
```

## client 서비스가 할당된 노드에서 클라이언트 기능 확인

```bash
# client 서비스 컨테이너가 생성된 노드에서 컨테이너 ID 확인
[root@swarm-worker1 tmp]# docker ps --format "table {{.ID}}\\t{{.Image}}\\t{{.Command}}"
CONTAINER ID        IMAGE                     COMMAND
e1eef8bb14e2        alicek106/book:curl       "ping docker.com"

# 컨테이너 내부 진입
[root@swarm-worker1 tmp]# docker exec -it e1ee bash
root@e1eef8bb14e2:/#

# 컨테이너 내부에서 curl 명령으로 server 서비스에 접근
root@e1eef8bb14e2:/# curl -s server | grep Hello
        <p>Hello,  e66e4e3e1846</p>     </blockquote>

## 오버레이 네트워크에 속하도록 서비스를 생성했다면 도커 스웜의 DNS가 이를 자동 변환(resolve)
## 요청을 보낼 때 마다 다른 컨테이너에 접근하는 것을 확인 라운드 로빈 방식
```

## server 서비스의 컨테이너 레플리카 수를 늘림

```bash
[root@swarm-manager ~]# docker service scale server=3
server scaled to 3
overall progress: 3 out of 3 tasks
1/3: running   [==================================================>]
2/3: running   [==================================================>]
3/3: running   [==================================================>]
verify: Service converged

# 핑을 보내면 새로운곳에도 가는걸 확인할 수 있다.
```

### server 서비스의 VIP 확인

```bash
[root@swarm-manager ~]# docker service inspect --format {{.Endpoint.VirtualIPs}} server
[{6z8eqhxwodwq6dsjomnqspye4 10.0.0.2/24}]
```

server 라는 호스트 이름이 여러개의 IP를 가지고 있는 것이 아니라 서비스의 VIP(Virual IP)를 가지고 있다.

스웜 모드가 활성화된 도커 엔진의 내장 DNS 서버는 server라는 호스트 이름을 10.0.0.2 라는 IP로 변환

컨테이너 네트워크 네임스페이스 내부에서 실제 server 서비스의 컨테이너 IP로 포워딩

## VIP 방식이 아닌 도커의 내장 DNS 서버 기반으로 라운드 로빈 적용

```bash
[root@swarm-manager ~]# docker service create --name server-dnsrr --replicas 2 --network discovery --endpoint-mode dnsrr alicek106/book:hostname
xeewqmqgsjyszouix8irr3bxt
overall progress: 2 out of 2 tasks
1/2: running   [==================================================>]
2/2: running   [==================================================>]
verify: Service converged
# 애플리케이션에 따라 캐시 문제가 발생 할 수 있어 VIP 를 권장한다.
```

# 스웜 모드 볼륨

도커 데몬 명령어 중 run 명령에서 -v 옵션을 사용할 때는 호스트와 디렉토리를 공유하는 경우와 볼륨을 사용하는 경우에 대한 구분이 없다.

스웜 모드에서는 서비스를 생성할 때 도커 볼륨을 사용할지, 도커와 디렉토리를 공유할지 명시한다.

## volume 타입의 볼륨 생성

```bash
docker service create --name ubuntu --mount type=volume,source=myvol,target=/root ubuntu:14.04 ping docker.com
```

`--mount` 옵션의 type 값에 volume을 지정 : 도커 볼륨을 사용하는 서비스 생성

`source` : 사용할 볼륨, 도커 데몬에 해당 볼륨이 존재하면 해당 볼륨을 사용하고 없으면 새로 생성

source 옵션을 명시하지 않으면 임의의 16진수로 구성된 익명의 이름을 가진 볼륨 생성

target : 컨테이너 내부에 마운트될 디렉토리 위치

서비스의 컨테이너에서 볼륨에 공유할 컨테이너의 디렉토리에 파일이 이미 존재하면 이 파일들은 볼륨에 복사되고, 호스트에서 별도의 공간을 차지하게 된다.

서비스를 생성하는 볼륨 옵션에 `volume-nocopy` 를 추가하면 source쪽의 파일들이 볼륨(컨테이너쪽)에 복사되지 않는다. 백업용으로만 사용되는것.

볼륨을 사용하는 컨테이너 생성

```bash
docker run -ti --name iusetest -v test:/root ubuntu:14.04

# iusetest 에서 ls root
ls root
vimrc vimrc.tiny
```

## bind 타입의 볼륨 생성

바인드 타입은 호스트와 디렉토리를 공유할 때 사용한다.

type 옵션의 값을 bind 로 설정. 볼륨 타입과 달리 공유될 호스트의 디렉토리를 설정해야 하므로 source 옵션을 반드시 명시

```bash
docker service create --name ubuntu --mount type=bind,src=/home/yooa/host,dst=/root/container ubuntu:14.04 ping docker.com
```

## 스웜 모드에서 볼륨의 한계

서비스를 할당받을 수 있는 모든 노드가 볼륨 데이터를 가지고 있어야 하므로 스웜 클러스터에서 볼륨을 사용하는 것은 어렵다. 왜냐? 로컬 볼륨이 존재하지 않는 노드에는 컨테이너가 할당될 수 없어서

우회법은 어느 노드에서나 접근 가능한 퍼시스턴트 스토리지(Persistent Storage)를 사용하는것이다.

### 퍼시스턴트 스토리지

호스트, 컨테이너와 별개로 외부에 존재해 네트워크로 마운트할 수 있는 스토리지

도커 자체가 제공하지 않으므로 사용자가 구성해야함.

# 도커 스웜 모드 노드 다루기

마스터 노드는 최대한 부하를 받지 않도록 서비스를 할당받지 않게 하거나, 문제가 발생한 특정 노드를 유지보수할 때 해당 노드에 컨테이너를 할당하지 않게 만들고 싶을 때 등 docker node update --availabilty 명령으로 노드의 상태를 변경할 수 있다.

## 구축한 스웜 클러스터의 노드 상태 확인

```bash
docker node ls
```

## Active 상태

```bash
docker node update --availablity active node_name
```

새로운 노드가 스웜 클러스터에 추가되면 기본적으로 설정되는 상태로 노드가 서비스의 컨테이너를 할당받을 수 있음을 의미한다.

## Drain 상태

```bash
docker node update --availablity drain node_name
```

스웜 매니저의 스케줄러는 컨테이너를 drain 노드에 할당하지 않는다. 일반적으로 매니저 노드 또는 노드에 문제가 생겨 일시적으로 사용하지 않는 상태로 설정할 때 사용

노드를 Drain 상태로 변경하면 해당 노드에서 실행 중이던 서비스의 컨테이너는 전부 중지되고  Active 상태의 노드로 다시 할당된다. Drain 상태의 노드를 Active 상태로 다시 변경한다고 해서 서비스의 컨테이너가 다시 분산되어 할당되지 않으므로 `docker service scale` 명령을 사용해 컨테이너의 균형을 재조정 해야한다.

## Pause 상태

```bash
docker node update --availablity pause node_name
```

서비스 컨테이너를 더는 할당받지 않는다는 점에서 Drain과 같지만 실행 중인 컨테이너가 중지되지 않는다.

# 노드 라벨 추가

특정 노드에 라벨을 추가하면 서비스를 할당할 때 컨테이너를 생성할 노드의 그룹을 지정하는 것이 가능하다.

`docker node update --label-add` 옵션을 사용해 라벨을 설정한다.

```bash
docker node update --label-add storage=ssd swrarm-worker1
# 라벨 추가를 확인
docker node inspect --pretty swarm-worker1
[...]
Labels:
 - storage=ssd
[...]
```

## 서비스 제약 설정

docker service create 명령어에 --constraint 옵션을 추가해 서비스의 컨테이너가 할당될 노드의 종류를 지정할 수 있다.

### node.labels 제약조건

storage 키의 값이 ssd로 설정된 노드에 서비스의 컨테이너를 할당하도록 제한

```bash
docker service create --name label_test --constraint 'node.labels.storage == ssd' --replicas=5 ubuntu:14:04 ping docker.com

# 할당 확인
docker service ps label_test

# storage 라벨이 ssd로 설정된 노드에만 컨테이너가 생성된다.
# 제약조건을 만족하는 노드를 찾지 못할 경우 서비스의 컨테이너는 생성되지 않는다.
```

### [node.id](http://node.id) 제약조건

노드 ID를 명시해 서비스의 컨테이너를 할당할 노드를 선택한다.

다른 도커 명령어와 달리 docker node ls 명령어에 출력된 ID를 전부 입력해야 한다.

```bash
docker node ls | grep swarm-worker2
docker service create --constraint 'node.id == fullnodename' --replicas=5 --name label_test2 ubuntu:14.04 ping docker.com
```

### node.hostname 과 node.role 제약조건

스웜 클러스터에 등록된 이름 및 역할로 제약 조건을 설정

```bash
# host name으로 지정
docker service create --constraint 'node.hostname == swarm-worker1' --replicas=5 --name label_test3 ubuntu:14.04 ping docker.com

# role 로 지정
docker service create --constraint 'node.role == manager' --replicas=5 --name label_test4 ubuntu:14.04 ping docker.com
docker service create --constraint 'node.role != manager' --replicas=5 --name label_test5 ubuntu:14.04 ping docker.com
```

### engine.labels 제약조건

도커 엔진, 즉 도커 데몬 자체에 라벨을 설정해 제한 조건을 설정. 도커 데몬 실행 옵션을 변경해야 한다.

도커 데몬에 설정된 라벨은 `docker info` 명령으로 확인 가능하다.

```bash
# 도커 데몬의 라벨 중 mylabel 이라는 키가 worker2 값이라는 노드에 서비스 컨테이너를 할당
docker service create --constraint 'engine.labels.mylabel == worker2' --name engine_label --replicas=5 ubuntu:14.04 ping docker.com
```

### 여러개의 제약조건을 동시에 설정

`--constraint 명령문` 의 반복으로 여러 제약조건을 동시에 설정할 수 있다.