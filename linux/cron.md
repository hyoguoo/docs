# 리눅스 반복 예약 작업

## 설정
---

- 설정 : `crontab -e`

- 목록 : `crontab -l`

- 제거 : `crontab -r`

### 작성 서식

```
15,30 10-18/2 * * * /Users/command.sh
# [min] [hour] [day] [month] [week sun~mon] [command]
```

-> 10시~18시 동안 2시간 간격으로 15분 30분마다 실행

- \* : 해당된 매 단위마다 실행
- num1-num2 : num1~num2 사이동안 실행
- num1,num2 : num1,num2 때만 실행
- /num : num 간격으로 실행

[시간 서식 사이트](https://crontab.guru/)

#### crontab을 통한 python 실행 방법

```
* * * * * /usr/bin/python test.py
# [시간] [파이썬 설치 경로] [실행할 파일]
```