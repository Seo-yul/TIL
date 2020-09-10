# 앤서블 Ansible

# 인프라 구성 관리 도구

특징 : 선언적, 추상화, 수렴화, 멱등성, 간소화

파이썬으로 만들어진 인프라 구성 관리 도구

앤서블은 본체와 인벤토리, 모듈로 이루어진다.

본체: 앤서블 소프트웨어 자체

인벤토리 : 대상(서버)를 정의하고 있는것

모듈 : 서비스 명령어

설치 : `yum install -y ansible`

### 인벤토리 작성 하고 실행

```bash
# /etc/ansible/hosts 파일 마지막에 localhost 추가
sudo sh -c "echo \"localhost\" >> /etc/ansible/hosts"
ansible localhost -b -c local -m service -a "name=nginx state=started"
```

`ansible localhost -b -c local -m service -a "name=nginx state=started"`

- ansible : 실행
- [localhost](http://localhost) : 인벤토리 파일에 기재된 서버 중 이번 명령어를 수행할 대상
- -b : 원격 실행되는 대상 서버의 사용자 (-b = root)
- -c local : 대상 서버가 자신이므로 ssh를 사용하지 않고 local로 연결
- -m service : service 명령어를 사용한다.
- -a "name=nginx state=started" : 모듈의 추가 인자.

## playbook 을 실행해서 구축

`ansible-playbook -i 인벤토리파일명 yml파일명`

기본 인벤토리 파일은 `/etc/ansible/hosts` 를 사용하지만 -i 옵션을 이용해 지정할 수 있다. 

## yaml 파일

```yaml
---                                    # yaml 파일이라는 의미
- hosts: webservers                    # 배포 대상 호스트
  become: yes                          # 대상 호스트에 root 사용자로 작업 수행하도록 지정
  connection: local                    # 대상 호스트가 원격이 아니므로 ssh 대신 local 연결
  roles:                               # roles 역할은 목록만 있고 실제 내용은 다른 파일
    - common                           # roles 중 common 과 nginx 를 실행
    - nginx
#    - serverspec
#    - serverspec_sample
#    - jenkins
```

### roles 의 경로를 보면 인벤토리 와 같은 레벨에 roles 라는 디렉토리 안에 존재

## 실행(task) 내용 정의를 확인

role 별로 실행 될 내용을 담고 있다. 각 디렉토리는 그 역할 에서 수행해야 할 내용을 정의한다.

예시 nginx/tasks/main.yml

```yaml
# tasks file for nginx
- name: install nginx
  yum: name=nginx state=installed

- name: replace index.html
  template: src=index.html.j2 dest=/usr/share/nginx/html/index.html

- name: nginx start
  service: name=nginx state=started enabled=yes
```

## templates 내용 확인

roles 디렉토리 안에 templates 를 확인한다.

템플릿에서 사용하는 변수 값은  `group_vars` 디렉토리에 그룹별로 정의되어 있다.

## dry-run 모드로 실행

실제로 변경된 내용을 반영하지 않고 반영되었을 때 결과를 미리 확인

`ansible-playbook -i development site.yml --check --diff`

`--check` : dry-run 모드로 실행

`--diff` : 변경의 차이를 표시

![Untitled.png](./Ansible/Untitled.png?raw=true)

templates에서 변경한 것을 확인 가능. 하지만 실제로 적용안됨.

## 변경을 실제 적용

`ansible-playbook -i development site.yml --diff`

실제로 적용됨

# 인프라 테스트 자동화

## Serverspec

- 테스트 수행을 간단하고 쉽게 하기 위한 도구
- 인프라(서버) 설정 테스트 가능
- 테스트 항목에 대한 목록을 정해진 포맷을 기반으로 기술이 가능
- 테스트 결과를 리포트 형식으로 출력이 가능

## Ansible을 이용해 Serverspec 을 설치

0.  루비를 사용하기 위해 루비 설치.

```bash
# ruby version manager 설치
command curl -sSL https://rvm.io/mpapis.asc | sudo gpg2 --import -
command curl -sSL https://rvm.io/pkuczynski.asc | sudo gpg2 --import -
curl -L get.rvm.io | sudo bash -s stable
sudo usermod -aG rvm $USER
source /etc/profile.d/rvm.sh
rvm reload
# ruby 설치
sudo su
rvm requirements run
rvm install 2.7

# ruby 기본 버전 수정
rvm use 2.7 --default

# 사용자 계정에서 ruby 위치 확인
which ruby
# root 계정 ruby 위치 확인
sudo which ruby

# root 계정의 ruby 이동
sudo mv /bin/ruby /bin/ruby_2.0.0

# root 계정 ruby 기본 버전 변경
sudo ln -s /usr/local/rvm/rubies/ruby-2.7.0/bin/ruby /bin/ruby

# root 계정과 사용자 계정 루비 버전 확인
ruby -v
sudo ruby -v
```

1. Playbook 파일(site.yml)에서 serverspec 롤을 추가

```yaml
# site.yml

---
- hosts: webservers
  become: yes
  connection: local
  roles:
    - common
    - nginx
    - serverspec
#    - serverspec
#    - serverspec_sample
#    - jenkins
```

2. serverspec 롤 확인

`./roles/serverspec/tasks/main.yml`

3.  serverspec 확인

`ansible-playbook -i development site.yml --diff`

4.  serverspec 설정

`serverspec-init`

```bash
[vagrant@demo tasks]$ serverspec-init
Select OS type:

  1) UN*X
  2) Windows

Select number: 1

Select a backend type:

  1) SSH
  2) Exec (local)

Select number: 2

 + spec/
 + spec/localhost/
 + spec/localhost/sample_spec.rb                  # 샘플 파일
 + spec/spec_helper.rb
 + Rakefile
 + .rspec
```

5. sample_spec.rb  파일 확인

`cat ./spec/localhost/sample_spec.rb`

```ruby
require 'spec_helper'

describe package('httpd'), :if => os[:family] == 'redhat' do   # 레드햇 계열일때
  it { should be_installed }                 # httpd 가 설치되어 있어야 함
end

describe package('apache2'), :if => os[:family] == 'ubuntu' do
  it { should be_installed }
end

describe service('httpd'), :if => os[:family] == 'redhat' do # 레드햇 계열일때
  it { should be_enabled }             # httpd 서비스가 활성화되어 있어야 한다.
  it { should be_running }             # httpd 서비스가 실행되고 있어야 한다.
end

describe service('apache2'), :if => os[:family] == 'ubuntu' do
  it { should be_enabled }
  it { should be_running }
end

describe service('org.apache.httpd'), :if => os[:family] == 'darwin' do
  it { should be_enabled }
  it { should be_running }
end

describe port(80) do
  it { should be_listening }
end
```

6. serverspec을 이용한 테스트 실행

`rake spec` : 자동으로 _spec.ruby 파일을 찾아서 실행해준다.

## Ansible에 Serverspec을 이용한 테스트 롤 추가

1. Playbook 파일(site.yml)에 Serverspec 테스트 롤을 추가

```yaml
---
- hosts: webservers
  become: yes
  connection: local
  roles:
    - common
    - nginx
    - serverspec                 
    - serverspec_sample           # 주석 해제 후 저장
#    - jenkins
```

2. serverspec_sample 롤 정의 파일을 확인

`cat ./roles/serverspec_sample/tasks/main.yml`

```yaml
---
# tasks file for serverspec_sample
- name: distribute serverspec suite
  copy: src=serverspec_sample dest={{ serverspec_base_path }}

- name: distribute spec file
  template: src=web_spec.rb.j2 dest={{ serverspec_path }}/spec/localhost/web_spec.rb
```

**task 에서 사용하는 변수를 정의**

`cat ./roles/serverspec_sample/vars/main.yml`

```yaml
serverspec_base_path: "/tmp"
serverspec_path: "{{ serverspec_base_path }}/serverspec_sample"
```

`cat ./roles/serverspec_sample/templates/web_spec.rb.j2`

```ruby
require 'spec_helper'

describe package('nginx') do
  it { should be_installed }
end

describe service('nginx') do
  it { should be_enabled }
  it { should be_running }
end

describe port(80) do
  it { should be_listening }
end

describe file('/usr/share/nginx/html/index.html') do
  it { should be_file }
  it { should exist }
  its(:content) { should match /^Hello, {{ env }} ansible!!$/ }
end
```

3. ansible-playbook 으로 spec 파일을(test case file)을 배포

`ansible-playbook -i development site.yml -diff`

```bash
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details

PLAY [webservers] *********************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [localhost]
...
...
PLAY RECAP ****************************************************************************************
localhost                  : ok=9    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

4. spec 파일(테스트 케이스를 정의) 생성을 확인

`cat /tmp/serverspec_sample/spec/localhost/web_spec.rb`

```ruby
require 'spec_helper'

describe package('nginx') do
  it { should be_installed }
end

describe service('nginx') do
  it { should be_enabled }
  it { should be_running }
end

describe port(80) do
  it { should be_listening }
end

describe file('/usr/share/nginx/html/index.html') do
  it { should be_file }
  it { should exist }
  its(:content) { should match /^Hello, development ansible!!$/ }
end
```

5. 테스트하는 방법 !!!!!!! ___ 배포된 spec 파일을 이용해서 테스트를 실행

`cd /tmp/serverspec_sample/`    그 경로로 가서

**`rake spec`**                                  rake 를 사용해 실행

```bash
File "/usr/share/nginx/html/index.html"
  is expected to be file
  is expected to exist
  content
    is expected to match /^Hello, development ansible!!$/ (FAILED - 1)
```

실패를 확인 할 수 있다.  test case 랑 배포된 템플릿과 내용이 달라서

6. 테스트 케이스를 통과하도록 컨텐츠를 수정

7. ansible-playbook으로 배포 후 테스트를 실행

`ansible-playbook -i development site.yml`

![Untitled%201.png](./Ansible/Untitled%201.png?raw=true)

통과!

## coderay를 이용한 html에서 테스트 케이스 확인

```bash
sudo gem install coderay
rake spec SPEC_OPTS="--format html" > ~/result.html
sudo mv ~/result.html /usr/share/nginx/html/
sudo setenforce 0
sudo systemctl start nginx.service
```

`sudo firewall-cmd --permanent --zone=public --add-port=80/tcp` 웹서버 방화벽  열고 가상머신 ip의 result.html 에서 확인한다.
