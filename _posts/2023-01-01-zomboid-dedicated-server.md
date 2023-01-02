---
title: "좀보이드 서버를 열어보자!"
categories:
  - 게임
tags:
  - zomboid-dedicated-server
  - docker
toc: true
toc_sticky: true
---

## 사건의 시작

요즘 주위에서 좀보이드라는 게임을 재밌게 하는거같아서 어떤지 물어봤다.

![발단1]({{ "/assets/images/2023-01-02/발단1.JPG" | relative_url}})

<center style="font-size: 14px; padding-bottom: 14px">4명 번들로 샀는데 자리가 하나 남는덴다</center>

![발단2]({{ "/assets/images/2023-01-02/발단2.JPG" | relative_url}})

<center style="font-size: 14px; padding-bottom: 14px">고심끝에 구매를 결정했다</center>

혼자해봤더니 생각보다 재밌다.  
그래서 친구가 열어둔 서버에서 플레이하는데, 아니 너무나 불편한 부분이 보였다.

![발단3]({{ "/assets/images/2023-01-02/발단3.png" | relative_url}})

아니... 호스트가 직접 열어서 초대해야 들어갈 수 있고, 호스트가 없을땐 못들어가는게 말이되나?  
당장 이 문제때문에 난 24시간 돌아가는 개인서버를 써먹기로 했다.

그래서 좀보이드 서버 여는 방법을 검색해본 결과, zomboid-dedicated-server를 이용해서 언제든지 친구들이 입장가능한 서버를 열었다.

## zomboid-dedicated-server가 뭔데 십덕아

좀보이드 서버를 여는데 필요한 각종내용들을 쉽게 돌릴 수 있도록 docker image화 해둔 이미지 이다.

- <a href="https://github.com/Renegade-Master/zomboid-dedicated-server" target="_blank">zomboid-dedicated-server repo Link</a>

처음부터 직접 다만들려면 steamcli부터해서 온갖 귀찮은 일들이 있었지만, 이걸 발견하고 쉽게 해결했다.

상세하게 사용하는 설명은 위의 링크에 있다.

## 서버 설치 & 열기

(이하의 내용은 linux와 Docker에 대한 지식이 없다면 이해하기 힘들다.)

docker가 설치되어있는 linux환경에서 시작한다.

### 소스코드 받기 & 설치

```bash
> git clone https://github.com/Renegade-Master/zomboid-dedicated-server.git // 서버 소스코드를 가져온다.
> cd zomboid-dedicated-server // 받은 소스코드로 이동
> docker-compose up -d // 소스에있는 docker-compose 스크립트를 이용하여 컨테이너 구축
```

정상적으로 설치가 됐다면 소스코드에 아래의 두 디렉토리들이 생겼을것이다.

- ZomboidConfig : 서버설정 및 서버데이터 저장 장소
- ZomboidDedicatedServer : 서버 구축에 필요한 에셋들의 리소스파일들

원래대로라면 nginx같은 웹서버로 외부 인터넷 요청을 해당 컨테이너의 열려있는 udp port로 transport해주는 과정이 필요하지만, 해당 컨테이너는 기본적으로 전체 아이피에 대해 열려있어 바로 받을수 있게 되어있다.

### 포트포워딩

이후 공유기에서 포트를 열어준다.

![공유기포트]({{ "/assets/images/2023-01-02/공유기포트.png" | relative_url}})

<center style="font-size: 14px; padding-bottom: 14px">default로 16261, 16262포트를 사용한다. 설정을 통해 바꿀수도 있다.</center>

여기까지 진행하면 일단 접속은 가능해진다.

### 접속방법

접속방법은 다음과 같다.

1. 메인화면 > 여러 명이서 하기 > 즐겨찾기추가
2. 내용작성
   서버이름: (맘대로)
   IP: (서버의 도메인 또는 아이피)
   LAN IP: (공백)
   포트: 16261
   서버 암호: (설정한 암호)
   서버 설명: (맘대로)
   유저 이름 : (맘대로)
   비밀번호: (맘대로)
3. 작성후 저장
4. 이후 생성된 서버목록의 서버를 더블클릭 or 접속하기 눌러서 접속

하지만 이후 서버의 맵 sandbox설정 또는 모드적용이 필요한데, 해당 기능을 이용하기 위해서는 docker-compose.yaml파일 등의 수정이 필요하다.

### 서버 세부설정 - 컨테이너

하나씩 알아보도록하자

```yaml
version: "3.8"

services:
  zomboid-dedicated-server:
    build:
      context: .
      dockerfile: docker/zomboid-dedicated-server.Dockerfile
    image: "docker.io/renegademaster/zomboid-dedicated-server:latest"
    container_name: zomboid-dedicated-server #컨테이너 이름
    restart: "no" #pc부팅시 자동시작여부, always면 부트되면서 자동으로켜진다.
    environment:
      - "ADMIN_USERNAME=${ADMIN_USERNAME}" #마스터계정 id
      - "ADMIN_PASSWORD=${ADMIN_PASSWORD}" #마스터계정 pw
      - "AUTOSAVE_INTERVAL=15"
      - "BIND_IP=${BIND_IP}" # 외부로 묶어주는 아이피. 기본으로 0.0.0.0으로 되어있으며, 이는 모든 아이피에대해 받는것을 의미한다.
      - "DEFAULT_PORT=16261" # 서버 통신시 사용하는 포트
      - "GAME_VERSION=public"
      - "GC_CONFIG=ZGC"
      - "MAP_NAMES=Muldraugh, KY"
      - "MAX_PLAYERS=16"
      - "MAX_RAM=8192m" #사용할 RAM 크기. 8기가는 쓰는게좋다
      - "MOD_NAMES=${MOD_NAMES}" # 서버에서 사용하는 모드들. 이는 후술한다
      - "MOD_WORKSHOP_IDS=${MOD_WORKSHOP_IDS}" # 서버에서 사용하는 모드에 대한 ID들.
      - "PAUSE_ON_EMPTY=true"
      - "PUBLIC_SERVER=true"
      - "RCON_PASSWORD=${RCON_PASSWORD}" #재접시 사용하는 PW. 써본적없긴함
      - "RCON_PORT=27015"
      - "SERVER_NAME=${SERVER_NAME}" #서버운영할 이름. 이 이름에 따라 적용되는 파일들이 달라진다. 기억해두자
      - "SERVER_PASSWORD=${SERVER_PASSWORD}" #서버 PW
      - "STEAM_VAC=true"
      - "UDP_PORT=16262"
      - "USE_STEAM=true"
    ports:
      - target: 16261
        published: 16261
        protocol: udp
      - target: 16262
        published: 16262
        protocol: udp
      - target: 27015
        published: 27015
        protocol: tcp
    volumes:
      - ./ZomboidDedicatedServer:/home/steam/ZomboidDedicatedServer
      - ./ZomboidConfig:/home/steam/Zomboid/
```

위에서 `${특정값}`으로 값을 넣어주고있는데, 편의상 환경변수를 사용한것이다(직접 작성해도 상관없음).

환경변수로써 사용하기위해서는 `.env`파일을 생성하고, 다음과 같이 쓰면된다.

```text
SERVER_NAME=test-server
SERVER_PASSWORD=p1a2s3s4word
...
```

### 서버 세부설정 - 모드

<a href="https://github.com/Renegade-Master/zomboid-dedicated-server" target="_blank">이 소스코드의 설명</a>을 보면 알겠지만, 모드를 적용하기 위해서는 모드명과 모드가 가진 숫자로된 고유 아이디 두가지가 필요하다.

예시로 넣으려면 이렇게 하면된다.

- Ex: <a href="https://steamcommunity.com/sharedfiles/filedetails/?id=2004998206" target="_blank">Minimal Display Bars</a>
- Ex: <a href="https://steamcommunity.com/sharedfiles/filedetails/?id=2619072426" target="_blank">Minimal Display Bars</a>

Minimal Display Bars, Weapon Condition Indicator두 모드를 넣으려면 각 링크 최하단의
`Workshop ID, Mod ID`를 확인한다.
그리고 아래와 같이 입력한다.

```text
(env)
MOD_NAMES='MinimalDisplayBars;TheStar'
MOD_WORKSHOP_IDS='2004998206;2619072426'
```

```yaml
#(yaml에서)
---
- "MOD_NAMES=MinimalDisplayBars;TheStar"
- "MOD_WORKSHOP_IDS=2004998206;2619072426"
```

여러개를 넣을때마다 그 사이를 `;`으로 구분하는것이 핵심이다.

### 서버 세부설정 - 센드박스 등 기타 설정

기타 설정을 건드리고 싶다면 `lua`나 `ini`파일을 수정해야한다.

이때, 중요한점은 위에서 yaml파일에서 선언한 `SERVER_NAME`의 파일을 수정해야 적용이 된다는 것이다.

![기타설정]({{ "/assets/images/2023-01-02/기타설정.png" | relative_url}})

- 샌드박스 설정파일 : {SERVER_NAME}\_SandboxBars.lua
- 기타 서버설정파일 : {SERVER_NAME}.ini

각각의 파일에 주석으로 사용가능한 옵션과 설명이 상세하게 쓰여있으므로 그것을 참고하는것이 더 좋다.  
(생각보다 모드가 아니라 sandbox, ini에서 건드리는 설정이 많다.)  
sandbox는 미니맵이나 다른유저 위치알림등의 편의기능, ini파일은 좀비 리스폰이나 날씨, 세부난이도 등의 설정이 위주이므로 찾아보자.

### git을 적용한 더 편한 서버관리

해당 내용들을 편하게 건드리고싶다면 github에 소스코드를 올려서 적용하는 방법이 편하다.  
gitignore등을 사용해서 편하게 관리하자.

```text
(.gitignore)
### User Additions ###
ZomboidConfig/backups/
ZomboidConfig/Saves/
ZomboidConfig/db/
ZomboidConfig/Logs/
ZomboidConfig/messaging/
ZomboidConfig/ip.txt
ZomboidConfig/server-console.txt
ZomboidDedicatedServer/
.env

...(이하 기존코드들)
```

기본적으로 `ZomboidConfig`, `ZomboidDedicatedServer`는 전체 ignore처리 되어있지만, 위처럼 설정하면 ZomboidConfig의 server데이터만 소스로서 관리가 가능하다.

## 마치며

docker에 대해 어느정도 안다면 쉽게 사용할 수 있도록 잘 만들어져있다.
다만 모드 버전관리이슈, ini 오버라이딩 이슈등의 해당 repo의 이슈들이 있으므로 참고해서 사용하자.
