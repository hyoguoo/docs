---
layout: editorial
---

# Instruction Level Parallelism(명령어 병렬 처리 기법)

> 컴퓨터는 명령어를 빠르고 효율적으로 처리하기 위해 CPU 쉬지 않고 작동 시키기 위해 명령어 병렬 처리 기법을 사용한다.

명령어 병렬 처리 기법에는 대표적으로 명령어 파이프라이닝, 슈퍼스칼라, 비순차적 명령어 처리 등이 존재한다.

## 명령어 파이프 라인

명령어 처리 과정은 클럭 단위로 나누어 보면 일반적으로 아래와 같이 4단계로 구분할 수 있다.

1. 명령어 인출(Instruction Fetch)
2. 명령어 해석(Instruction Decode)
3. 명령어 실행(Execute Instruction)
4. 결과 저장(Write Back)

위에서 같은 단계가 겹치지 않으면 CPU는 각 단계를 동시에 실행할 수 있기 때문에, `인출` 하는 동안 다른 명령어인 `해석`, `실행`, `저장`을 실행할 수 있다.  
때문에 CPU는 명령어를 병렬적으로 처리할 수 있게 되는데, 이를 명령어 파이프라인이라고 한다.

하지만 특정 상황에서는 성능 향상을 가져오지 않기 때문(파이프라인 위험)에 주의해야 한다.

## 슈퍼스칼라

파이프라이닝은 단일 파이프라인으로 구현 가능하지만, 현대 CPU에서는 여러 개의 파이프라인을 이용해 여러 개의 명령어 파이프라인을 포함한 구조를 가지고 있다.(=슈퍼스칼라)

```
# 단일 파이프라인
명령어 1 : 인출 -> 해석 -> 실행 -> 저장
명령어 2 : --  -> 인출 -> 해석 -> 실행 -> 저장
명령어 3 : --  -> --  -> 인출 -> 해석 -> 실행 -> 저장
명령어 4 : --  -> --  -> --  -> 인출 -> 해석 -> 실행 -> 저장
...

# 슈퍼스칼라
명령어 1 : 인출 -> 해석 -> 실행 -> 저장
명령어 2 : 인출 -> 해석 -> 실행 -> 저장
명령어 3 : --  -> 인출 -> 해석 -> 실행 -> 저장
명령어 4 : --  -> 인출 -> 해석 -> 실행 -> 저장
...
```

위의 구조로 명령어를 처리하기 때문에 파이프라인 개수에 비례하여 프로그램 처리 속도가 증가한다.  
하지만 명령어 파이프 라인에서 발생하는 파이프라인 위험 등 예상치 못한 문제가 있기 때문에 항상 성능이 비례하여 증가하는 것은 아니다.

## 비순차적 명령어 처리

이름 그대로 명령어를 순차적으로 실행하지 않고, 순서를 바꿔가며 실행하는 기법으로, 현대 CPU의 성능 향상에 크게 기여한 기법이다.(현재 대부분의 CPU가 차용 중)  
위에서 소개된 처리 기법들은 순차적으로 처리하는 도중 예상치 못한 문제로 인해 파이프라인이 멈춰 버리게 되는데, 이를 해결하기 위해 순서를 바꿔가며 더 효율적인 처리를 할 수 있게 해준다.

###### 참고자료

- [혼자 공부하는 컴퓨터 구조+운영체제](https://kobic.net/book/bookInfo/view.do?isbn=9791162243091)
