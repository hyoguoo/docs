---
layout: editorial
---

# Mac

## 메뉴 단축키 지정

- System Settings > Keyboard > Keyboard Shortcuts > App Shortcuts -> + -> `Application` - `Menu Title`
- 메뉴에 명시된 글씨와 동일하게 입력

![img.png](image/menu_shortcut.png)

## `Karabiner-Elements` 활용한 키 할당 변경

- path: /.config/karabiner/assets/complex_modifications

![img.png](image/karabiner.png)

### 한글 자판 시 원화 `₩` 출력 방지

시스템 전역으로 적용하게 되면 애플리케이션 내 창 전환이 안되는 문제가 있어서 IntelliJ에서만 적용되도록 설정  
(`DefaultkeyBinding.dict` 파일을 통한 \`키 변경을 하게 되면 IntelliJ 내부에서는 한글 자판에선 그대로 `₩`가 출력)

```json
{
  "title": "Change Won (₩) to grave accent (`)",
  "rules": [
    {
      "description": "Change Won (₩) to grave accent (`) in Korean layout.",
      "manipulators": [
        {
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
            },
            {
              "type": "input_source_if",
              "input_sources": [
                {
                  "language": "ko"
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

### COMMAND-H / COMMAND-M 비활성화

`to`에 아무런 동작을 하지 않도록 하여 `from`에 할당한 키를 눌렀을 때 아무런 동작이 없도록 설정

```json
{
  "title": "Disable Key",
  "rules": [
    {
      "description": "Disable COMMAND-M(Minimize)",
      "manipulators": [
        {
          "type": "basic",
          "description": "",
          "from": {
            "key_code": "m",
            "modifiers": {
              "mandatory": [
                "command"
              ]
            }
          }
        }
      ]
    },
    {
      "description": "Disable COMMAND-H(Hide Window)",
      "manipulators": [
        {
          "type": "basic",
          "description": "",
          "from": {
            "key_code": "h",
            "modifiers": {
              "mandatory": [
                "command"
              ]
            }
          }
        }
      ]
    }
  ]
}
```
