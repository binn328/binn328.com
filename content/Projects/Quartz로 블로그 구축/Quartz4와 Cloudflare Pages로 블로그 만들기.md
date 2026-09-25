---
title: Quartz4와 Cloudflare Pages로 블로그 만들기
description: 옵시디언을 이용한 블로그를 만들어봅시다.
date: 2025-07-08
tags:
  - service/quartz
  - cloudflare
aliases:
draft: false
permalink:
---
# 개요
기존에 집에 있는 서버에서 돌아가는 블로그를 가지고 있었다. 실시간으로 반영이 된다는 장점이 있기는 했지만, NFS와 연결되어 있고 따로 정적 호스팅 서비스를 사용하는 등 복잡도가 높았고, 자질구레한 문제점이 몇 개 존재했다. 

그러던 중 **Cloudflare**의 **Pages** 서비스를 이용하면 정적 호스팅에 한해서 무제한 대역폭을 제공한다는 이야기를 듣게 되었고, 이참에 옮겨보기로 했다.
# Cloudflare Pages
Cloudflare Pages는 개발자들이 웹사이트와 웹 애플리케이션을 빠르고 쉽게 배포하고 호스팅할 수 있도록 설계된 **프론트엔드 플랫폼**으로, Git 기반 워크플로우를 사용하여 정적 사이트 생성기(SSG), 단일 페이지 애플리케이션(SPA) 등 다양한 프론트엔드 기술을 지원하며, Cloudflare의 강력한 글로벌 엣지 네트워크를 활용하여 매우 빠른 성능을 제공하는 것이 특징이다.
## 요금제
![[Pasted image 20250708190106.png]]
무료 요금제는 다음과 같은 특징이 존재한다.
### 한 달에 빌드 횟수 500회 제한
**Cloudflare Pages**를 Github 저장소와 연동한 경우, 저장소에 커밋이 이루어지면 자동으로 커밋된 저장소를 가져와 새로 빌드를 시작한다. 블로그에 게시물을 하나 올릴 때마다 커밋을 한다고 가정하면, 한 달에 500개의 게시물을 올릴 수 있다는 이야기가 된다. 만약 게시물에 틀린 부분에 있어 수정하는 것을 게시물당 4번씩 한다고 가정해도 100개의 게시물을 올릴 수 있다. 하루에 글을 4개씩 올릴 정도로 부지런한 것이 아니라면 충분한 횟수이다.
### 무제한 대역폭
AWS에 서비스를 올리고 신경도 안 쓰고 있다가 무료 할당량이 모두 소진되어 요금 폭탄을 맞는 사례가 종종 있다. **Cloudflare Pages**는 **무제한 요청과 무제한 대역폭을 제공**하기에 요금폭탄 맞을 생각에 조마조마할 필요가 없다. 하지만 과도하게 사용하면 제제를 받을 수 있다고는 하니 동영상 스트리밍 사이트 같은 것을 만들 생각은 접어두도록 하자.
### 웹 Analytics 제공
**Cloudflare Pages**를 이용해 배포를 하고 나면 해당 페이지의 접속자 통계 보고서를 확인할 수 있다. 접속자의 국가는 어디인지, 어떤 페이지가 몇 번 조회되었는지, 페이지 로딩 속도는 어떤지 등, Google Analytics과 같은 것을 따로 설정할 필요가 없기에 매우 편리하다. 
# Quartz4
![[Pasted image 20250708193544.png]]
[Quartz4](https://quartz.jzhao.xyz)는 Obsidian 노트를 웹사이트로 쉽게 발행할 수 있도록 도와주는 빠르고 강력한 **정적 사이트 생성기**(Static Site Generator, SSG)이다. 특히 개인 지식 관리 시스템(PKM)인 옵시디언 사용자들 사이에서 "디지털 가든"이나 개인 블로그를 구축하는 데 많이 사용된다. 

[Obsidian Publish](https://obsidian.md/publish)라는 Obsidian 측에서 제공하는 호스팅 서비스도 존재하지만 유료라는 단점이 있고, 어차피 도메인 연결하려면 비용이 또 발생하는데 기왕이면 돈도 아끼고 구축해보는 재미도 있는 쪽이 개인적으로는 더 마음에 든다.
# Quartz4와 Cloudflare Pages로 블로그 만들기
## 1. Quartz4 설정
우선 호스팅할 서비스인 [Quartz4](https://quartz.jzhao.xyz/)를 먼저 구성해보자. 사실 Quartz4 홈페이지에 들어가면 잘 설명되어 있지만 영어로 되어있으니 따로 적어본다.
### 1) Quartz4 가져오기
**준비물**: [Git](https://git-scm.com/), [Node.js v22 이상](https://nodejs.org/ko)

우선 적당한 곳에 터미널을 열고 아래 커맨드를 입력해 **Quartz4**를 복제해온다.
```bash
# Quartz4를 복제해온다.
git clone https://github.com/jackyzha0/quartz.git
# 복제한 디렉터리로 이동하여
cd quartz
# 필요한 것을 설치하고
npm i
# 초기 설정을 한다.
npx quartz create
```
### 2) 게시글 작성하기
**Quartz4**에서 모든 콘텐츠는 `/content` 디렉터리에 담겨있어야 하며, 접속 시 첫 페이지는 `/content/index.md` 노트이다.
#### 문법
게시글을 작성하기 위한 노트에는 속성 값이 필요하다.
![[Pasted image 20250708204143.png]]
옵시디언에서 `Ctrl + :` 를 누르면 속성을 작성할 수 있다. 템플릿 플러그인을 이용해 미리 만들어두면 편하며, 각 속성의 역할은 아래와 같다.
- `title`: 페이지의 제목이다. 이 속성이 없다면 파일의 이름을 제목으로 사용한다.
- `description`: 링크 미리보기에서 표시되는 설명이다.
- `permalink`: 파일 경로가 달라져도 일정하게 유지되는 사용자 지정 URL이다.
- `aliases`: 이 노트의 별칭이다.
- `tags`: 이 노트에 대한 태그이다. 태그를 지정하면 나중에 태그별로 모아볼 수가 있기에 지정해주는 것이 좋다.
- `draft`: 이 노트의 공개 여부이다. `true`로 지정하면 숨겨지긴 하지만 파일 자체는 호스팅 서버에 올라가기 때문에 주의해야 한다.
- `date`: 메모가 게시된 날짜를 나타낸다. 

이 외에도 다양한 태그가 있으며, 전체 목록은 [Quartz4 - Front Matter](https://quartz.jzhao.xyz/plugins/Frontmatter) 페이지에서 확인할 수 있다.
### 3) 페이지 꾸미기
복제해온 디렉터리를 보면 `quartz.config.ts`와 `quartz.layout.ts` 파일을 확인할 수 있다. `config` 파일은 로케일, URL, 테마 등의 설정을 할 수 있으며, `layout` 파일은 페이지 레이아웃을 변경할 수 있다.
#### quartz.config.ts
나는 설정 파일에서 몇몇 부분만 수정했으므로 여기에 적혀있지 않은 내용은 [Quartz4 - Configuration](https://quartz.jzhao.xyz/configuration)에서 확인하는 것을 추천한다.

해당 파일을 열어보면 상단부에 아래와 같은 코드 블럭이 존재한다.
```ts
const config: QuartzConfig = {
  configuration: {
	pageTitle: "🪴 binn328",  // 페이지의 제목을 설정한다.
    pageTitleSuffix: "",   
    enableSPA: true,  // SPA의 활성화 여부이다. 기본은 true
    enablePopovers: true,  // 팝오버 미리보기를 활성화할지 여부이다. 기본은 true
    analytics: {  // 애널리스틱 제공자를 작성한다. 
      provider: null,  // 우리는 클라우드플레어가 알아서 해주니 필요 없다.
    },
    locale: "ko-KR",  // 페이지 로케일이다.
    baseUrl: "binn328.com",  // 사이트가 배포된 URL이다. 
    ignorePatterns: ["private", "templates", ".obsidian"],
    defaultDateType: "modified",
    theme: {
      fontOrigin: "googleFonts",
      cdnCaching: true,  // Google CDN을 이용해 글꼴을 캐시할지 설정한다.
// .......
```
#### quartz.layout.ts
나는 레이아웃에서 일부 구성만 변경했으므로 여기에 적혀있지 않은 내용은 [Quartz4 - Layout](https://quartz.jzhao.xyz/layout)에서 확인하는 것을 추천한다.

해당 파일을 열어보면 상단부에 아래와 같은 코드 블럭이 존재한다.
```ts
// components shared across all pages
export const sharedPageComponents: SharedLayout = {
  head: Component.Head(),
  header: [],
  afterBody: [],
  footer: Component.Footer({
    links: { // 여기를 변경하여 페이지 하단부에 바로가기 링크를 만들 수 있다.
      GitHub: "https://github.com/binn328",
      About: "https://binn328.com/about",
    },
  }),
}
//........
```
![[Pasted image 20250708205626.png]]
`links` 속성을 수정하여 하단부에 바로가기 링크를 작성할 수 있다. 기본적으로 **Quartz4**의 Github와 Discord 바로가기 링크가 존재한다. 나는 내 Github 링크와 About 페이지로 가는 링크로 수정해주었다.

해당 코드 아래에서는 레이아웃을 수정할 수 있는데, 나는 기본 레이아웃이 마음에 들기도 하고 모바일에서도 딱히 불편한 점이 없어서 그대로 사용하기로 하였다.
### 4) Github 저장소 설정하기
이제 Github 저장소에 동기화를 시켜보자. 
#### 저장소 생성 후 연결
우선 Github에서 저장소를 하나 만든다. 생성 시에 **README** 및 **gitignore**를 만들면 안된다.
![[Pasted image 20250708210819.png]]

저장소가 생성되었다면 아래 사진과 같은 링크를 얻을 수 있을 것이다.
![[Pasted image 20250708210902.png]]

해당 링크를 사용하여 아래 명령어를 `quartz` 디렉터리의 터미널에 입력한다.
```bash
# 추적 중인 모든 원격 저장소를 나열한다.
git remote -v
 
# 저장소를 origin으로 설정한다. REMOTE-URL에는 아까 복사한 링크를 넣는다.
git remote set-url origin REMOTE-URL
 
# 업데이트가 가능하도록 upstream 원격 저장소를 추가한다. 
git remote add upstream https://github.com/jackyzha0/quartz.git

# 최초로 Github 저장소에 push한다.
# --no-pull 옵션은 초기 동기화 시 원격 저장소의 변경 사항을 가져오지 않아 
# 로컬 변경 사항이 덮어씌워지는 것을 방지하는 역할을 한다.
npx quartz sync --no-pull
```
이제 Github 저장소와의 연결이 끝났다.
#### 초기 설정 이후 동기화 방법
이후 `content`를 작성하거나 설정을 변경한 후 동기화를 하고 싶다면 아래 명령어를 사용하면 된다.
```bash
npx quartz sync
```
### 5) Cloudflare Pages로 배포하기
이제 모든 준비가 끝났으니 배포하는 일만 남았다. 
#### Cloudflare Pages 생성하기
우선 클라우드플레어 계정이 있어야 한다, 없다면 가입하자.

![[Pasted image 20250708211842.png]]

클라우드플레어 대시보드 좌측 사이드바를 보면 **컴퓨팅(Workers) - Workers 및 Pages** 메뉴가 있다. 

![[Pasted image 20250708211933.png]]
해당 메뉴에 들어오면 **생성** 버튼을 찾을 수 있다.

![[Pasted image 20250708211957.png]]

**생성** 버튼을 누르면 표시되는 창에서 상단의 **Pages** 탭에 들어가 **기존 Git 리포지토리 가져오기**를 시작한다.

![[Pasted image 20250708212123.png]]

Github 계정을 연결하고 **아까 만든 저장소를 선택**해준다.

![[Pasted image 20250708212239.png]]

**아래를 참고하여 속성 값을 지정**해준다.

| 구성 옵션       | 값                                           |
| ----------- | ------------------------------------------- |
| 프로젝트 이름     | 페이지 이름 (프로젝트 이름이 주소에 반영됨)                   |
| 프로덕션 분기     | v4                                          |
| 프레임워크 사전 설정 | None                                        |
| 빌드 명령       | `git fetch --unshallow && npx quartz build` |
| 빌드 출력 디렉터리  | public                                      |

모두 완료했으면 **저장 및 배포**를 누르면 1분 내에 배포가 완료된다. 이후 표시된 URL을 통해 배포된 사이트에 접속할 수 있다!

게시글을 작성 후 `npx quartz sync`로 Github에 동기화시키면 **Cloudflare**가 자동으로 해당 커밋을 가져와 빌드 후 재배포하는 것을 확인할 수가 있다!
### 6) 도메인 연결하기
만약 자신의 도메인을 가지고 있다면 사이트에 연결을 해줄 수 있다.

#### 일반적인 경우

![[Pasted image 20250708213039.png]]

아까 배포된 페이지에서 **본인 프로젝트 이름을 클릭**한다. 파란 번개모양 아이콘이 있는 곳이다.

![[Pasted image 20250708213308.png]]

상단부를 보면 **사용자 설정 도메인** 탭이 있다. 

![[Pasted image 20250708213330.png]]

해당 탭에 들어오면 **사용자 설정 도메인 설정** 버튼이 존재한다.

![[Pasted image 20250708213414.png]]

**본인이 소유한 도메인, 또는 하위 도메인을 입력**한다. 만약 루트 도메인이 리버스프록시 등에 연결되어 있다면 하위 도메인을 사용해야 한다. 그렇지 않으면 A 태그가 변경되어 기존에 존재하던 서비스들에 연결할 수 없게 된다.

![[Pasted image 20250708213452.png]]

**Cloudflare** 에서 자동으로 DNS 레코드를 추가해준다.
#### 리버스프록시 등이 있음에도 루트 도메인을 사용하고 싶은 경우
나 같은 경우에는 루트 도메인에 A태그로 집에 있는 서버에서 돌고 있는 리버스프록시를 가리키도록 해두었기에 불가능했다. 

![[Pasted image 20250708215011.png]]

본인이 현재 사용 중인 리버스프록시에 **Redirect**를 추가해주는 방식으로 해결할 수 있다. 이렇게 하면 루트 도메인으로 접속 시 **Cloudflare Pages**에서 호스팅 중인 도메인으로 보내준다. 