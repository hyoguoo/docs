# brew

각종 커맨드라인 프로그램과 일반 애플리케이션을 설치할 수 있는 패키지 매니저로 업데이트나 삭제 등을 간편하게 할 수 있다.

## brew 설치

[brew](https://brew.sh) 공식 홈페이지에서 안내하는대로 설치(명령어 입력 후 일부 설치 과정에서 추가 명령어 입력이 필요할 수 있음)

## 설치할 수 있는 것

- brew: 개발 관련 패키지
- cask: 웹사이트에서 받을 수 있는 애플리케이션
- mas: 앱스토어에서 받을 수 있는 애플리케이션

### mas

일반 `brew`나 `cask`는 공식 홈페이지에서 검색하여 쉽게 설치할 수 있지만 AppStore에 등록된 id를 통해 설치해야한다.

```shell
mas install 441258766
```

위의 AppStore id를 확인하기 위해서는 AppStore 애플리케이션 페이지의 url의 id를 확인하거나 mas 조회 명령어로 검색하여 확인할 수 있다.

```shell
mas search "Magnet"
```

## Brewfile

위의 설치할 수 있는 것들을 `Brewfile`에 명시 해두면 `brew bundle` 명령어로 한번에 설치할 수 있다.

```shell
brew bundle
```

```shell
# Brewfile
brew "git"
brew "git-lfs"

cask "discord"
cask "docker"
cask "eul"
cask "figma"
cask "google-chrome"
cask "iterm2"
cask "jetbrains-toolbox"
cask "karabiner-elements"
cask "keka"
cask "logi-options-plus"
cask "notion"
cask "postman"
cask "shottr"
cask "visual-studio-code"

mas "Aladin eBook", id: 1023251042
mas "Amphetamine", id: 937984704
mas "Cascadea", id: 1432182561
mas "HolaTranslator", id: 1518955356
mas "KakaoTalk", id: 869223134
mas "Keys Multiplatform", id: 1494642810
mas "Magnet", id: 441258766
mas "Paste", id: 967805235
mas "PrettyJSON for Safari", id: 1445328303
mas "Slack", id: 803453959
mas "Unicorn Blocker", id: 1231935892
``` 

반대로 현재 설치된 것들을 `brew bundle dump` 명령어를 입력하여 `Brewfile`로 추출할 수도 있다.