# 13. 파이썬 SQLite 연동, 테이블 생성 삽입

## SQLite 연동하기

```python
import datetime
import sqlite3

# 삽입 날짜 생성
now = datetime.datetime.now()
print('now', now)

nowDatetime = now.strftime('%Y-%m-%d %H:%M:%S')
print('nowDatetime', nowDatetime)

# sqlite3 버전
print('sqlite3.version : ', sqlite3.version)
print('sqlite3.sqlite_version', sqlite3.sqlite_version)

# DB생성 & Autocommit
# 본인 DB 파일 경로
conn = sqlite3.connect('원하는 경로/database.db', isolation_level=None)
# database.db 라는 sqlite에서 사용하는 db 파일이 생성된다.

# DB생성(메모리)
# conn = sqlite3.connect(":memory:")
# auto commit 사용으로 지금은 사용 안함 isolation_level=None

# Cursor연결
c = conn.cursor()
print('Cursor Type : ', type(c))
```

아래 모든 코드는 하나의 파일안에서 이어진다.

## 테이블 생성

```python
c.execute(
    "CREATE TABLE IF NOT EXISTS users(id INTEGER PRIMARY KEY, username text, email text, phone text, website text, regdate text)")  # AUTOINCREMENT
```

## 데이터 삽입

```python
# prepareStatement 를 이용한다.
c.execute("INSERT INTO users VALUES (1 ,'Kim','Kim@naver.com', '010-0000-0000', 'Kim.com', ?)", (nowDatetime,)) # 튜플형태로 ? 에 입력처리
c.execute("INSERT INTO users(id, username, email, phone, website, regdate) VALUES (?, ?, ?, ?, ?, ?)",
          (2, 'Park', 'Park@naver.com', '010-1111-1111', 'Park.com', nowDatetime))

# Many 삽입(튜플, 리스트)
userList = (
    (3, 'Lee', 'Lee@naver.com', '010-2222-2222', 'Lee.com', nowDatetime),
    (4, 'Cho', 'Cho@naver.com', '010-3333-3333', 'Cho.com', nowDatetime),
    (5, 'Yoo', 'Yoo@naver.com', '010-4444-4444', 'Yoo.com', nowDatetime)
)
c.executemany("INSERT INTO users(id, username, email, phone, website, regdate) VALUES (?, ?, ?, ?, ?, ?)", userList)
```

## 데이터 삭제

```python
print("users db deleted : ", conn.execute("delete from users").rowcount, "rows")
```

## 커밋 - isolation_level=None 일 경우 자동 반영(Auto Commit)

```python
conn.commit()
```

## 롤백

```python
conn.rollback()
```

## 접속 해제

```python
conn.close()
```