---
layout: editorial
---

# 교착 상태(Deadlock)

여러 프로세스가 서로의 자원을 점유하고 기다리면서 진행이 멈추는 상태로, 서로가 소로의 자원을 기다리고 있는 상태를 말한다.  
교착 상태에서는 어떤 프로세스도 진행하지 못하고 무한정 기다리게 될 수 있다.

## 교착 상태 발생 조건

교착 상태는 다음의 네 가지 조건이 있으며, 네 가지 모든 조건이 동시에 성립할 때 교착 상태가 발생한다.

1. 상호 배제(Mutual Exclusion): 자원은 한 번에 한 프로세스만 사용할 수 있어야 한다.
2. 점유 대기(Hold and Wait): 최소한 하나의 자원을 점유한 상태에서 다른 자원을 기다려야 한다.
3. 비선점(No Preemption): 다른 프로세스가 자원을 강제로 빼앗을 수 없어야 한다.
4. 원형 대기(Circular Wait): 각 프로세스는 순환적으로 다음 프로세스가 요구하는 자원을 가지고 있어야 한다.
   
위 조건들이 동시에 모두 충족하면 교착상태가 발생하게 되며, 발생했을 때는 시스템을 다시 시작하거나 프로세스를 강제 종료시켜야 한다.

## 교착 상태 해결

교착 상태를 해결하기 위해서는 애초에 위의 상황이 충족되지 않도록 예방, 회피 혹은 회복하는 방법을 사용하여 교착 상태를 해결할 수 있다.

### 예방

위에 명시 된 네 가지 중 하나를 충족하지 못하게 하여 교착 상태를 예방하는 방법이다.  
여기서 현실적으로 가장 합리적인 방법은 원형 대기를 방지하는 것으로, 나머지 세 가지 조건을 방지하기 힘든 이유는 다음과 같다.

1. 상호 배제 예방: 상호 배제를 없애면 모든 자원을 공유 가능하게되므로, 현실적으론 불가능하다.
2. 점유 대기 예방: 특정 프로세스에만 자원이 모두 할당되거나 자원이 필요한 프로세스가 계속해서 자원을 기다리는 경우가 발생할 수 있다.
3. 비선점 예방: 한 프로세스 작업이 끝날 때까지 다른 프로세스가 기다려야 하는 자원이 있을 수 있기 때문에 모든 자원에 대해 자원을 빼앗을 수 있게 하면 정상적인 작업이 진행되지 않는다.

하지만 원형 대기 조건은 자원을 할당할 때 순서를 정하여 순서대로 자원을 할당하면 원형 대기는 더이상 발생하지 않게 되면서 간단하게 해결할 수 있다.  
하지만 이 방법은 자원을 할당할 때 순서를 정해야 하므로, 이 작업에 대해 자원이 소모되고 순서를 할당하는 방법에 따라 자원을 사용하는 효율이 달라질 수 있다.

### 회피

교착 상태가 발생하지 않을 정도로만 자원을 할당하는 방식으로, 자원의 상태와 요청 상태를 고려하여 자원을 할당하는 방법이다.  
교착 상태가 발생하지 않고 자원을 할당하고 종료할 수 있는 상태를 안전 상태(Safe State)라고 하며, 그 반대로 교착 상태가 발생할 수 있는 상황을 위험 상태(Undesirable State)라고 한다.

### 회복

교착 상태를 하용하되 교착 상태가 발생하면 아래의 방식으로 회복하는 방법이다.

1. 선점을 통한 회복: 교착 상태가 해결될 때까지 다른 프로세스로부터 자원을 빼앗고 한 프로세스에 자원을 할당하여 회복
2. 프로세스 강제 종료를 통한 회복: 교착 상태에 놓인 프로세스를 모두 강제 종료하거나, 한 프로세스씩 강제 종료하여 회복

###### 참고자료

- [혼자 공부하는 컴퓨터 구조 + 운영체제 - 1:1 과외하듯 배우는 컴퓨터 공학 전공 지식 자습서](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=혼자+컴퓨터+구조&isbn=9791162243091&cipId=228751835%2C)