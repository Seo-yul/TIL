# Git 특강

파이참에서 ctrl alt s 누르면 설정

terminal 설정

ignore 세팅

[https://www.toptal.com/developers/gitignore](https://www.toptal.com/developers/gitignore)

# 장고의 경우

django-admin startproject

일반 장고의 경우 rest api 만드는게 힘들다 때문에

restframe work라는것을 사용한다.

# 의존성 관리

requirements.txt 라는 거로 pip list목록을 만들어 의존성을 관리한다.

**pip freeze > requirements.txt**

pip install -r requirements.txt

## venv 가상환경(독립환경)

python -m venv venv

python 으로 venv를 만들겠다 이름은? venv로 (순서대로 의미함)

source venv/Scripts/activate

# 사용자 관리

git config —global [user.name](http://user.name) 'username'

git config —global [user.email](http://user.email) 'usermail@email.com'

폴더 안에서만 환경바꿈

git config —local [user.name](http://user.name) 'username'

git config —local [user.email](http://user.email) 'useremail'

# git 초기화

git init

# git diff

git diff 사용시 더 명확히 보는방법

—color-words

라인대신 단어를 색으로 표현해준다.

현재 git add 한걸 제외한다. ( staged 된거 다시 unstaged 하게 )

git restore —staged <filename>

# 트래킹 하지 않는 파일 목록

.gitignore

# git stash

[https://git-scm.com/book/ko/v2/Git-도구-Stashing과-Cleaning](https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-Stashing%EA%B3%BC-Cleaning)

branch 이동시 커밋하지 않고 이동하려고.. 잠시 다녀오는 느낌으로 저장해둠

워킹 디렉토리에서 수정한 파일들만 저장한다.

git stash

git stash list 로 저장한 stash를 확인 할 수 있다.

git stash apply (stash 이름)를 사용하여 다시 저장할 수 있다.

이름없이 apply 하면 가장 최근의 stash를 적용한다.

어플라이시 -index 옵션으로 staged 상태까지 적용할 수 있다.

# git add

취소하기 파일 상태를 Unstage 변경하기

git rest HEAD [file] 

파일명이 없으면 add한 파일 전체를 취소한다.

# git push

``git push -u origin <branch name>`

# python django 프로젝트를 진행한다.

1. .gitignore부터 진행
2. python -m venv venv
3. source venv/scrpits/activate
4. dajngo-admin startproject django_git .
5. gitignore 넣어준다.
6. git init

도커를 사용한다면 도커 이그노어도 먼저 만들고 시작

# 파이참 인터프리터 venv 로 세팅잡기

세팅에서 인터프리터 검색, 프로젝트: 프로젝트명 아래 파이썬 인터프리터

[manage.py](http://manage.py) 를 못잡는경우

Language & freamworks 에서 스크립트 파일로 manage.py를 잡는다.

# branch 분기 사용하기

- 분기 나누기

`git branch <branch name>`

분기를 나누면서 이동

`git checkout -b <branch name>`

- 분기 목록 확인

`git branch`

- 분기 삭제

`git branch -d <branch name>`

- 분기에서 파일 체크

분기에서 파일 체크는 tracked 된 commit 파일만 체크한다. 때문에 새로만들어 untracked 인 파일은 branch switch를 하여도 계속 존재한다.

## **switch: 브랜치를 변경한다.**

브랜치를 (없는 경우 생성하면서) 변경할 수 있다.

- 브랜치 변경

    `git switch develop
    Switched to branch 'develop'`

- 브랜치 생성 및 변경

    `git switch -c feature/git-switch
    Switched to a new branch 'feature/git-switch'`

이 외에도 다양한 선택사항이 존재한다. `git switch --help`로 확인가능하다.

## 브랜치 합치기

git switch <합칠 branch name>

git merge <합쳐질 branch name>

ex) master에서 reso를 합친다.

`git switch master`

`git merge reso`

## 브랜치 합치고 정리하기

git branch -d <branch name>

# 깃 저장소에서 파일 내려받기

git clone (url path) [디렉토리명]

# git commit

## commit 에 추가하기 stage

커밋을 했는데 stage하는것을 깜빡하고 빠트린 파일이 있을때

add를 더 한후.

git commit —amend

## 파일 상태를 unstage로 변경하기

**restore: 작업중인 파일을 제외한다.**

git restore <file> 더이상 추적하지않음.

git restore —staged <file> 추적된거 빼기

## 마지막 commit 상태로 파일 되돌리기

git restore — <file>

# git log 확인하기

`git log`

그래프로 보기 `git log --pretty=format:"%h %s" --graph`

# origin 이란?

remote 이름을 말하는 것이다.

remote -v 명령으로 remote 목록을 확인 가능

# pull request 보내기

branches 를 눌러 merge 요청을 보내면 마스터에 머지 할 수 있도록 요청을 보낸다.

# fetch

최신본에서 commit 을 제외하고 변경 사항만 받아낸것. commit은 개발자가 직접 해야한다.

# 장고 프로젝트 시작하기

```python
mkdir DJANGO_COLLABO
cd DJANGO_COLLABO
python -m venv venv
source venv/Scrips/activate
python -m pip install --upgrade pip
pip install django django_extensions ipython
pip freeze > requirements.txt
django-admin startproject django_collabo .

장고 세팅 django의 manage script가 잘 잡혔는지 확인.

gitignore.io에서
python django venv

app을 만든다.
ex) django-admin startapp <app name>

django_extensions를 쓰는 방법
settings.py의 INSTALLED_APPS에 django_extensions 를 추가한다.
```

django queryset ←  중요! django documentation aggregation 

https://cholol.tistory.com/467