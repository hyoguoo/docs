# Intellij Setting

## 설정 백업

계정 Sync도 존재하지만 계정을 옮겨야하는 경우에 사용할 수 있다.  
**`2022.3` 버전부터 `Settings Sync`가 정식 지원되면서 계정 이동을 하는 경우가 아니면 아래 방법을 사용할 필요는 없다.

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

## 라이브 템플릿 생성

`Setting` - `Edit` - `Live Templates` - `+`

- `Group`: 템플릿 그룹(실제 동작 범위와는 무관)
- `abbreviation` : 단축어
- `Template Text`: 템플릿 내용(변수는 `$VARIABLE$`로 표현)
- `Description`: 템플릿 설명
- `applicable`: 적용할 언어 및 범위
- `Edit Variables`: 템플릿 내용에서 사용할 변수 설정
