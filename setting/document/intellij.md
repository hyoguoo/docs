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

## 개인적으로 사용하는 플러그인

- GitToolBox
- Key Prometer X
- String Manipulation
- Translation

## 그 외 세팅

- 한글 자판 시 원화 `₩` 출력 방지

`DefaultkeyBinding.dict` 파일을 통한 \`키 변경을 하게 되면 IntelliJ 내부에서는 한글 자판에선 그대로 `₩`가 출력되는데,  
해당 파일을 사용하지 않고 `Karabiner-Elements`를 통해 아래 파일을 생성하여 적용하면 IntelliJ 내부에서 ` 키를 사용할 수 있다.  
(시스템 전역으로 적용하게 되면 애플리케이션 내 창 전환이 안되는 문제가 있어서 IntelliJ에서만 적용되도록 설정)

```json
// path: /.config/karabiner/assets/complex_modifications
{
  "title": "Change Won (₩) to grave accent (`)",
  "rules": [
    {
      "description": "Change Won (₩) to grave accent (`) in Korean layout.",
      "manipulators": [
        {
          "conditions": [
            {
              "type": "input_source_if",
              "input_sources": [
                {
                  "language": "ko"
                }
              ]
            }
          ],
          "type": "basic",
          "from": {
            "key_code": "grave_accent_and_tilde",
            "modifiers": {
              "optional": [
                "any"
              ]
            }
          },
          "to": [
            {
              "key_code": "grave_accent_and_tilde",
              "modifiers": [
                "option"
              ]
            }
          ],
          "conditions": [
            {
              "type": "frontmost_application_if",
              "bundle_identifiers": [
                "^com\\.jetbrains\\.intellij$"
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

![img.png](../image/karabiner.png)