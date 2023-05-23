# 인터럽트

프로그램을 실행 중에 예기치 않은 상황이 발생할 경우 현재 실행중인 작업을 중단하고 발생된 상황을 처리한 후 다시 실행중인 작업으로 복귀하는 작업

## 종류

- 외부 인터럽트
    - 전원 이상 인터럽트(Power Fail Interrupt)
    - 기계 착오 인터럽트(Machine Check Interrupt)
    - 외부 신호 인터럽트(External Interrupt)
    - 입출력 인터럽트(I/O Interrupt)

- 내부 인터럽트
    - 프로그램 검사 인터럽트(Program Check Interrupt)

## 동작 순서

1. 인터럽트 요청
2. 프로그램 실행 중단: 현재 실행중이던 Micro operation 까지 수행한 후 중단
3. 현재 프로그램 상태 보존: PCB(Process Control Block), PC(Program Counter) 등의 정보를 저장
4. 인터럽트 처리 루틴 실행: 인터럽트를 요청한 장치 식별
5. 인터럽트 서비스 루틴 실행: 인터럽트를 발생시킨 원인 파악 후 실질적인 작업 수행
6. 상태 복구: 인터럽트 발생 시 저장해둔 3번의 정보를 복구
7. 중단된 프로그램 실행 재개: PC(Program Counter)의 값을 이용하여 수행 중이던 프로그램 재개

###### 출처

- [곰책으로 쉽게 배우는 최소한의 운영체제론](https://www.inflearn.com/course/곰책-쉽게-배우는-운영체제)
- [쉽게 배우는 운영체제](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=쉽게+배우는+운영체제&ebookYn=N&isbn=9791156648888&cipId=228149278%2C)