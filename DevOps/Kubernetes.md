# Kubernets

# 쿠버네티스

대부분의 리소스를 `오브젝트` 라고 불리는 형태로 관리한다.

쿠버네티스에서는 컨테이너의 집합(pods), 컨테이너의 집합을 관리하는(replica set), 사용자(service account), 노드(node) 까지도 하나의 오브젝트로 사용할 수 있다.

kubectl 명령어 또는 YAML 파일로 정의하여 쿠버네티스를 사용한다.

## 쿠버네티스 실습 환경 설정 (우분투)

1. 가상머신 생성

Vagrantfile

```bash
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.hostname = "ubuntu"
  config.vm.network "private_network", ip: "192.168.111.110"
  config.vm.synced_folder ".", "/home/vagrant/sync", disabled: true
  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 2
    vb.memory = 2048
  end
end
```

1. 패키지 최신화 및 도커 설치, 설정

```bash
apt update
apt upgrade

apt install docker.io -y
usermod -a -G docker $USER
service docker restart
chmod 666 /var/run/docker.sock
docker version
```

1. kubectl 설치

```bash
# <https://kubernetes.io/ko/docs/tasks/tools/install-kubectl/>

# 공식 레퍼 따른다. 끝난 후 확인
kubectl version
```

1. Minikube 설치

```bash
# <https://kubernetes.io/ko/docs/tasks/tools/install-minikube/>

# 공식 레퍼 따른다.
# 클러스터 시작
minikube start

# 클러스터 정지
minikube stop

#클러스터 전부 삭제
minikube delete

# 확인
minikube status

kubectl version --short
```

## 쿠버네티스에서 사용할 수 있는 오브젝트 확인

```
kubectl api-resources
```

## 포드(pod)

컨테이너 애플리케이션의 기본 단위

1개 이상의 컨테이너로 구성된 컨테이너의 집합

여러 개의 컨테이너를 추상화해서 하나의 애플리케이션으로 동작하도록 묶어 놓은 컨테이너의 묶음

### nginx 컨테이너로 구성된 포드를 생성

# nginx-pod.yml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx-container
spec:
  containers:
  - name: my-nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
      protocol: TCP
# YAML 파일에서 정의한 오브젝트의 API 버전
# 리소스의 종류
# 라벨, 주석, 이름과 같은 리소스의 부가 정보

# 컨테이너 리소스 생성을 위한 정보
```

## 포드 생성

```
kubectl apply -f <YML_FILE_NAME>
# 새로운 포드 생성
vagrant@ubuntu:~/kub01$ kubectl apply -f nginx-pod.yml
pod/my-nginx-container created
```

## 모든 포드 확인

```
kubectl get pods
# 포드 확인
vagrant@ubuntu:~/kub01$ kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
my-nginx-container   1/1     Running   0          63s

# 축약어를 사용할 수 있다
vagrant@ubuntu:~/kub01$ kubectl get po
NAME                 READY   STATUS    RESTARTS   AGE
my-nginx-container   1/1     Running   0          68s
```

## 상세 정보 조회

```
kubectl describe pods <POD_NAME>
# 생성된 리소스의 자세한 정보 확인
vagrant@ubuntu:~/kub01$ kubectl describe pods my-nginx-container
Name:         my-nginx-container
Namespace:    default
Priority:     0
Node:         minikube/172.17.0.3
Start Time:   Fri, 18 Sep 2020 06:39:13 +0000
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           172.18.0.3
IPs:
  IP:  172.18.0.3
Containers:
  my-nginx-container:
# <이하생략>
```

포드 컨테이너 생성과 확인 실습

```bash
# 클러스터 내부에 테스트를 위한 임시 포드를 생성
# alicek106/ubuntu:curl 이미지를 이용해서 debug 이름의 포드를 생성
vagrant@ubuntu:~/kub01$ kubectl run -it --rm debug --image=alicek106/ubuntu:curl --restart=Never bash
If you don't see a command prompt, try pressing enter.
root@debug:/#
# debug 포드가 생성 된 상태에서 포드 조회
vagrant@ubuntu:~/kub01$ kubectl get pod
NAME                 READY   STATUS    RESTARTS   AGE
debug                1/1     Running   0          73s
my-nginx-container   1/1     Running   0          8m36s

# nginx 포드의 동작을 확인 (172.18.0.3)
root@debug:/# curl 172.18.0.3
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
# <이하생략>
```

## kubectl exec 명령으로 포드의 컨테이너에 명령을 전달

```
kubectl exec -it <POD_NAME> -- <COMAND>
vagrant@ubuntu:~/kub01$ kubectl exec -it my-nginx-container -- bash
root@my-nginx-container:/# whoami
root
root@my-nginx-container:/# exit
exit
```

## kubectl logs 명령으로 포드 로그 확인

```
kubectl log <POD_NAME>
vagrant@ubuntu:~/kub01$ kubectl logs my-nginx-container
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
172.18.0.4 - - [18/Sep/2020:06:48:42 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.35.0" "-"
```

## 쿠버네티스 오브젝트를 YAML을 이용해 삭제

```
kubectl delete -f <YAML_FILE_NAME>
vagrant@ubuntu:~/kub01$ kubectl delete -f nginx-pod.yml
pod "my-nginx-container" deleted

# 모든 pod 조회
vagrant@ubuntu:~/kub01$ kubectl get pods
No resources found in default namespace.
vagrant@ubuntu:~/kub01$
```

1개의 pod안에 여러개의 컨테이너가 있는 경우 실습

# nginx-pod-with-ubuntu.yml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx-pod
spec:
  containers:
- name: my-nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
      protocol: TCP

  - name: ubuntu-sidecar-container
    image: alicek106/rr-test:curl
    command: [ "tail" ]
    args: [ "-f", "/dev/null" ]
```

pod 생성

```bash
vagrant@ubuntu:~/kub01$ kubectl apply -f nginx-pod-with-ubuntu.yml
pod/my-nginx-pod created
vagrant@ubuntu:~/kub01$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
my-nginx-pod   2/2     Running   0          20s
```

- pod로 명령어를 전달할 때 실행할 컨테이너를 지정 안하면 default 설정 컨테이너가 실행됨

```bash
vagrant@ubuntu:~/kub01$ kubectl exec -it my-nginx-pod -- bash
Defaulting container name to my-nginx-container.
Use 'kubectl describe pod/my-nginx-pod -n default' to see all of the containers in this pod.
root@my-nginx-pod:/# exit
exit
```

- 특정 컨테이너에게 명령어를 전달 할 때는 `-c` 옵션을 사용한다.

```bash
vagrant@ubuntu:~/kub01$ kubectl exec -it my-nginx-pod -c ubuntu-sidecar-container -- bash
root@my-nginx-pod:/#

root@my-nginx-pod:/# curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>

# 이 실습으로 알 수 있는점: 하나의 Pod 내에서는 네트워크 네임스페이스를 공유하는것을 알 수 있다.
```

## 레플리카셋(Replica Set)

똑같은 pod의 레플리케이션 개수를 관리 및 제어하는 리소스.

보장하는 개수의 기준은 LABELS을 보고 판단한다. NAME이 아님

```bash
minikube status
# minikube 의 상태 확인

minikube start
minikube status
# minikube 시작 후 다시 확인

kubectl get pods
# 실행중인 포드 확인
kubectl delets pods <POD_NAME>
# 실행중인 포드 삭제

# pod 삭제 방법1
kubectl delete -f <YAML_FILE_NAME>
# pod 생성
kubectl apply -f <YAML_FILE_NAME>

# pod 삭제 방법2
kubectl delete pods <POD_NAME>
```

### 레플리카셋 정의

실습을 위한 세팅 생성 vim replicaset-nginx.yml

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-nginx-pods-label
  template:
    metadata:
      name: my-nginx-pod
      labels:
        app: my-nginx-pods-label
    spec:
      containers:
      - name: my-nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
          protocol: TCP
```

### 레플리카셋 생성 및 확인

```bash
kubectl apply -f replicaset-nginx.yml
kubectl get pod
kubectl get replicase

vagrant@ubuntu:~/kub01$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
replicaset-nginx-df4nw   1/1     Running   0          25m
replicaset-nginx-qf8cq   1/1     Running   0          25m
replicaset-nginx-spzvx   1/1     Running   0          25m
vagrant@ubuntu:~/kub01$ kubectl get replicaset
NAME               DESIRED   CURRENT   READY   AGE
replicaset-nginx   3         3         3       25m
```

### Pod 개수를 늘려 실행

yml 파일의  spec 어트리뷰트의 replicas 수를 변경

kubectl apply -f YAML_FILE_NAME

### 레플리카셋을 삭제

레플리카셋을 삭제하면  Pod 도 함께 삭제된다.

```bash
kubectl delete replicaset <REPLICASET_NAME>
# 또는
kubectl delete rs <REPLICASET_NAME>

# YAML 파일을 이용한 삭제 방법
kubectl delete -f <YAML_FILE_NAME>
```

### 레플리카셋은 셀렉터로 정의한 pod 가 일정 개수가 되도록 유지

1. my-nginx-pods-label 라벨을 가지는 파드를 생성

```bash
vagrant@ubuntu:~/kub01$ vi nginx-pod-without-rs.yml

`YAML
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx-pod
  labels:
    app: my-nginx-pods-label
spec:
  containers:
  - name: my-nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
`

vagrant@ubuntu:~/kub01$ kubectl apply -f nginx-pod-without-rs.yml
pod/my-nginx-pod created

vagrant@ubuntu:~/kub01$ kubectl get pods --show-labels
NAME           READY   STATUS    RESTARTS   AGE   LABELS
my-nginx-pod   1/1     Running   0          19s   app=my-nginx-pods-label
```

1. my-nginx-pods-label 라벨을 가지는 파드 3개를 생성하는 레플리카셋을 생성

~~~bash
vagrant@ubuntu:~/kub01$ vi replicaset-nginx.yml
```YAML
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-nginx-pods-label
  template:
    metadata:
      name: my-nginx-pod
      labels:
        app: my-nginx-pods-label
    spec:
      containers:
      - name: my-nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
          protocol: TCP
~~~

vagrant@ubuntu:~/kub01$ kubectl apply -f replicaset-nginx.yml replicaset.apps/replicaset-nginx created

vagrant@ubuntu:~/kub01$ kubectl get pods --show-labels NAME                     READY   STATUS    RESTARTS   AGE     LABELS my-nginx-pod             1/1     Running   0          4m12s   app=my-nginx-pods-label replicaset-nginx-59992   1/1     Running   0          27s     app=my-nginx-pods-label replicaset-nginx-gbzrr   1/1     Running   0          27s     app=my-nginx-pods-label

```
3. pod 수동으로 삭제해보고 조회

​```bash
vagrant@ubuntu:~/kub01$ kubectl delete pods my-nginx-pod
pod "my-nginx-pod" deleted

vagrant@ubuntu:~/kub01$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
replicaset-nginx-59992   1/1     Running   0          4m40s
replicaset-nginx-gbzrr   1/1     Running   0          4m40s
replicaset-nginx-vfjqn   1/1     Running   0          14s
```

1. 레플리카셋이 생성한 pod의 라벨 변경

라벨 주석처리 해본다.

```bash
vagrant@ubuntu:~/kub01$ kubectl edit pods replicaset-nginx-59992
`YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-09-21T02:18:06Z"
  generateName: replicaset-nginx-
  #  labels:
  #    app: my-nginx-pods-label
  name: replicaset-nginx-59992
<이하생략>
`

vagrant@ubuntu:~/kub01$ kubectl get pods --show-labels
NAME                     READY   STATUS    RESTARTS   AGE     LABELS
replicaset-nginx-59992   1/1     Running   0          9m31s   <none>
replicaset-nginx-bhz8w   1/1     Running   0          56s     app=my-nginx-pods-label
replicaset-nginx-gbzrr   1/1     Running   0          9m31s   app=my-nginx-pods-label
replicaset-nginx-vfjqn   1/1     Running   0          5m5s    app=my-nginx-pods-label
```

1. 레플리카셋 삭제

결과 예상 : 같은 라벨 pod 삭제

```bash
vagrant@ubuntu:~/kub01$ kubectl get replicaset
NAME               DESIRED   CURRENT   READY   AGE
replicaset-nginx   3         3         3       12m

vagrant@ubuntu:~/kub01$ kubectl get pods --show-labels
NAME                     READY   STATUS    RESTARTS   AGE     LABELS
replicaset-nginx-59992   1/1     Running   0          12m     <none>
replicaset-nginx-bhz8w   1/1     Running   0          3m49s   app=my-nginx-pods-label
replicaset-nginx-gbzrr   1/1     Running   0          12m     app=my-nginx-pods-label
replicaset-nginx-vfjqn   1/1     Running   0          7m58s   app=my-nginx-pods-label

vagrant@ubuntu:~/kub01$ kubectl delete replicasets replicaset-nginx
replicaset.apps "replicaset-nginx" deleted

vagrant@ubuntu:~/kub01$ kubectl get replicaset
No resources found in default namespace.

vagrant@ubuntu:~/kub01$ kubectl get pods --show-labels
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
replicaset-nginx-59992   1/1     Running   0          13m   <none>
```

1. 라벨이 삭제된 파드는 직접 삭제

```bash
vagrant@ubuntu:~/kub01$ kubectl delete pods replicaset-nginx-59992
pod "replicaset-nginx-59992" deleted

vagrant@ubuntu:~/kub01$ kubectl get pods --show-labels
No resources found in default namespace.
```

## 디플로이먼트(Deployment)

https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/

레플리카셋을 관리하고 컨트롤한다. 애플리케이션의 업데이트와 배포를 쉽게 하기 위해 만든 개념

### 디플로이먼트 생성 확인

```bash
vagrant@ubuntu:~/kub01$ vi deployment-nginx.yml
`YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-nginx
  template:
    metadata:
      name: my-nginx-pod
      labels:
        app: my-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.10
        ports:
        - containerPort: 80
`
vagrant@ubuntu:~/kub01$ kubectl apply -f deployment-nginx.yml
deployment.apps/my-nginx-deployment created

vagrant@ubuntu:~/kub01$ kubectl get deployment
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
my-nginx-deployment   3/3     3            3           17s

vagrant@ubuntu:~/kub01$ kubectl get replicasets
NAME                             DESIRED   CURRENT   READY   AGE
my-nginx-deployment-7484748b57   3         3         3       30s

vagrant@ubuntu:~/kub01$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
my-nginx-deployment-7484748b57-dlhbn   1/1     Running   0          38s
my-nginx-deployment-7484748b57-sc6bk   1/1     Running   0          38s
my-nginx-deployment-7484748b57-vr74n   1/1     Running   0          38s
```

### 디플로이먼트를 삭제했을 때, replicaset, pod도 함께 삭제됨 확인

```bash
vagrant@ubuntu:~/kub01$ kubectl delete deployment my-nginx-deployment
deployment.apps "my-nginx-deployment" deleted

vagrant@ubuntu:~/kub01$ kubectl get deployment
No resources found in default namespace.

vagrant@ubuntu:~/kub01$ kubectl get replicasets
No resources found in default namespace.

vagrant@ubuntu:~/kub01$ kubectl get pods
No resources found in default namespace.
```

### 디플로이먼트 사용 이유?

디플로이먼트는 컨테이너 애플리케이션을 배포하고 관리하는 역할

애플리케이션을 업데이트 할 때 레플리카셋의 변경 사항을 저장하는 리비전을 남겨 롤백 가능하게 해 주고, 무중단 서비스를 위한 포드의 롤링 업데이트 전략을 지정할 있음

### --record 옵션을 추가해 디플로이먼트 생성

```bash
vagrant@ubuntu:~/kub01$ kubectl apply -f deployment-nginx.yml --record
deployment.apps/my-nginx-deployment created

vagrant@ubuntu:~/kub01$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
my-nginx-deployment-7484748b57-8ph87   1/1     Running   0          8s
my-nginx-deployment-7484748b57-fkc2q   1/1     Running   0          8s
my-nginx-deployment-7484748b57-v7hkr   1/1     Running   0          8s
```

### kubectl set image 명령으로 pod 이미지 변경

```bash
vagrant@ubuntu:~/kub01$ kubectl set image deployment my-nginx-deployment nginx=nginx:1.11 --record
                                                     디플로이먼트 이름    컨테이너 이름
deployment.apps/my-nginx-deployment image updated

vagrant@ubuntu:~/kub01$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
my-nginx-deployment-556b57945d-mlj7s   1/1     Running   0          14s
my-nginx-deployment-556b57945d-sklj2   1/1     Running   0          6s
my-nginx-deployment-556b57945d-tdg2t   1/1     Running   0          4s
```

pods 의 NAME을 보면 바뀐것을 확인 할 수 있다. 실습 예제에선 nginx 버전을 바꿨기 때문에 기존 pod이 전부 내려가고 새로운 pod 가 replicasetd규칙에 의해 3개가 올라옴

replicas 값만 변경해서는 레플리카셋의 교체가 일어나지는 않는다.

### 리비전 정보를 확인

--record=true 옵션으로 디플로이먼트를 변경하면 변경 사항을 기록하여 해당 버전의 레플리카셋을 보관할 수 있다.

```bash
vagrant@ubuntu:~/kub01$ kubectl rollout history deployment my-nginx-deployment
deployment.apps/my-nginx-deployment
REVISION  CHANGE-CAUSE
1         kubectl apply --filename=deployment-nginx.yml --record=true
2         kubectl set image deployment my-nginx-deployment nginx=nginx:1.11 --record=true
```

### 이전 버전의 레플리카셋으로 롤백

```bash
vagrant@ubuntu:~/kub01$ kubectl rollout undo deployment my-nginx-deployment --to-revision=1
deployment.apps/my-nginx-deployment rolled back

vagrant@ubuntu:~/kub01$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
my-nginx-deployment-7484748b57-2mks2   1/1     Running   0          12s
my-nginx-deployment-7484748b57-2xpsw   1/1     Running   0          15s
my-nginx-deployment-7484748b57-ngcd4   1/1     Running   0          14s

vagrant@ubuntu:~/kub01$ kubectl get replicasets
NAME                             DESIRED   CURRENT   READY   AGE
my-nginx-deployment-556b57945d   0         0         0       11m
my-nginx-deployment-7484748b57   3         3         3       15m

vagrant@ubuntu:~/kub01$ kubectl rollout history deployment my-nginx-deployment
deployment.apps/my-nginx-deployment
REVISION  CHANGE-CAUSE
2         kubectl set image deployment my-nginx-deployment nginx=nginx:1.11 --record=true
3         kubectl apply --filename=deployment-nginx.yml --record=true
```

### 디플로이먼트 상세 정보 확인

```bash
vagrant@ubuntu:~/kub01$ kubectl describe deploy my-nginx-deployment
Name:                   my-nginx-deployment
Namespace:              default
CreationTimestamp:      Mon, 21 Sep 2020 04:13:55 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 3
                        kubernetes.io/change-cause: kubectl apply --filename=deployment-nginx.yml --record=true
Selector:               app=my-nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=my-nginx
  Containers:
   nginx:
    Image:        nginx:1.10
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   my-nginx-deployment-7484748b57 (3/3 replicas created)
Events:
  Type    Reason             Age                  From                   Message
  ----    ------             ----                 ----                   -------
  Normal  ScalingReplicaSet  15m                  deployment-controller  Scaled up replica set my-nginx-deployment-556b57945d to 1
  Normal  ScalingReplicaSet  15m                  deployment-controller  Scaled down replica set my-nginx-deployment-7484748b57 to 2
  Normal  ScalingReplicaSet  15m                  deployment-controller  Scaled up replica set my-nginx-deployment-556b57945d to 2
  Normal  ScalingReplicaSet  14m                  deployment-controller  Scaled down replica set my-nginx-deployment-7484748b57 to 1
  Normal  ScalingReplicaSet  14m                  deployment-controller  Scaled up replica set my-nginx-deployment-556b57945d to 3
  Normal  ScalingReplicaSet  14m                  deployment-controller  Scaled down replica set my-nginx-deployment-7484748b57 to 0
  Normal  ScalingReplicaSet  3m43s                deployment-controller  Scaled up replica set my-nginx-deployment-7484748b57 to 1
  Normal  ScalingReplicaSet  3m42s                deployment-controller  Scaled down replica set my-nginx-deployment-556b57945d to 2
  Normal  ScalingReplicaSet  3m42s                deployment-controller  Scaled up replica set my-nginx-deployment-7484748b57 to 2
  Normal  ScalingReplicaSet  3m40s (x2 over 19m)  deployment-controller  Scaled up replica set my-nginx-deployment-7484748b57 to 3
  Normal  ScalingReplicaSet  3m40s                deployment-controller  Scaled down replica set my-nginx-deployment-556b57945d to 1
  Normal  ScalingReplicaSet  3m37s                deployment-controller  Scaled down replica set my-nginx-deployment-556b57945d to 0
```

## 서비스(Service)

쿠버네티스 클러스터 안에서 pod의 집합에 대한 경로나 서비스 디스커버리를 제공하는 리소스

### 타입

서비스 타입에 따라 포드에 접근하는 방법이 다르다.

- ClusterIP 타입 : 쿠버네티스 내부에서만 pod에 접근할 때 사용. 외부로 포드를 노출하지 않기 때문에 쿠버네티스 클러스터 내부에서만 사용되는 포드에 적합
- NodePort 타입 : 포드에 접근할 수 있는 포트를 클러스터의 모드 노드에 동일하게 개방 외부에서 포드에 접근할 수 있는 서비스 타입, 접근할 수 있는 포트는 랜덤으로 정해지지만, 특정 포트로 접근하도록 설정할 수 있음
- LoadBalancer 타입 : 클라우드 플랫폼에서 제공하는 로드 밸러서를 동적으로 프로비저닝해서 포드에 연결, 외부에서 포드에 접근할 수 있는 서비스 타입, 일반적으로 AWS, GCP, … 등과 같은 클라우드 플랫폼 환경에서 사용

### 디플로이먼트 생성

1. 매니페스트 파일 정의(yml)

```bash
vagrant@ubuntu:~/kub01$ vi deployment-hostname.yml
`YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hostname-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webserver
  template:
    metadata:
      name: my-webserver
      label:
        app: webserver
    spec:
      containers:
      - name: my-webserver
        image: alicek106/rr-test:echo-hostname
        ports:
        - containerPort: 80
`

# alicek106/rr-test:echo-hostname 이미지로 컨테이너를 실행
# host name을 보여주는 web이 뜬다.
vagrant@ubuntu:~$ docker run -itd --name test -p 80:80  alicek106/rr-test:echo-hostname
346c0be44baebdd8be42a281fc4bd7fa56b157ffaf6ee62c6726857825df71e0

vagrant@ubuntu:~$ docker container ls
CONTAINER ID        IMAGE                               COMMAND                   CREATED             STATUS              PORTS                                                                                                      NAMES
346c0be44bae        alicek106/rr-test:echo-hostname     "/bin/sh -c /entrypo…"   6 seconds ago       Up 4 seconds        0.0.0.0:80->80/tcp                                                                                         test

vagrant@ubuntu:~$ docker container exec -it test /bin/bash
root@346c0be44bae:/# exit

vagrant@ubuntu:~$ curl <http://localhost:80>

`html
<!DOCTYPE html>
<meta charset="utf-8" />
<link rel="stylesheet" type="text/css" href="./css/layout.css" />
<link rel="stylesheet" href="<https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css>">

<div class="form-layout">
        <blockquote>
        <p>Hello,  346c0be44bae</p>     </blockquote>		⇐ 컨테이너 ID = 호스트 이름
</div>
`
```

1. 디플로이먼트 생성

```
kubectl apply -f deployment-hostname.yml
```

1. 디플로이먼트를 통해서 생성된 pod 조회

```
kubectl get pods
```

1. -o wide 옵션을 이용해서 각 pod 에 할당된 IP 확인

`kubectl get pods -o wide` o, --output='' : 출력 포맷을 지정

1. 임시 pod 를 생성하여 hostname-deployment 디플로이먼트로 생성된 파드로 HTTP 요청

## ClusterIP 타입의 서비스 _ 쿠버네티스 내부에서만 pod 접근 가능

1. 매니페스트 파일 작성

```bash
vagrant@ubuntu:~/kub01$ vi hostname-svc-clusterip.yml
`YAML
apiVersion: v1
kind: Service
metadata:
  name: hostname-svc-clusterip
spec:
  ports:
    - name: web-port
      port: 8080   
      targetPort: 80
  selector:
    app: webserver
  type: ClusterIP
`
```

`port` : 서비스의 IP에 접근할 때 사용할 포트

`targetPort` : selector 항목에서 정의한 라벨에 의해 접근 대상이 된 pod 내부에서 사용하고 있는 포트

`selector` :  어떤 라벨을 가지는 pod에 접근할 수 있게 만들지 결정

`type` : 서비스 타입

1. 서비스 생성 및 확인

```bash
vagrant@ubuntu:~/kub01$ kubectl apply -f hostname-svc-clusterip.yml
service/hostname-svc-clusterip created

vagrant@ubuntu:~/kub01$ kubectl get services
NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
hostname-svc-clusterip   ClusterIP   10.110.56.159   <none>        8080/TCP   8s
kubernetes               ClusterIP   10.96.0.1       <none>        443/TCP    3d
# NAME : kubernetes 쿠버네티스 API에 접근하기 위한 서비스
```

1. 임시 pod 생성해서 요청 전송

```bash
kubectl run -it --rm debug --image=alicek106/ubuntu:curl --restart=Never -- bash

vagrant@ubuntu:~/kub01$ kubectl run -it --rm debug --image=alicek106/ubuntu:curl --restart=Never -- bash
If you don't see a command prompt, try pressing enter.

root@debug:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
82: eth0@if83: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:12:00:06 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.6/16 brd 172.18.255.255 scope global eth0
       valid_lft forever preferred_lft forever

root@debug:/# curl <http://10.110.56.159:8080> --silent | grep Hello
        <p>Hello,  hostname-deployment-7dfd748479-pp8x4</p>     </blockquote>
root@debug:/# curl <http://10.110.56.159:8080> --silent | grep Hello
        <p>Hello,  hostname-deployment-7dfd748479-pp8x4</p>     </blockquote>
```

### 서비스 확인

`kubectl get services` : 서비스 목록 보여줌

서비스의 IP와 PORT를 통해 pod에 접근, 서비스와 연결된 pod에 로드밸런싱 수행

### 서비스 삭제

```bash
kubectl delete service hostname-svc-clusterip
```

## NodePort 타입의 서비스_서비스를 이용해 pod외부에 노출

모든 노드의 특정 포트를 개방해서 서비스에 접근하는 방식

1. 매니패스트 파일

```bash
vagrant@ubuntu:~/kub01$ vi hostname-svc-nodeport.yml

'YAML
apiVersion: v1
kind: Service
metadata:
  name: hostname-svc-nodeport
spec:
  ports:
    - name: web-port
      port: 8080
      targetPort: 80
  selector:
    app: webserver
  type: NodePort
'
```

1. 서비스 생성 및 확인

```bash
vagrant@ubuntu:~/kub01$ kubectl apply -f hostname-svc-nodeport.yml
service/hostname-svc-nodeport created

vagrant@ubuntu:~/kub01$ kubectl get services
NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
hostname-svc-nodeport   NodePort    10.111.29.91   <none>        8080:30309/TCP   7s
kubernetes              ClusterIP   10.96.0.1      <none>        443/TCP          3d1h

vagrant@ubuntu:~/kub01$ kubectl get nodes -o wide
NAME       STATUS   ROLES    AGE    VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION       CONTAINER-RUNTIME
minikube   Ready    master   3d1h   v1.19.0   172.17.0.2    <none>        Ubuntu 20.04 LTS   4.15.0-117-generic   docker://19.3.8
```

1. 노드(172.17.0.2)에서 서비스 포토(30309)로 접근

```bash
vagrant@ubuntu:~/kub01$ curl <http://172.17.0.2:30309> -s | grep Hello
        <p>Hello,  hostname-deployment-7dfd748479-rtgzv</p>     </blockquote>
vagrant@ubuntu:~/kub01$ curl <http://172.17.0.2:30309> -s | grep Hello
        <p>Hello,  hostname-deployment-7dfd748479-pp8x4</p>     </blockquote>
```

1. ClusterIP 타입의 서비스와 같이 내부 IP와 서비스 이름으로 접근도 가능

```bash
vagrant@ubuntu:~/kub01$ kubectl run -it --rm debug --image=alicek106/ubuntu:curl --restart=Never -- bash
If you don't see a command prompt, try pressing enter.
root@debug:/#
root@debug:/# curl <http://10.111.29.91:8080> -s | grep Hello
        <p>Hello,  hostname-deployment-7dfd748479-pp8x4</p>     </blockquote>
root@debug:/# curl <http://10.111.29.91:8080> -s | grep Hello
        <p>Hello,  hostname-deployment-7dfd748479-7zdv7</p>     </blockquote>
root@debug:/# exit
exit
pod "debug" deleted
```

## 네임스페이스(namespace)

리소스(pod, replicaset, deployment, service, node, …)를 논리적으로 구분하는 기준

YAML 파일의 메타데이터 아래 namespace 속성으로 지정

### 네임스페이스 조회

```bash
vagrant@ubuntu:~/kub01$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   3d1h
kube-node-lease   Active   3d1h
kube-public       Active   3d1h
kube-system       Active   3d1h
```

기본적으로 kube-* 네임스페이스를 가진다.

default : 쿠버네티스를 설치하면 자동으로 사용하도록 설정되는 네임스페이스

### 특정 네임스페이스에 생성된 pod 확인

```bash
vagrant@ubuntu:~/kub01$ kubectl get pods --namespace default
NAME                                   READY   STATUS    RESTARTS   AGE
hostname-deployment-7dfd748479-7zdv7   1/1     Running   0          106m
hostname-deployment-7dfd748479-pp8x4   1/1     Running   0          106m
hostname-deployment-7dfd748479-rtgzv   1/1     Running   0          106m

vagrant@ubuntu:~/kub01$ kubectl get pods --namespace kube-system
NAME                               READY   STATUS    RESTARTS   AGE
coredns-f9fd979d6-lsnmk            1/1     Running   2          3d1h
etcd-minikube                      1/1     Running   1          3d1h
kube-apiserver-minikube            1/1     Running   1          3d1h
kube-controller-manager-minikube   1/1     Running   2          3d1h
kube-proxy-sbghs                   1/1     Running   2          3d1h
kube-scheduler-minikube            1/1     Running   2          3d1h
storage-provisioner                1/1     Running   4          3d1h
```

### 특정 네임스페이스에 생성된 service 확인

```bash
vagrant@ubuntu:~/kub01$ kubectl get service --namespace kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   3d1h
```

## 네임스페이스 사용

1. YAML 파일 생성

```bash
vagrant@ubuntu:~/kub01$ vi production-namespace.yml
`YAML
apiVersion: v1
kind: Namespace
metadata:
  name: production
`
```

1. 다른방법. 명령어로 생성

```bash
kubectl create namespace mynamespace
```

1.1 네임스페이스 조회

```bash
vagrant@ubuntu:~/kub01$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   3d1h
kube-node-lease   Active   3d1h
kube-public       Active   3d1h
kube-system       Active   3d1h
mynamespace       Active   8s
production        Active   2m4s
```

### 특정 네임스페이스에 리소스를 생성하는 방법

```bash
vagrant@ubuntu:~/kub01$ cp deployment-hostname.yml hostname-deploy-svc-ns.yml
vagrant@ubuntu:~/kub01$ vi hostname-deploy-svc-ns.yml

`YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hostname-deployment-ns
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webserver
  template:
    metadata:
      name: my-webserver
      labels:
        app: webserver
    spec:
      containers:
      - name: my-webserver
        image: alicek106/rr-test:echo-hostname
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: hostname-svc-clusterip-ns
  namespace: production
spec:
  ports:
    - name: web-port
      port: 8080
      targetPort: 80
  selector:
    app: webserver
  type: ClusterIP
`
vagrant@ubuntu:~/kub01$ kubectl apply -f hostname-deploy-svc-ns.yml
deployment.apps/hostname-deployment-ns created
service/hostname-svc-clusterip-ns created

vagrant@ubuntu:~/kub01$ kubectl get pods,services --namespace production
NAME                                          READY   STATUS    RESTARTS   AGE
pod/hostname-deployment-ns-7dfd748479-4mg2b   1/1     Running   0          50s
pod/hostname-deployment-ns-7dfd748479-jgzg6   1/1     Running   0          50s
pod/hostname-deployment-ns-7dfd748479-rg8lw   1/1     Running   0          50s

NAME                                TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/hostname-svc-clusterip-ns   ClusterIP   10.97.13.64   <none>        8080/TCP   50s
```

서로 다른 네임스페이스에서 이름으로 접근하려면 .NAMESPACE.svc 붙여 명시해줘야한다.

```bash
vagrant@ubuntu:~/kub01$ kubectl run -it --rm debug --image=alicek106/ubuntu:curl --restart=Never -- bash
If you don't see a command prompt, try pressing enter.
root@debug:/#
root@debug:/# curl <http://hostname-svc-clusterip-ns:8080>
curl: (6) Could not resolve host: hostname-svc-clusterip-ns
# default 네임스페이스에서는 접근 못함

root@debug:/# curl <http://hostname-svc-clusterip-ns.production.svc:8080>
<!DOCTYPE html>
<meta charset="utf-8" />
# <이하생략>
```

### 네임스페이스 삭제

```bash
vagrant@ubuntu:~/kub01$ kubectl delete namespace production
namespace "production" deleted
```

### 네임스페이스에 속하는 오브젝트 종류

```bash
vagrant@ubuntu:~/kub01$ kubectl api-resources --namespaced=true
```

### 네임스페이스에 속하지 않는 오브젝트 종류

```bash
vagrant@ubuntu:~/kub01$ kubectl api-resources --namespaced=false
```

# 컨피그맵(Configmap), 시크릿(Secret)

설정값을 pod로 전달하는 방법

# 컨피그맵

일반적인 설정 정보(값)을 저장할 수 있는 쿠버네티스 오브젝트로 네임스페이스 별 존재한다.

### **컨피그맵 설정**

```bash
vagrant@ubuntu:~$ kubectl create configmap log-level-configmap --from-literal LOG_LEVEL=DEBUG
configmap/log-level-configmap created

vagrant@ubuntu:~$ kubectl create configmap start-k8s --from-literal k8s=kubernetes --from-literal container=docker
configmap/start-k8s created
```

### 컨피그맵 확인

```bash
vagrant@ubuntu:~$ kubectl get configmap
NAME                  DATA   AGE
log-level-configmap   1      3m41s
start-k8s             2      2m27s

vagrant@ubuntu:~$ kubectl describe configmap log-level-configmap
Name:         log-level-configmap
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
LOG_LEVEL:
----
DEBUG
Events:  <none>
```

### 컨피그맵을 YAML 로 출력

```bash
vagrant@ubuntu:~$ kubectl get configmap log-level-configmap -o yaml
apiVersion: v1
data:
  LOG_LEVEL: DEBUG
kind: ConfigMap
metadata:
  creationTimestamp: "2020-09-22T01:13:06Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:LOG_LEVEL: {}
    manager: kubectl-create
    operation: Update
    time: "2020-09-22T01:13:06Z"
  name: log-level-configmap
  namespace: default
  resourceVersion: "68473"
  selfLink: /api/v1/namespaces/default/configmaps/log-level-configmap
  uid: 6ddcde79-4ff9-4403-87ac-00335744bb7a
```

## pod 에서 컨피그맵을 사용하기. 1 - 컨피그맵의 값을 환경변수로 사용

### 1. pod 정의 - YAML 파일 생성

```bash
vagrant@ubuntu:~$ vi all-env-from-configmap.yml

`YAML
apiVersion: v1
kind: Pod
metadata:
  name: container-env-example
spec:
  containers:
    - name: my-container
      image: busybox
      args: ['tail', '-f', '/dev/null']
      envFrom:
        - configMapRef:
            name: log-level-configmap
        - configMapRef:
            name: start-k8s
`
# envFrom : 컨피그맵에 정의된 모든 키-값 쌍을 가져와서 환경변수로 설정
# configMapRef - name : my-container의 환경변수로 설정 위에선 두개 생성
```

### pod 생성

```bash
vagrant@ubuntu:~$ kubectl apply -f all-env-from-configmap.yml
pod/container-env-example created

vagrant@ubuntu:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
container-env-example                  1/1     Running   0          30s
hostname-deployment-7dfd748479-7zdv7   1/1     Running   0          19h
hostname-deployment-7dfd748479-pp8x4   1/1     Running   0          19h
hostname-deployment-7dfd748479-rtgzv   1/1     Running   0          19h
```

### 컨테이너의 환경변수를 확인

k8s=kubernetes , LOG_LEVEL=DEBUG , container=docker 을 확인할 수 있다.

```bash
vagrant@ubuntu:~$ kubectl exec container-env-example -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=container-env-example
k8s=kubernetes
LOG_LEVEL=DEBUG
container=docker
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_PORT_443_TCP_PROTO=tcp
HOSTNAME_SVC_NODEPORT_PORT=tcp://10.111.29.91:8080
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
HOSTNAME_SVC_NODEPORT_SERVICE_HOST=10.111.29.91
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
HOSTNAME_SVC_NODEPORT_SERVICE_PORT_WEB_PORT=8080
HOSTNAME_SVC_NODEPORT_PORT_8080_TCP_PROTO=tcp
HOSTNAME_SVC_NODEPORT_PORT_8080_TCP_PORT=8080
HOSTNAME_SVC_NODEPORT_PORT_8080_TCP_ADDR=10.111.29.91
KUBERNETES_PORT_443_TCP_PORT=443
HOSTNAME_SVC_NODEPORT_SERVICE_PORT=8080
HOSTNAME_SVC_NODEPORT_PORT_8080_TCP=tcp://10.111.29.91:8080
HOME=/root
```

### 컨피그맵에 존재하는 키-값 쌍 중에서 원하는 데이터만 환경변수로 설정

```bash
vagrant@ubuntu:~$ cp all-env-from-configmap.yml selective-env-from-configmap.yml
vagrant@ubuntu:~$ vi selective-env-from-configmap.yml

`YAML
apiVersion: v1
kind: Pod
metadata:
  name: container-env-example
spec:
  containers:
    - name: my-container
      image: busybox
      args: ['tail', '-f', '/dev/null']
      env:
        - name: ENV_KEYNAME_1               ⇐ 새롭게 설정될 환경변수 이름 -------------+ ENV_KEYNAME_1=LOG_LEVEL값
          valueFrom:                                                                    | 
            configMapKeyRef:                                                               | 
              name: log-level-configmap     ⇐ 참조할 컨피그맵 이름                     |
              key: LOG_LEVEL                ⇐ 참조할 컨피그맵에서 가져올 데이터의 키 --+
        - name: ENV_KEYNAME_2
          valueFrom:
            configMapKeyRef:
              name: start-k8s
              key: k8s
`
```

### pod 중복시 삭제

```bash
vagrant@ubuntu:~$ kubectl delete -f all-env-from-configmap.yml
pod "container-env-example" deleted
```

### pod 생성

```bash
vagrant@ubuntu:~$ kubectl apply -f selective-env-from-configmap.yml
pod/container-env-example created
```

### pod 생성 확인

```bash
vagrant@ubuntu:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
container-env-example                  1/1     Running   0          28s
hostname-deployment-7dfd748479-7zdv7   1/1     Running   0          19h
hostname-deployment-7dfd748479-pp8x4   1/1     Running   0          19h
hostname-deployment-7dfd748479-rtgzv   1/1     Running   0          19h
```

### 컨테이너 내부의 환경변수를 확인

ENV_KEYNAME_1=DEBUG 와 ENV_KEYNAME_2=kubernetes 를 확인 할 수 있다.

```bash
vagrant@ubuntu:~$ kubectl exec container-env-example -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=container-env-example
ENV_KEYNAME_1=DEBUG
ENV_KEYNAME_2=kubernetes
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
HOSTNAME_SVC_NODEPORT_SERVICE_PORT=8080
HOSTNAME_SVC_NODEPORT_SERVICE_PORT_WEB_PORT=8080
HOSTNAME_SVC_NODEPORT_PORT_8080_TCP=tcp://10.111.29.91:8080
HOSTNAME_SVC_NODEPORT_PORT_8080_TCP_PORT=8080
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
HOSTNAME_SVC_NODEPORT_SERVICE_HOST=10.111.29.91
HOSTNAME_SVC_NODEPORT_PORT=tcp://10.111.29.91:8080
HOSTNAME_SVC_NODEPORT_PORT_8080_TCP_PROTO=tcp
HOSTNAME_SVC_NODEPORT_PORT_8080_TCP_ADDR=10.111.29.91
KUBERNETES_SERVICE_PORT_HTTPS=443
HOME=/root
```

## pod 에서 컨피그맵을 사용하기. 2 - 컨피그맵의 값을 pod 내부 파일로 마운트해서 사용

### 컨피그맵의 모든 키-쌍 데이터를 포드에 마운트

```bash
vagrant@ubuntu:~$ vi volume-mount-configmap.yml
`YAML
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod
spec:
  containers:
    - name: my-container
      image: busybox
      args: ["tail", "-f", "/dev/null"]
      volumeMounts:                  ⇐ #1에서 정의한 볼륨을 컨테이너 내부의 어떤 디렉터리에 마운트할 것인지 명시
        - name: configmap-volume     ⇐ 컨피그맵 볼룸의 이름 (#1에서 정의한 이름)
          mountPath: /etc/config     ⇐ 컨피그맵 파일이 위치할 경로

  volumes:                           ⇐ #1 사용할 볼륨 목록 
    - name: configmap-volume           
      configMap:
        name: start-k8s              ⇐ 컨피그맵 이름
`
```

### pod 생성

```bash
vagrant@ubuntu:~$ kubectl apply -f volume-mount-configmap.yml
pod/configmap-volume-pod created
```

### pod 생성 확인

```bash
vagrant@ubuntu:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
configmap-volume-pod                   1/1     Running   0          27s
container-env-example                  1/1     Running   0          53m
hostname-deployment-7dfd748479-7zdv7   1/1     Running   0          20h
hostname-deployment-7dfd748479-pp8x4   1/1     Running   0          20h
hostname-deployment-7dfd748479-rtgzv   1/1     Running   0          20h
```

### pod의 /etc/config 확인

```bash
vagrant@ubuntu:~$ kubectl exec configmap-volume-pod -- ls -l /etc/config
total 0
lrwxrwxrwx    1 root     root            16 Sep 22 02:33 container -> ..data/container
lrwxrwxrwx    1 root     root            10 Sep 22 02:33 k8s -> ..data/k8s
```

### file 내용 확인

```bash
vagrant@ubuntu:~$ kubectl exec configmap-volume-pod -- cat /etc/config/container
docker                        

vagrant@ubuntu:~$ kubectl exec configmap-volume-pod -- cat /etc/config/k8s
kubernetes
```

컨피그맵에 키는 파일명으로, 값은 파일의 내용으로 변경되어서 전달

## 원하는 키-값 쌍의 데이터만 선택해서 pod로 마운트

```bash
vagrant@ubuntu:~$ cp volume-mount-configmap.yml selective-volume-configmap.yml
vagrant@ubuntu:~$ vi selective-volume-configmap.yml

`YAML
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod
spec:
  containers:
    - name: my-container
      image: busybox
      args: ["tail", "-f", "/dev/null"]
      volumeMounts:
        - name: configmap-volume
          mountPath: /etc/config
  volumes:
    - name: configmap-volume
      configMap:
        name: start-k8s
        items:
        - key: k8s                ⇐ 가져올 키를 명시
          path: k8s_fullname      ⇐ 키 값을 저장할 파일명
`
```

### 앞에서 생성한 pod를 삭제

```bash
vagrant@ubuntu:~$ kubectl delete -f volume-mount-configmap.yml
pod "configmap-volume-pod" deleted
```

### 파드 생성 및 확인

```bash
vagrant@ubuntu:~$ kubectl apply -f selective-volume-configmap.yml
pod/configmap-volume-pod created
vagrant@ubuntu:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
configmap-volume-pod                   1/1     Running   0          8s
container-env-example                  1/1     Running   0          63m
hostname-deployment-7dfd748479-7zdv7   1/1     Running   0          20h
hostname-deployment-7dfd748479-pp8x4   1/1     Running   0          20h
hostname-deployment-7dfd748479-rtgzv   1/1     Running   0          20h
```

### 파드(컨테이너) 내부의 파일 생성 여부 확인

```bash
vagrant@ubuntu:~$ kubectl exec configmap-volume-pod -- ls /etc/config
k8s_fullname

vagrant@ubuntu:~$ kubectl exec configmap-volume-pod -- cat /etc/config/k8s_fullname
kubernetes
```

## file 로 컨피그맵 생성

### 테스트 파일 생성

```bash
vagrant@ubuntu:~$ echo Hello, world! >> index.html
vagrant@ubuntu:~$ cat index.html
Hello, world!
```

### 테스트 파일(index.html)을 이용해서 index-file이라는 컨피그맵을 생성

```bash
vagrant@ubuntu:~$ kubectl create configmap index-file --from-file ./index.html
configmap/index-file created
```

### 생성한 index-file 컨피그맵을 확인

```bash
vagrant@ubuntu:~$ kubectl describe configmap index-file
Name:         index-file
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
index.html:		⇐ 파일명이 키(key)로 사용
----
Hello, world!		⇐ 파일의 내용이 값(value)로 사용

Events:  <none>
```

### 키이름을 직접 지정해서 컨피그맵을 생성

```bash
vagrant@ubuntu:~$ kubectl create configmap index-file-customkey --from-file myindex=./index.html
configmap/index-file-customkey created

vagrant@ubuntu:~$ kubectl describe configmap index-file-customkey
Name:         index-file-customkey
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
myindex:		⇐ 컨피그맵 생성 시 지정한 키 이름을 사용
----
Hello, world!

Events:  <none>
```

## 여러 개의 키-값 형태의 내용으로 구성된 설정 파일을 한번에 컨피그맵으로 설정

### 키-값 형태의 내용으로 구성된 설정 파일을 생성

```bash
vagrant@ubuntu:~$ vi ./multiple-keyvalue.env

mykey1=myvalue1
mykey2=myvalue2
mykey3=myvalue3
```

### 설정 파일에 정의된 키-값 형태를 컨피그맵의 키-값 항목으로 일괄 전환

```bash
kubectl create configmap abcd --from-literal mykey1=myvalue1 --from-literal mykey2=myvalue2 --from-literal mykey3=myvalue3 … 형식의 명령어를 파일을 이용해서 구현

vagrant@ubuntu:~$ kubectl create configmap from-envfile --from-env-file ./multiple-keyvalue.env
configmap/from-envfile created
vagrant@ubuntu:~$ kubectl describe configmap from-envfile
Name:         from-envfile
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
mykey1:
----
myvalue1
mykey2:
----
myvalue2
mykey3:
----
myvalue3
Events:  <none>
```

## YAML 파일로 컨피그맵을 정의

### 컨피그맵을 실제로 생성하지 않고 YAML 형식으로 출력

```bash
vagrant@ubuntu:~$ kubectl create configmap my-configmap --from-literal mykey=myvalue --dry-run -o yaml
W0922 04:34:05.917495  358937 helpers.go:553] --dry-run is deprecated and can be replaced with --dry-run=client.
apiVersion: v1
data:
  mykey: myvalue
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: my-configmap

vagrant@ubuntu:~$ kubectl get configmap
NAME                   DATA   AGE		⇒ my-configmap 이름의 컨피그맵이 존재하지 않음
from-envfile           3      4m2s		   → --dry-run 옵션: 실제로 컨피그맵 오브젝트를 생성하지는 않음
index-file             1      16m			   
index-file-customkey   1      12m
log-level-configmap    1      3h21m
start-k8s              2      3h20m
```

### YAML 형식의 출력을 YAML 파일로 저장

```bash
vagrant@ubuntu:~$ kubectl create configmap my-configmap --from-literal mykey=myvalue --dry-run -o yaml > my-config.yml
W0922 04:42:17.088587  360577 helpers.go:553] --dry-run is deprecated and can be replaced with --dry-run=client.
vagrant@ubuntu:~$

vagrant@ubuntu:~$ cat my-config.yml
apiVersion: v1
data:
  mykey: myvalue
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: my-configmap
```

### YAML 파일로 컨피그맵을 생성

```bash
vagrant@ubuntu:~$ kubectl apply -f my-config.yml
configmap/my-configmap created

vagrant@ubuntu:~$ kubectl get configmaps
NAME                   DATA   AGE
from-envfile           3      13m
index-file             1      26m
index-file-customkey   1      22m
log-level-configmap    1      3h31m
my-configmap           1      9s
start-k8s              2      3h30m
```

# 시크릿(Secret)

민감한 정보를 저장하기 위한 용도, 네임스페이스에 종속

## 시크릿 생성 방법

### password=1q2w3e4r 라는 키-값을 저장하는 my-password 이름의 시크릿을 생성

```bash
vagrant@ubuntu:~$ kubectl create secret generic my-password --from-literal password=1q2w3e4r
secret/my-password created

vagrant@ubuntu:~$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-sh8hv   kubernetes.io/service-account-token   3      3d22h ⇐ ServiceAccount에 의해 네임스페이스별로 자동으로 생성된 시크릿
my-password           Opaque                                1      9s
```

### 파일로부터 시크릿을 생성

```bash
vagrant@ubuntu:~$ echo mypassword > pw1 && echo yourpassword > pw2
vagrant@ubuntu:~$ cat pw1
mypassword
vagrant@ubuntu:~$ cat pw2
yourpassword

vagrant@ubuntu:~$ kubectl create secret generic out-password --from-file pw1 --from-file pw2
secret/out-password created
vagrant@ubuntu:~$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-sh8hv   kubernetes.io/service-account-token   3      3d22h
my-password           Opaque                                1      5m29s
out-password          Opaque                                2      51s
```

### 시크릿 내용을 확인

```bash
vagrant@ubuntu:~$ kubectl describe secret my-password
Name:         my-password
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  8 bytes		⇐ password 키에 해당하는 값을 확인할 수 없음 (값의 크기(길이)만 출력)

vagrant@ubuntu:~$ kubectl get secret my-password -o yaml
apiVersion: v1
data:
  password: MXEydzNlNHI=	⇐ BASE64로 인코딩
kind: Secret
metadata:
  creationTimestamp: "2020-09-22T04:49:44Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:password: {}
      f:type: {}
    manager: kubectl-create
    operation: Update
    time: "2020-09-22T04:49:44Z"
  name: my-password
  namespace: default
  resourceVersion: "81153"
  selfLink: /api/v1/namespaces/default/secrets/my-password
  uid: e597d8d2-479e-464f-934d-5d2ae7f232c8
type: Opaque

vagrant@ubuntu:~$ echo MXEydzNlNHI= | base64 -d
1q2w3e4r
```

## 시크릿에 저장된 키-값 쌍을 포드로 가져오기

### 시크릿에 저장된 모든 키-값 쌍을 포드의 환경변수로 가져오기

```bash
vagrant@ubuntu:~$ vi env-from-secret.yml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-example
spec:
  containers:
    - name: my-container
      image: busybox
      args: ["tail", "-f", "/dev/null"]
      envFrom:
        - secretRef:
            name: my-password

vagrant@ubuntu:~$ kubectl apply -f env-from-secret.yml
pod/secret-env-example created

vagrant@ubuntu:~$ kubectl exec secret-env-example -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=secret-env-example
password=1q2w3e4r
HOSTNAME_SVC_NODEPORT_SERVICE_PORT_WEB_PORT=8080
HOSTNAME_SVC_NODEPORT_PORT=tcp://10.111.29.91:8080
HOSTNAME_SVC_NODEPORT_PORT_8080_TCP_PORT=8080
KUBERNETES_SERVICE_PORT=443
HOSTNAME_SVC_NODEPORT_SERVICE_PORT=8080
KUBERNETES_PORT_443_TCP_PROTO=tcp
HOSTNAME_SVC_NODEPORT_PORT_8080_TCP_ADDR=10.111.29.91
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
HOSTNAME_SVC_NODEPORT_SERVICE_HOST=10.111.29.91
HOSTNAME_SVC_NODEPORT_PORT_8080_TCP=tcp://10.111.29.91:8080
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_SERVICE_HOST=10.96.0.1
HOSTNAME_SVC_NODEPORT_PORT_8080_TCP_PROTO=tcp
HOME=/root
```

### 시크릿에 저장된 특정 키-값 쌍을 포드의 환경변수로 가져오기