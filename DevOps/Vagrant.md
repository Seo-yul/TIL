# Vagrant

# Vagrantfile 사용하기

1. 작업 디렉토리 생성
2. Vagrantfile 파일 생성

`vagrant init` 자동으로 vagrant 파일 을 생성해준다.

3. vagrant 파일 편집

`config.vm.box` 내가 설치 할 것을 담고 있는것 vagrant에서 기본으로 가지고 있다. 웹에서 검색해보면 나옴. [https://app.vagrantup.com/boxes/search](https://app.vagrantup.com/boxes/search)

```bash
# Vagrantfile 파일
Vagrant.configure("2") do |config|
		config.vm.box = "generic/centos7"
		config.vm.hostname = "demo"
		config.vm.network "private_network", ip: "192.168.33." # 아이피 안주면 DHCP 가 할당
		config.vm.synced_folder ".", "/home/vagrant/sync", disabled: true
		# 가상 머신하고 호스트 머신하고 공유 폴더를 지정 "호스트패스", "가상머신패스"
		# disable 하는 이유는 자동세팅하면 지금 오류가 있어서 막는것.
		config.vm.provision "shell", inline: $script
		# end 밑에서 script 를 넣는다.
end

$script = <<SCRIPT
	yum install -y epel-release
	yum install -y nginx
	echo "Hello, Vagrant" > /usr/share/nginx/html/index.html
  systemctl start nginx
SCRIPT
```

4. 세팅이 끝나면 vagrant up 으로 생성한다.

`Encoding::CompatibilityError: incompatible character encodings: UTF-8 and CP-949`

위와 같은 오류가 발생시 패스에 한글이 잡혀있는 경우 ex) 사용자 계정이 한글 로 인코딩 깨짐. 

해결방법으로는 사용자 홈 디렉토리 아래 `.vagrand.d` 폴더를 한글이 없는 디렉토리로 복사 한 후 경로 환경 변수에 VAGRANT_HOME 변수를 추가하고 이동한 `.vagrand.d` 경로를 지정

5. VM 에서 직접 로그인 vagrant / vagrant 또는 vagrantfile 이 있는곳에서 `vagrant ssh` 

6. ssh 설정 확인 vagrant ssh-config

7. IdentityFile 주소에 private_key 파일이 존재함!

8.본인이 사용하는 ssh 소프트웨어에 따라 key 설정 할 것 (putty, Bitvise, etc..)

## vagrant 명령어로 관리하기

도움말 `vagrant —help` 로 확인

주로 사용 : `destroy` , `init` , `halt`, `reload` , `resume` , `snapshot` , `ssh` , `ssh-config` , `suspend` , `up` 

서브 커맨드 도움말 subcommand -h

### 스냅샷 찍기 vagrant snapshot save snapshot_name

default: Snapshotting the machine as 'Snapshot_name'...

### 가상머신 정지 vagrant halt

default: Attempting graceful shutdown of VM..

### 가상머신 삭제 vagrant destory

default: Are you sure you want to destroy the 'default' VM? [y/N] y

### 가상머신 기동 중 프로비전 된 내용 반영

`vagrant provison` 또는 `vagrant reload —provision`

# Vagrantfile 인프라 구성의 장점

- 환경 구축 작업이 간소
- 환경 공유 용이
- 환경 파악 용이
- 팀 차원의 유지보수 가능