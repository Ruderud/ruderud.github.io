---
title: "yarn berry를 써보자!"
categories:
  - 개발
tags:
  - yarn
toc: true
toc_sticky: true
---

## 선요약

- (package.json이 없을때 이걸먼저)

```bash
> yarn init -2
```

- 이후 yarn berry 마이그레이션 과정

```bash
> yarn set version berry
```

- .yarnrc.yml 수정 (pnp모드)

```yml
yarnPath: .yarn/releases/yarn-3.3.0.cjs
nodeLinker: pnp
```

- 의존성 설치 및 sdk 설치 (vscode)

```bash
> yarn
> yarn dlx @yarnpkg/sdks vscode
```

<a href="#안되는데요-2022-12-19-기준">에러발생시 참조</a>

## yarn berry 도입배경

지금껏 무지성 npm만 사용해왔다. (yarn이 있는건 알지만, npm이 엄청 범용적이라.)  
하지만 얼마전, yarn을 써서 개발할 일이 있었고 이걸 왜쓰는지 궁금해졌다.  
찾다보니까 npm보다 효율적으로 모듈을 관리하기 위해 facebook이 만들었다고 한다.  
그래서 계속 읽다보니 비교적 최근, yarn berry라는것이 나왔는데 이게 좀 특이하게 모듈을 관리하더라.

- <a href="https://toss.tech/article/node-modules-and-yarn-berry" target="_blank">node_modules로부터 우리를 구원해 줄 Yarn Berry(Toss)</a>

구글에 `yarn berry`라고 검색하면 이걸 써야하는 이유랍시고 열심히 써놓은글들 개많고, 다 읽어봤다.  
공통점은 하나같이 다들 저 글 링크로 걸어놓고 node_modules가 태양이나 블랙홀보다 무겁다는 **"그 짤"**을 걸어놓는다는것.

어찌됐든 내용을 요약해보면,

- 로컬에서 너무나도 큰 용량을 차지하고 있는 node_modules를 없애고 최적화 가능. (Plug'n Play)
- 배포환경에서 의존성을 추가로 설치하는 과정을 제거하여, CI속도 개선 향상. (zero-install)

이 두가지 효능?이 있다는것이 주 내용이다.
하지만 마냥 좋다기엔 불편한점이 아직있다.

- yarn 사용이 익숙치 않은 협업환경일 경우.
- 최신 typescript를 쓰고있는경우.
- 기타 yarn berry의 업데이트 속도 이상의 최신 기능 사용시

우선 1의 이유는 당연히 못쓴다. 무지성 `> npm i ~ ` 이러기 바쁠텐데 어떻게 쓰게만드냐.  
실제로는 2,3의 문제가 제일 크다. 이는 <a href="#안되는데요-2022-12-19-기준">아래</a>에서 서술할예정.

개인적으로는 최신 TS문법을 좀 쓰고싶은 마음이 있는지라 yarn berry를 잘 쓸지는 모르겠다.  
버전이슈가 해결되기 전까지는 가능하면 yarn & node_modules로 사용할 예정.

## yarn 적용방법

웬만한건 공식문서선에서 다 해결된다. 안되면 지옥시작.

- <a href="https://yarnpkg.com/getting-started/install" target="_blank">yarn 공식문서 링크(install)</a>

저기서 `install`, `Editor SDK(vscode 세팅)`, `(Features) Plug'n Play` 이거 세개가 핵심이다.

처음 적용할때는 두가지 상황으로 나뉜다

1. 내가 처음부터 맨땅에 프로젝트를 만들때 (거의없음)
2. 남이 만들어준 프로젝트에 마이그레이션할때 (거의대부분)

### 맨땅에 프로젝트 만들때

#### 1. Berry를 사용하는 프로젝트 생성

```bash
> yarn init -2
```

이걸로 `package.json`을 초기화해서 만들고 시작하자.  
이후 과정은 아래의 마이그레이션 과정과 동일.

### 이미 만들어진 프로젝트를 마이그레이션 할 때

2의 경우는 이미 프로젝트에 `package.json`이 존재하는 CRA, VITE, NestJS, NextJS같은 프레임워크를 이용해서 프로젝트 생성을 하거나, git clone받아오는 상황이다.

#### 1. Berry 사용선언

```bash
> yarn set version berry
```

나는 이제부터 berry를 쓰겠다는것. (.yarnrc.yml생성)  
생성된 파일인 .yarnrc.yml을 수정하자.

#### 2. PnP 사용 선언

```yml
yarnPath: .yarn/releases/yarn-3.3.0.cjs
nodeLinker: pnp
```

`nodeLinker: pnp`을 추가해준다.  
(`nodeLinker: node_modules`는 node_moduels를 사용할때 입력)  
**꼭곢 저장을 하자!!!**

```bash
> yarn //or yarn install
```

package.json에 명시된 의존성들을 다운받는다.  
(zero-install일경우 이 과정이 필요없어진다. 나중에 설명)

#### 3. zipFS Extension 설치(vscode)

![zipFS]({{ "/assets/images/2022-12-19/zipFS.png" | relative_url}})

yarn berry에서는 모듈을 zip으로 관리하기때문에, 여기에 접근하기 위해서는 추가적으로 설치해야할 확장 프로그램이 필요하다.  
vscode Extensions에서 `zipFS`로 검색해서 설치하자.

#### 4. vscode sdk 설치

vscode에서 PnP & TypeScript를 쓰려할때 추가로 깔아야할 sdk가 있다.

```bash
> yarn dlx @yarnpkg/sdks vscode
//dlx는 npm에서 npx에 해당함. 모듈을 저장하지않고 즉시 사용
```

이러면 `.yarn/sdks`에 이것저것 설치가 된다.  
(만약, eslint & prettier도 있었다면 이것도 찾아서 sdk로 깔아준다. package.json을 참조하는 듯)

TS의 경우, 설치한 SDK버전으로 vscode에서 사용할지 물어보는 메시지가 뜰것이다.  
Allow해서 맞춰주자.  
(안뜰경우, `Cmd + Shift + P`로 타입스크립트 버전 선택이 가능하다.)

여기까지 하면 기본적으로 이제 작동은 된다. 하지만...

## 안되는데요? (2022-12-19 기준)

![없다는데요?]({{ "/assets/images/2022-12-19/없다는데요.jpg" | relative_url}})

<center style="font-size: 14px; padding-bottom: 20px">모듈이 어디있는지 몰?루</center>

불러온 모듈들을 인식하지 못한다.  
심지어 필요한 메서드를 자동으로 모듈에서 찾아와주는 개꿀기능도 작동하지 않는다.

타입 추론은 로컬 타입스크립트 서버가,  
설치 된 모듈을 찾는건 zipFS Extension가 담당하기에 이 두가지에서 문제가 있을거라는 **킹리적 갓심**이 든다.
다만 zipFS Extension은 내가 건드릴 수 있는 범주를 넘어서니, 타입스크립트에서 문제를 해결해보려 한다.

문제를 해결하기 위해 온갖 검색을 해본<a href="https://www.jadru.com/pnperror-solution-yarnberry" target="_blank">결과</a>,
yarn berry가 지원하는 Ts버전과, 실제 로컬에서 사용중인 Ts버전에서 문제가 있는것 같다.

비교적 안정적으로 작동하고있는 예전 버전을 사용하면 잘 작동하지 않을까?

![TS4.9.4]({{ "/assets/images/2022-12-19/ts4.9.4.png" | relative_url}})

현재 적용된 TS SDK버전은 최신버전인 4.9.4이다. 나는 4.4.4버전으로 많이 내려서 적용할 예정이다

### 1. TS버전 다운그레이드

```bash
> yarn remove typescript // ts 삭제
> rm -rf .yarn/sdks/typescript // 4.9.4 ts sdk 삭제
> yarn add -D typescript@4.4.4 // 개발 의존성으로 ts 4.4.4버전 설치
> yarn dlx @yarnpkg/sdks vscode // sdk 재설치
```

### 2. 다운그레이드 TS버전 적용

다운그레이드한 TS버전을 적용하자.
js파일을 아무거나 누르고 `Cmd + Shift + P`커맨드를 통해 타입스크립트 옵션을 설정할 수 있다.

![selectTsOption]({{ "/assets/images/2022-12-19/selectTsOption.png" | relative_url}})

<center style="font-size: 14px; padding-bottom: 14px">커맨드 입력시 옵션 창</center>
![selectSdkVersion]({{ "/assets/images/2022-12-19/selectSdkVersion.png" | relative_url}})
<center style="font-size: 14px; padding-bottom: 14px">설치된 sdk인 4.4.4버전 선택</center>

이제 정상적으로 사용가능하다.

![캬]({{ "/assets/images/2022-12-19/캬.png" | relative_url}})

<center style="font-size: 14px; padding-bottom: 14px">모듈 인식 및 메서드 추론이 정확히 되는모습</center>

## 마치며

사실은 배포환경에서의 zero-install도 다루고 싶었지만, sdk이슈로 인해 시간적 여유가 없어 다루지 못했다.  
이 sdk이슈는 작성하는 현재, yarn issue에서 TS 4.9.4 버전을 호환하도록 패치했다는 <a href="https://github.com/yarnpkg/berry/pull/5127" target="_blank">내용</a>을 확인하긴 했지만... 안되더라.
버전도 마이너버전으로 하나씩 내려가면서 확인을 해본결과 다음과 같다.

### TS 4.9.3 ~ 4.6.4 버전

![4.9.3-2]({{ "/assets/images/2022-12-19/4.9.3-2.png" | relative_url}})

<center style="font-size: 14px; padding-bottom: 14px">모듈 import시 인식하고, 모듈내부 메서드들을 잘 가져온다. 하지만...</center>
![4.9.3-1]({{ "/assets/images/2022-12-19/4.9.3-1.png" | relative_url}})
<center style="font-size: 14px; padding-bottom: 14px">모듈 import 선언 전에는, 그 모듈 내의 메서드들을 가져오지 못하고있다.</center>

### TS 4.5.5 이하 버전

![4.5.5]({{ "/assets/images/2022-12-19/4.5.5.png" | relative_url}})

<center style="font-size: 14px; padding-bottom: 14px">4.5.5버전부터는 모듈 인식 및 메서드 추론이 잘된다.</center>

테스트 결과 현재 불러온 모듈 자체를 인식하지 못하는 이슈는 4.9.4에만 존재하며, 4.9.3이하의 버전에서는 타입추론은 잘 된다.  
하지만 모듈 인식 이슈는 TS 4.6.4 버전 이상부터 존재하며, 4.5.5 버전 이하부터는 모듈 인식까지 정상 작동하는 모습을 보였다.
이는 zipFS Extension의 이슈로 보이며, extension의 패치를 기다려야할것...같다...

### TS 버전 정리

이를 정리한 버전별 테이블은 다음과 같다.

**2022-12-21 기준**

| TS 버전       | 모듈 추론 | 타입 추론 |
| ------------- | --------- | --------- |
| 4.9.4(latest) | ❌        | ❌        |
| 4.9.3         | ❌        | ✅        |
| 4.8.4         | ❌        | ✅        |
| 4.7.4         | ❌        | ✅        |
| 4.6.4         | ❌        | ✅        |
| 4.5.5         | ✅        | ✅        |
| 4.4.4         | ✅        | ✅        |

따라서, yarn berry & PnP 개발환경에서 TS적용시 추천하는 **버전은 4.5.5 버전** 이다.  
만약 최신 TS 버전을 사용해야할경우, 개발환경에서는 nodeLinker로 PnP가 아닌 node_modules를 사용하는 방법을 사용해야한다.
