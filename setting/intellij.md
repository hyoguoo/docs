---
layout: editorial
---

# IntelliJ

## 라이브 템플릿 생성

라이브 템플릿은 자주 사용하는 코드 조각을 미리 지정한 단축어(Abbreviation)로 쉽게 입력할 수 있게 돕는 기능이다.

- 설정 경로: `Settings > Editor > Live Templates`
- 우측의 `+` 버튼을 눌러 새로운 템플릿이나 그룹을 추가 가능
- 주요 항목
    - `Abbreviation`: 템플릿을 호출할 때 사용할 단축어 지정
    - `Description`: 템플릿에 대한 설명 작성
    - `Template text`: 실제로 생성될 코드 내용 작성
        - 변수를 사용하고 싶을 경우 `$VARIABLE_NAME$` 형식으로 지정
    - `Applicable in`: 템플릿이 활성화될 언어와 코드 영역(예: 주석, 문자열 등) 정의
    - `Edit variables`: `Template text`에서 사용한 변수(`$VARIABLE_NAME$`)의 기본값이나 동적으로 생성될 값을 설정
