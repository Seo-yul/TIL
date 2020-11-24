# Ansible

Local 개발 환경의 Infrastructure as Code의 한계

- 구축 절차를 알기 어렵다
- 설정을 추가할 수 없다
- 구축 절차를 다른 환경에서 유용하기 힘들다.

    Scale Out 한 서버나 개발 환경/실제 운영 환경, 별도 시스템의 Web 서버 등에 대해 적용하고 싶은 경우 Vagrantfile에 직접 구축 절차와 설정 값을 기술해서는 코드의 유용성이 어려워진다. 왜냐하면 환경에 기반하는 파라미터가 반드시 존재하기 때문(CPU core, memory 등 spec 의존 등 heap..)

**결론 → 속인성의 배제와 상이한 환경에서 동일한 절차를 활용하고자 함**

# 인프라 구성 관리 도구

Ansible과 Chef 등 인프라 구성 관리 도구로 해결할 수 있다. 인프라 구성 관리 도구는 5가지의 특징이 있다.

### 선언적

### 추상화

### 수렴화

### 멱등성

### 간소화

# Ansible

https://www.ansible.com

- 구축 대상 서버에 구성 관리 클라이언트 도구를 도입할 필요가 없다.
- 정해진 포맷으로 설정을 간단하게 기술할 수 있다.
- 경우에 따라 명령어 하나로(Ansible 설정 없이) 실행할 수 있어 도입이 용이하다.

Ansible은 파이썬 환경에서 돌아간다.

## 주요 특징

- 에이전트리스

    공급 대상 서버에 에이전트를 도입할 필요가 없다. 따라서 설치가 간편하다.

- YAML 형식의 설정 파일

    YAML 형식의 인프라 구성을 선언하기 때문에, 프로그램에 익숙하지 않은 사람도 취급하기 쉽다.

- 설치가용이

    패키지 하나로 도입이 가능하여 구성 관리를 시작하기 쉽다.

## 가상 머신 환경에서 Ansible 설치

먼저, nginx가 실행중이라면 데모를 실행하기 위해 정지한다.

`sudo systemctl stop nginx.service`

EPEL(Linux의 확장 패키지 레포지토리 셋)을 먼저 업데이트 해주고.

`sudo yum -y install epel-relese`

ansible을 설치한다.

`sudo yum -y install ansible`

## 가상 머신 환경의 Ansible 버전 확인

`ansible --version`

![Ansible%200efc4608e41245e0b2fe6abe32b6db94/Untitled.png](Ansible%200efc4608e41245e0b2fe6abe32b6db94/Untitled.png)

## ansible 명령어에 의한 nginx 기동

Ansible을 단독으로 실행하려면 ansible 명령어를 이용한다.

`sudo sh -c "echo \"localhost\" >> /etc/ansible/hosts"`

`/etc/ansible/host`는 Inventory File이라고 불려지는 것으로, Ansible에 의해 향후 원격 실행을 하기 위한 대상이 되는 서버 목록을 정의한다. 여기 기재된 서버를 실행 대상으로 해서 선택하는 것이 가능하다.

`ansible localhost -b -c local -m service -a "name=nginx state=started"`

ansible 명령어 실행으로 '로컬 호스트의 nginx 서비스를 기동 상태가 되게 한다.' 의 지시를 한다.

- 명령 구문 의미 ( 인수 : 의미 )

    [localhost](http://localhost) :  Inventory File에 기재된 서버 중에서 이번 명령어에 의해 실제 명령 실행을 수행하는 대상을 정의

    -b : 원격 실행되는 대상 서버에서 어떤 사용자에 의해 조작이 실행되는가? -b의 경우 root 사용자로 실행

    -c local : 대상 서버가 자기 자신인 경우 ssh가 필요 없기 때문에, local 연결을 하기 위해 부여(일반적으로 원격 서버 접속은 ssh로 실행)

    -m service : service 모듈을 이용하는 것을 정의, Ansible에서 모듈이란 조작의 종류이다. 예를 들어 "파일을 생성한다/삭제한다.", "서비스를 시작한다./중지한다.", "패키지를 설치한다./제거한다." 라는 단위로 처리되는 것을 가리킨다. 모듈은 다양한 것이 준비되어 있다.
    http://docs.ansible.com/ansible/modules.html

nginx가 중지된 상태에서 명령 실행시

![Ansible%200efc4608e41245e0b2fe6abe32b6db94/Untitled%201.png](Ansible%200efc4608e41245e0b2fe6abe32b6db94/Untitled%201.png)

changed 구문에서 true 확인가능

nginx가 이미 실행된 상태에서 명령 실행시

![Ansible%200efc4608e41245e0b2fe6abe32b6db94/Untitled%202.png](Ansible%200efc4608e41245e0b2fe6abe32b6db94/Untitled%202.png)

changed 가 false임을 확인가능

이와같이 언제 실행하여도 항상 nginx가 실행됨을 유지하는 선언적인 것으로 상태가 수렴화 된다 하는 멱등성을 지닌다. nginx가 시작 상태임을 보증하는것.

## Ansible을 잘 다루는 법 ansible-playbook

ansible 명령은 하나의 처리를 조작하지만 실제 과정에선 그렇지 않다. 때문에 `ansible` 명령어가 아닌 `ansible-playbook`명령어를 이용하여 처리와 대상을 Group화하여 함께 관리한다.

`ansible-playbook` 명령어는 playbook 파일이라고 하는 것으로, 구축 정보에 대한 정의를 미리 실시해야 한다.

### Ansible 실행결과

- ok : 결과가 이미 예상했던 대로 되었다.(즉, 아무것도 할 필요가 없었기 때문에 실행하지 않았다.)
- skip : 명시적인 조건에 의해 Task 자체가 Skip 되었다.(실행하지 않았다.) 예를들어 'Task 1이 성공하면 Task 2는 실행하지 않는다.' 라는 정의를 Ansible에 미리 설정해 놓은 경우가 있다.
- changed : Task 실행에 의해 예상했던 대로 변경되었다.
- unreachable : 원래의 실행 대상 호스트에 통신이 도달하지 않았다.(Error)
- failed : 실행 대상 호스트에 도달했지만, 조작이 어떤 이유로 실패했다.(Error)

![Ansible%200efc4608e41245e0b2fe6abe32b6db94/Untitled%203.png](Ansible%200efc4608e41245e0b2fe6abe32b6db94/Untitled%203.png)

### playbook 파일?

어떤 서버에서 어떤 작업을 수행하는 방법을 정의하는 것이 playbook 파일이다. 샘플에서 site.yml 이 해당한다.

```yaml
---
- hosts: webservers
  become: yes
  connection: local
  roles:
    - common
    - nginx
```

라인 순서대로 의미

- 실행 대상을 결정한다.  별도 정의되어 있는 Inventory File 내부에서 webservers의 Host Group에 대한 처리를 실행하는 것을 의미함
- 대상 호스트에 root 사용자로 작업을 수행하도록 지정
- 대상 호스트가 Remote가 아니기 때문에 ssh 대신 local 연결을 하는 것을 의미
- role로써 common과 nginx처리를 수행하는 것을 지정

## 실행 대상을 정의하는 playbook 과 Inventory File

playbook 파일 안에 `host: webservers`는 별도 정의된 Inventory File에서 지정한 그룹을 참조한다. ansible 명령어의 예시는 /etc/ansible/hosts에 기재되어 있지만, Inventory File은 여기처럼 별도 파일에 정의하여 -i 옵션으로 지정하는것이 가능하다. 

예시 development 인벤토리 파일

```yaml
[development-webservers]
localhost

[webservers:children]
development-webservers
```

부모/자식 관계. localhost는 webservers 그룹과 development-webservers 그룹에 속한다.

ansible-playbook 명령 실행 시 playbook 파일 내에서 webservers로 지정하고 있기 때문에, Inventory File을 참조한 후 localhost를 실행 대상으로 하고 있다.

## 실행 내용을 정의하는 playbook 과 role

playbook 파일인 `site.yml` 에는 `role`로서 common과 nginx 두 가지가 지정되어 있지만, 구체적인 수행 내용은 없다. 이 처리 내용은 roles directory 아래 각각 `tasks` 안에 기록되어 있다.

![Ansible%200efc4608e41245e0b2fe6abe32b6db94/Untitled%204.png](Ansible%200efc4608e41245e0b2fe6abe32b6db94/Untitled%204.png)

## dry-run 모드의 실행과 변화 확인

새로운 템플릿을 환경에 반영하려고 할 경우, 실제 환경에 갑자기 반영하는 것이 불안하다 생각될 때, 실제 반영하기 전 변화를 예상하고 확인할 때 사용한다. Ansible에서는 dry-run 모드로 불리는 실행 옵션이다. 이 모드를 실행하면 실제 반영하진 않고 "실행하면 어디가 변경되는지" 미리 표시해 준다.

`ansible-playbook -i development site.yml --check --diff`

--diff 옵션으로 변경을 확인하고 --check 옵션으로 드라이 모드를 이용한다.  --check를 빼면 실제로 반영한다.

![Ansible%200efc4608e41245e0b2fe6abe32b6db94/Untitled%205.png](Ansible%200efc4608e41245e0b2fe6abe32b6db94/Untitled%205.png)

# 인프라 구성 관리 도구의 해결점

- 구축 절차를 이해하기 어렵다

    Ansible이 가지는 '선언적'인 기술 방법에 의해 상태가 '수렴화'되도록 기재할 수 있다. 즉, 절차가 아닌 결과만을 보는 것이 가능해졌다.

- 설정을 추가할 수 없다.

    모든 설정은 '멱등성'과 '수렴화'에 의해 의존 관계로부터 해방되었다. 이는 전제가 되는 환경 조건을 걱정할 필요가 없어졌으며, 설정의 추가도 도구에게 맡기며 기재만 하면 해결되도록 하였다.

- 구축 절차를 다른 환경에서 유용하기 어렵다.

    '추상화' 기술에 의해 OS 등의 환경 조건을 걱정하지 않고 구축으로 인해 나타나는 상태만을 간단하게 이해할 수 있게 되었다.

    # 기타 인프라 구성 관리 도구

    Chef, Puppet