# Intellij Setting

## 설정 백업

계정 Sync도 존재하지만 계정을 옮겨야하는 경우나 계정 Sync와는 다르게 구성 가능한 모든 요소를 백업할 수 있는 차이점이 있고, 더 명확히(?) 백업을 할 수 있는 장점이 존재한다.

### 준비물

- IntelliJ
- GitHub Repository(Private 추천)
- GitHub Account Password Token

### 방법

1. GitHub Repository 생성
2. `File` - `Settings Repository` -> 생성한 URL 입력
3. 선택
    - Overwrite Remote: Repositroy로 로컬 세팅 업로드
    - Overwrite Local: Repositroy의 세팅을 로컬에 적용
    - Merge: 서로 다른 두 세팅 Merge

## 개인적으로 사용하는 플러그인

- Atom Material Icons
- GitToolBox
- Key Prometer X
- Prettier
- Rainbow Brackets
- String Manipulation
- Tab Shifter

## 그 외 세팅

- 한글 자판 시 원화 `₩` 출력 방지

`DefaultkeyBinding.dict` 파일을 통한 \`키 변경을 하게 되면 IntelliJ 내부에서는 한글 자판에선 그대로 `₩`가 출력되는데,  
해당 파일을 사용하지 않고 `Karabiner-Elements`를 통해 키를 변경하면 한글 자판에서도 \`이 정상적으로 출력
![img.png](../image/karabiner.png)