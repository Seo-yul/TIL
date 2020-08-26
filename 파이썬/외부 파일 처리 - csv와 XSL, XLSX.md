# 외부 파일 처리 - csv와 XSL, XLSX

## 파이썬 Excel, CSV 파일 읽기 및 쓰기

```python
with open('./resource/sample2.csv', 'r') as f:
    reader = csv.reader(f, delimiter='|')  # 구분자 선택
    # next(reader) Header 스킵
    # 확인
    print(reader)
    print(type(reader))
    print(dir(reader))  # __iter__ 확인
    print()

    for c in reader:
        print(c)
```

## csv 데이터의 dict 변환

```python
with open('./resource/sample1.csv', 'r') as f:
    reader = csv.DictReader(f)
    # 확인
    print(reader)
    print(type(reader))
    print(dir(reader))  # __iter__ 확인
    print()

    for c in reader:
        for k, v in c.items():
            print(k, v)
        print('-----')
```

## 데이터를 csv로 쓰기

```python
# writerow 사용해서 한줄씩 쓰기
with open('./resource/sample3.csv', 'w') as f:  # newline='' 테스트
    wt = csv.writer(f)
    # dir 확인
    print(dir(wt))
    print(type(wt))
    for v in w:
        wt.writerow(v)

# writerows 한번에 모든 리스트를 다 쓰게함
with open('./resource/sample3.csv', 'w', newline='') as f: # newline 개행 형식
    wt = csv.writer(f)
    # dir 확인
    print(dir(wt))
    print(type(wt))

    wt.writerows(w) 
```

## XSL, XLSX 사용하기

- XSL, XLSX : MIME - applications/vnd.excel, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
- pip install pandas 설치 필요
- pip install xlrd 설치 필요
- pip install openpyxl 설치 필요
- openpyxl, xlsxwriter, xlrd, xlwt, xlutils 등 있으나 pandas를 주로 사용(openpyxl, xlrd) 포함