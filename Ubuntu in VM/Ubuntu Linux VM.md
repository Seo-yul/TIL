# Ubuntu Linux VM

## NAT(Network Address Translation)

통신이 가능하게 해주는것 범위까지 소프트웨어적으로 설정해줌

NAT 소프트웨어는 게스트간 통신이 불가능하다.

NAT Network는 게스트간 통신이 가능하다.

## Internal Network(내부 네트워크)

게스트 끼리만 통신을 한다. 외부로 나가지 않아.

## Host-Only Adapter(호스트 전용)

## Bridge Adapter(브릿지)

호스트와 게스트가 동일 네트워크 망을 사용하게 한다. 사설망의 예) 192.168.0.x

# vm 연결 관련

ssh설정 : vi /etc/ssh/sshd_config

local port 상태 확인 : nmap [localhost](http://localhost) 또는 netstat -tnlp

윈도우 cmd 에서 ssh 붙는법 : 계정명@ip -p port

시스템 종료 명령어 : poweroff, shutdown -P now, halt -p, init 0

- shutdown 명령어 옵션 중 now 부분에 시간을 지정하면 설정한 시간에 시스템 종료

```python
shutdown -P +10   # 10분 후에 종료 (P: poweroff)
shutdown -r 22:00 # 오후 10시 재부팅(r: reboot)
shutdown -c       # 예약된 shutdown 취소(c: cancel)
shutdown -k +15   # 현재 접속한 사용자에게 15분 후 종료된다는 메시지를 보냄
```

시스템 재부팅 : reboot, shutdown -r now, init 6

로그아웃 : logout, exit

가상 콘솔: ctrl+alt+f1 ~ f7 ( 7번은 x window)

## 런레벨(runlevel)

런레벨 모드를 확인하려면 /lib/systemd/system 디렉토리의 runlevel?.target 조회

0 : Power Off 종료모드

1 : Rescue 시스템 복구 모드 (단일 사용자 모드)

2 : Multi-User

3 : Multi-User 텍스트 모드의 다중 사용자 모드

4 : Multi-User

5 : Graphical 그래픽 모드의 다중 사용자 모드

6 : Reboot 재부팅 모드

```bash
# 런레벨 목록 확인
ls -al /lib/systemd/system/runlevel?.target
# 런레벨 디폴트 확인
ls -al /lib/systemd/system/default.target
# 런레벨 변경
ln -sf /lib/systemd/system/multi-user.target /lib/systemd/system/default.target
# 런레벨 변경을 확인
ls -al /lib/systemd/system/default.target
# 리부트
reboot
```

# 프롬프트 구조

```bash
사용자를 구분 ⇒ $: 일반사용자, #: 루트사용자

ubuntu@server:~$ ls -al /lib/systemd/system/default.target 
------ ------ -  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 명령어 ⇒ 명령어 [서브 명령어] [파라미터]
  |      |    |
  |      |    +--- 현재 명령어를 입력하고 있는 위치 (디렉터리)
  |      |         ~ ⇒ 로그인한 계정(사용자)의 홈 디렉터리 ⇒ root → /root, 일반사용자 → /home/계정명
  |      +--------- 호스트 명
  +---------------- 로그인한 계정
```

## CD-ROM 마운트 / 언마운트

```bash
# 디렉토리 생성
ls /mnt/cdrom
mkdir -p /mnt/cdrom            -p : 디렉토리를 순차적으로 생성
ls /mnt/cdrom 

# 마운트
mount /dev/cdrom /mnt/cdrom        # /dev/cdrom 디바이스를 /mnt/cdrom 디렉터리에 연결
cd /mnt/cdrom

# 언 마운트
umount /mnt/cdrom
umount: /mnt/cdrom: target is busy.	 # 마운트 디렉터리(/mnt/cdrom)에서 umount 했기 때문
/mnt/cdrom                           # umount -l /mnt/cdrom
mount                                # 
```

## 가상 머신의 특정 디렉토리를 ISO 파일로 제작

**genisoimage 프로그램 설치 여부를 확인**

`dpkg --get-selections genisoimage`

no packages found matching genisoimage

**genisoimage 설치**

`apt install genisoimage`

**/bin 디렉터리 아래에 있는 파일과 디렉터리를 boot.iso 파일로 압축**

`genisoimage -r -J -o boot.iso /boot`

## ISO file mount

**마운트 디렉토리 생성**

`mkdir -p /mnt/iso`

**mount**

`mount -o loop boot.iso /mnt/iso`

mount: /mnt/iso: WARNING: device write-protected, mounted read-only.

**/bin 디렉토리와 /mnt/iso 디렉토리 비교**

`ls -l /mnt/iso`

`ls -l /boot`

## ISO file umount

`umount /mnt/iso`

# 기본 명령어

- cat : conCATenate 의 약자로, 파일 내용을 화면에 출력한다. 명령어 뒤에 여러 개의 파일명을 나열하면 파일을 연결하여 내용을 화면에 출력 cat a.txt b.txt
- head, tail : 텍스트 형식으로 작성된 파일의 앞 10행 또는 마지막 10행만 화면에  출력 옵션으로 라인 수 지정 head -3 a.txt , tail -3 a.txt
- more : 텍스트 형식으로 작성된 파일을 페이지 단위로 화면에 출력 Space bar 로 다음 페이지 B 로 뒤로 , Q 종료
- less : more 명령어와 용도가 비슷하지만 더 확장된 기능의 명령어 more명령어에서 방향키, 페이지 업다운 사용 가능
- file : 해당 파일이 어떤 종류의 파일인지 알려줌

# 네트워크 정보 확인

- 기본 ip 정보 : `ip addr`
- 현재 Server에 설정된 게이트웨이 정보 : `ip route`
- 현재 설정된 DNS 정보 : `systemd-resolve --status 장치명`

구글 DNS 서버 8.8.8.8

- 네트워크 매니저 에디터 : `nm-connection-editor`

아이피 : 네트워크 상에 호스트를 식별 할 수 있는 유일한 식별자

네트워크 주소 : 동일한 아이피 대역을 사용하는 호스트의 대표 주소.

게이트웨이 : 서로 다른 네트워크를 이어주는 역할로 항상 인터페이스가 두개가 있다. 

`netstat -rn` : 게이트웨이 확인

# 네트워크 관련 설정과 명령

`systemctl start/stop/restart/status networking`

네트워크의 설정을 변경한 후 변경 내용을 시스템에 적용하는 명령 네트워크 매니저 에디터를 실행하여 내용 변경한 경우 반드시 systemctl restart networking 명령을 실행해 적용한다.

`ifconfig 장치명` : 해당 장치의 ip 주소와 관련 정보를 출력하는 명령

`nslookup` : DNS 서버의 작동을 테스트하는 명령

`/etc/netplan/`:네트워크 기본 정보 설정 파일

`/etc/resolv.conf` : DNS 서버의 정보와 호스트 네임이 들어 있는 파일. 임시 파일로 네트워크 재시작시 초기화

`/etc/hosts` : 현재 컴퓨터의 호스트 이름과 FQDN 이 들어 있는 파일

### cli환경의 IP 설정

`/etc/netpaln/` 디렉토리의 yaml 파일에서 설정한다.

```bash
network:
    ethernets:
        enp0s3:
            dhcp4: false
            addresses: [10.0.2.200/24]
            gateway4: 10.0.2.1
            nameservers:
                addresses: [8.8.8.8]
    version: 2
```

> yaml 파일 주의할 점. 탭(tab)을 사용할 수 없다. 반드시 공백 사용. 계층 indent주의.

# 사용자와 그룹

`/etc/passwd :`유저 계정이 정의 되어있는 파일

'사용자 이름:비밀번호:사용자 ID:사용자 소속 그룹 ID:추가 정보:홈 디렉토리:기본 셸'

![Untitled.png](https://github.com/resourceSaga/TIL/blob/master/Ubuntu%20in%20VM/Ubuntu%20Linux%20VM/Untitled.png?raw=true)

- 사용자 이름 : ubuntu
- 비밀번호 : x 표시 → /etc/shadow 파일에 정의
- 사용자 ID: 1000, 그룹 ID: 1000
- 홈 디렉토리: /home/ubuntu
- 기본 셸: /bin/bash

root 사용자는 사용자 ID와 소속 그룹 ID가 모두 0

`/etc/group` : 유저 그룹이 정의 되어있는 파일

'그룹 이름:비밀번호:그룹 ID:보조 그룹 사용자'

![Untitled1.png](https://github.com/resourceSaga/TIL/blob/master/Ubuntu%20in%20VM/Ubuntu%20Linux%20VM/Untitled%201.png?raw=true)

## 사용자 추가하기 adduser

adduser 명령어를 실행하면 /etc/passwd, /etc/shadow, /etc/ group 에 새로운 행이 추가됨

별도로 그룹을 지정하지 않으면 사용자 이름과 같은 그룹이 자동으로 생성된다./

## 사용자 비밀번호 변경 passwd

사용자의 비밀번호를 변경

## 사용자의 속성 변경 usermod —option

사용자의 속성(기본 셸, 그룹 등)을 옵션을 줘서 변경한다.

## 사용자 삭제 userdel

사용자를 삭제한다 -r 옵션시 사용자 삭제 및 홈 디렉토리까지 삭제

## 비밀번호 변경 chage

사용자의 비밀번호를 주기적으로 변경하도록 설정 `chage option username`

```python
chage -l username    # 사용자에 설정된 내용 확인
chage -m 2 username  # 사용자에 설정한 비밀번호를 사용해야 하는 최소 일자
chage -M 30 username # 사용자에 설정한 비밀번호를 사용할 수 있는 최대 일자
chage -E 2020/12/31  # username 사용자에 설정한 비밀번호 만료일
chage -W 10 username # 만료 전의 경고 기간
```

## 사용자의 그룹 groups

사용자가 소속된 그룹을 보여준다. `groups username`

## 새로운 그룹 생성 groupadd

`groupadd —gid group_number group_name` 그룹을 생성하고 그룹ID를 지정

## 그룹 속성 변경 groupmod

`groupmod —new-name old_name new_name` 그룹의 속성을 변경한다

## 그룹 삭제 groupdel

`groupdel group_name` 그룹을 삭제한다. 그룹을 주 그룹으로 지정한

## 그룹 비밀번호 설정. 그룹 관리 gpasswd

```python
gpasswd group_name                 # 그룹 비밀번호 설정
gpasswd -A user_name group_name    # 사용자를 그룹 관리자로 지정
gpasswd -a user_name group_name    # 사용자를 그룹에 추가
gpasswd -d user_name group_name    # 사용자를 그룹에서 제거
```

# 소유권과 허가권

![Untitled2.png](https://github.com/resourceSaga/TIL/blob/master/Ubuntu%20in%20VM/Ubuntu%20Linux%20VM/Untitled%202.png?raw=true)

r : read

w : write

x : execute

![Untitled3.png](https://github.com/resourceSaga/TIL/blob/master/Ubuntu%20in%20VM/Ubuntu%20Linux%20VM/Untitled%203.png?raw=true)
디렉토리에 접근하기 위해서는 x 권한이 있어야 한다.

## 파일 허가권의 변경 chmod

root 사용자나 해당 파일의 소유자만 실행 가능 `chome 644 filen_name`

`chmod u+x file_name` 은 소유자에게 실행 권한을 준다. 라는 식으로도 사용 가능

## 파일의 소유권을 변경 chown

root 사용자나 해당 파일의 소유자만 실행 가능

 `chown user_name file_name` : user_name 에게 file_name 소유권을 준다.

`chown user_name.group_name file_name` : user_name 에게, group_name 그룹에게도 소유권 줌

`chgrp group_name file_name` : 소유 그룹만 group_name 에게 준다.

# 하드 링크와 심볼릭 링크

파일의 링크는 hard link와 symbolic link 로 구분된다 symbolic link는 soft link 라고도 불림

`ln original_file link_file` : 하드 링크를 생성

`ln -s original_file link_file` : 심볼릭 링크 생성

![Untitled4.png](https://github.com/resourceSaga/TIL/blob/master/Ubuntu%20in%20VM/Ubuntu%20Linux%20VM/Untitled%204.png?raw=true)

하드링크는 원본 파일을 이동해도 inode 를 똑같이 추적하여 링크가 끊기지 않지만 심볼릭 링크는 새로운 inode를 가지기 때문에 원본 파일 이동시 포인터가 끊긴다.

# 프로세스

`ps` : .현재 프로세스의 상태를 확인, 주로 `ps -ef | grep 프로세스명` 을 사용

`kill` : 프로세스를 강제 종료, `kill -9 프로세스번호` : 프로세스 강제 종료

`pstree` : 부모 프로세스와 자식 프로세스의 관계 트리를 보여줌

`jobs` : 백그라운드에서 작업중이거나 중지된 프로세스를 보여준다.

## 프로그라운드 백그라운드

백그라운드 작업을 포그라운드로 가져오는 방법 `fg 작업번호` 작업번호는 jobs 로 확인한다.

백그라운드로 작업을 실행 `실행명령 &`

이미 실행중인 포그라운드 작업을 백그라운드로 보낼 때 한번 일시정지 시킨다 `ctrl Z`

# 서비스

> 데몬(daemon)이란? 멀티태스킹 운영 체제에서 데몬은 사용자가 직접적으로 제어하지 않고, 백그라운드에서 돌면서 여러 작업을 하는 프로그램을 말한다.

보이지 않는 곳에서 일해주는 도깨비에서 착안

데몬은 항상 백그라운드에서 실행되는 스탠드얼론 방식과 필요할때 마다 부르는 inetd (보안 이슈로 xinetd) 방식이 있다. 

## 서비스와 소켓의 구분

서비스 : 평상시에도 늘 작동하는 서버 프로세스

소켓(socket) : 필요할 때만 작동하는 서버 프로세스

## 서비스의 특징

시스템과 별도로 구동되는 프로세스로 웹 서버 DB 서버 FTP 서버 등 이 있다.

`systemctl start/stop/restart`서비스명 으로 관리

## 소켓의 특징

서비스는 항상 구동 중이지만 소켓은 외부에서 특정 서비스를 요청하는 경우에만 systemd 가 구동한다. 요청 수행시 소켓은 종료

소켓과 관련된 스크립트 파일은 `/lib/systemd/system/` 디렉터리에 있는 `소켓명.socket`

# dpkg와 apt

`dpkg` : apt-get 이전의 패키지 관리 프로그램

`apt-get` : dpkg 의 확장 개념

`*.deb` : 데비안 계열

`*.rpm` : 레드햇 계열

**dpkg의 단점으로 의존성 관리를 못하는것이 있다. 때문에 apt-get 등장**

## apt-get 명령 패키지명

`apt-get install` : 패키지 설치, 패키지를 다운로드 후 사용자에게 물음

`apt-get update` : 다운로드 할 패키지 목록 업데이트 `/etc/apt/sources.list` 파일 내용 수정 시 

`apt-get remove` : 설치 된 패키지 삭제

`apt-get purge` : 설치 된 패키지와 설정 파일 까지 모두 삭제

`apt-get autoremove` : 사용하지 않는 패키지 모두 삭제

`apt-get clean` : 설치할 때 다운로드한 파일과 기존 파일 삭제

`apt-cache show` : 패키지의 정보를 보여줌

`apt-cache rdepends` :  의존 패키지 목록

## 우분투 저장소

`/etc/apt/sources.list` 파일 구성

각 행은 deb 우분투패키지저장소URL 버전코드명 저장소종류 를 의미한다.

### main, universe, restricted, multiverse의 의미

- main : 우분투에서 공식적으로 지원하는 무료 소프트웨어
- universe : 우분투에서 지원하지 않는 무료 소프트웨어
- restricted : 우분투에서 공식적으로 지원하는 유료 소프트웨어
- multiverse : 우분투에서 지원하지 않는 유료 소프트웨어

# GRUB 부트로더

- 부트 정보를 사용자가 임의로 변경하여 부팅할 수 있음
- 다른 운영체제와 멀티부팅 가능
- 커널의 경로와 파일 이름만 알면 부팅 가능
- 동적 모듈 로딩 가능
- ISO 이미지를 이용하여 바로 부팅 가능

## GRUB 부트로더 설정

 `/boot/grub/grub.cfg` : GRUB의 설정 파일

일반 사용자에게는 읽기 전용이며, root 사용자도 직접 편집해서는 안 됨

설정된 내용을 변경하려면 /etc/default/grub 파일과 /etc/grub.d/ 디렉터리의 파일을 수정한 후 

rub-mkconfig 명령을 실행해야 함

![Untitled5.png](https://github.com/resourceSaga/TIL/blob/master/Ubuntu%20in%20VM/Ubuntu%20Linux%20VM/Untitled%205.png?raw=true)

1행: GRUB 목록 중에서 0번째(첫 번째)가 기본으로 선택

2행: 3행의 시간 동안 화면에 GRUB 목록이 보이지 않게 함

3행: 처음 화면이 나오고 자동으로 부팅되는 시간을 초 단위로 설정

4행: 초기 부팅 화면의 각 엔트리 앞에 붙을 배포판 이름을 추출

5~6행: 부팅 시 커널에 전달할 파라미터를 지정

# 셸 환경

셸에서 저장된 환경 변수는 `echo $환경변수` 로 확인 가능

`export 환경변수=값` 명령을 실행하면 환경 변수 값을 변경

- 배쉬셸의 주요 환경 변수

![Untitled6.png](https://github.com/resourceSaga/TIL/blob/master/Ubuntu%20in%20VM/Ubuntu%20Linux%20VM/Untitled%206.png?raw=true)

## 셸 스크립트

**첫 줄에 #!/bin/셸이름**

```bash
#!/bin/bash
```

셸에서 변수를 사용 `$변수명`

종료 코드 반환 `exit 0`

셸 스크립트 파일은 sh 파일로 실행 가능하다. `sh file_name.sh`

`chmod +x file_name` 실행 가능 속성으로 변경 후 `./file_name.sh` 로 실행 가능

셸 스크립트는 변수를 미리 선언하지 않으며, `모든 값은 문자열`로 취급된다.

**변수명**은 `대문자와 소문자를 구분`한다.

변수 대입시 `=` 앞뒤에 공백이 없어야 한다. `last_name=yoon`

이어지는 문자열은 `" "`을 이용해 묶는다. `first_name="seo yul"`

`$`를 출력하려면 이스케이프 문자를 사용하거나 `''` 를 사용한다. 쌍따옴표 불가.

수치연산에는 `expr 연산내용` 으로 해야한다. 아니면 문자열 합으로 보여준다.

수치연산에 `*`는 이스케이프 해줘야 곱 연산이 가능하다.

### 변수

파라미터 변수는 `$숫자` 를 이용해 불러올 수 있다.

- $0 : 실행파일 이름
- $1 : 첫번째 파라미터
- $n  : n번째 파라미터
- $* : 모든 파라미터

### if

셸 스크립트의 if 조건문은 각 단어 사이에 반드시 공백이 있어야 한다.

```bash
if [ 조건 ]
then
 참인 경우 코드문
else
 거짓인 경우 코드문
fi

ex)
#!/bin/sh
if [ "book" = "book" ]
then
  echo "True"
else
  echo "False"
fi
exit 0
```

**조건문의 비교 연산자**

조건문은 문자열 비교와 산술 비교가 가능하다.

![Untitled7.png](https://github.com/resourceSaga/TIL/blob/master/Ubuntu%20in%20VM/Ubuntu%20Linux%20VM/Untitled%207.png?raw=true)

**파일 조건**

![Untitled8.png](https://github.com/resourceSaga/TIL/blob/master/Ubuntu%20in%20VM/Ubuntu%20Linux%20VM/Untitled%208.png?raw=true)

### case문

```bash
#!/bin/bash
case "$1" in
	start)
		echo "start!"
	stop)
		echo "stop!"
	restart)
		echo "restart!"
	*)
		echo "unknown"
esac
exit 0
```

**&& ||**

and 는 옵션 `-a` 로 사용 가능, or 는 옵션 `-o` 로 사용 가능 이때는 [ ] 안에서 사용

밖에선 그대로 `&&` 또는 `||` 를 사용하는것이 편하다.

키보드로부터 문자열을 받을때는 `read 변수명` 으로 받고 `$변수명` 으로 사용

### for ~ in 반복문

변수에 각 값을 넣은 후 do 안의 실행문을 실행한다.

seq 를 이용하면 순차적인 순열을 뜻한다. 1~10 은 `seq 1 10` 또는 bash 에서 `{1..10}` 와 `for (( i = 1 ; i <= 10 ; i ++ ))` 이 가능하다.

```bash
for 변수 in 값
do
	반복문
done

#!/bin/bash
hap=0
for i in $(seq 1 10)
do
	hap=`expr $hap + $i`
done
echo "1부터 10까지 합:"$hap
exit 0
```

### while문

조건이 참인 동안 계속 반복을 실행한다. 조건식 위치에 [ 1 ] 또는 [ : ] 이 오면 항상 참

```bash
#!/bin/bash
while [ 조건식 ]
do
	code문
done
exit 0
```

while문에서 `break`, `continue`, `exit` 하기위해서는 각 명령에 `;;` 두번찍는다

### until문

while문과 용도가 같지만 조건식이 거짓인 동안 계속 반복

### 셸 스크립트 함수

사용자가 함수를 만들어 사용할 수 있다.

```bash
함수명 () {
내용
파라미터 사용 위해서는 $1, $2 ... $n 으로 호출
} # 함수 정의
함수명 파라미터# 함수 호출
```

### eval

문자열을 명령문으로 인식하여 실행한다.

셸 스크립트 안에서 `eval "ls"` 하면 셸에서 ls 실행과 같다.

### export

변수를 전역 변수로 만들어서 모든 셸에서 사용 할 수 있도록 한다.

### set $(명령)

명령의 결과를 띄어쓰기로 구분하여 파라미터로 받는다. $1, $2 .. $n

## Cron 기능

`crontab -l` : 등록된 크론 확인

`crontab - e` : 크론을 등록, 수정

명령 형식 : `분 시 일 월 요일 실행유저 명령어`

crontab 의 위치 `/etc/crontab`
