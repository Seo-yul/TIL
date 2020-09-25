# Jenkins

빌드 파이프라인 도구로 명령어 조작을 보다 간단하고 안전하며 확실하게 처리한다.

구축 및 테스트 결과와 이력을 축적하고, 팀에서 확인할 수 있다.

그 이유로는 작업을 프로젝트 단위로 모아서 쉽게 실행, 수작업을 할 필요가 없어 실수가 없음, 프로젝트 실행과 결과에 대한 이력의 목록화 가 있다.

## install

CentOS Jenkins 공식 문서 https://pkg.jenkins.io/redhat-stable/

1. 먼저 자바를 설치

```
yum -y install java-1.8.0-openjdk java-1.8.0-openjdk-devel
```

1. Jenkins 설치

```
yum -y install [<http://mirrors.jenkins-ci.org/redhat-stable/jenkins-2.235.5-1.1.noarch.rpm>](<http://mirrors.jenkins-ci.org/redhat-stable/jenkins-2.235.5-1.1.noarch.rpm>)
```

1. Jenkins 실행

```
systemctl start jenkins.service
```

1. Jenkins 접속

호스트 PC 에서 브라우저를 이용해 접속한다.

접속에 필요한 패스워드는 `/var/lib/jenkins/secrets/initialAdminPassword`

1. Jenkins 최초 설정

- `install suggested plugins`
- 설치 진행
- 관리자 계정 설정
- 재시작

1. 웹 브라우저에서 프로젝트를 만들어 사용한다.

예제

```bash
1. item name 을 만든다
2. Freestyle project를 사용한다.
3. ok

완료 되면
Build 탭 에서 Add build step 드롭 다운 메뉴에서 Excute shell을 사용한다.
Command 에 명령 넣고 저장

빌드를 사용하기
프로젝트 탭에서 Build now 클릭.

빌드 히스토리 보기
왼쪽 메뉴바에 Build History 가 있다.
Console Output 에 내용이 있음.
```

## pipe-line 사용

```groovy
node {
    stage 'ansible'
    build 'exec-ansible'
    stage 'serverspec'
    build 'exec-serverspec'
}
// stage 라는것은 단계, 혹은 라벨링 지정하는 것
// build 의 내용을 실행해라
// groovy  문법 사용
```