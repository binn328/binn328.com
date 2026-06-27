---
title: Quartz4를 Quartz5로 마이그레이션하기
description: Quartz 5로 업데이트하기
date: 2026-06-27
tags:
  - service/quartz
aliases:
draft: false
permalink:
---
[[Quartz4와 Cloudflare Pages로 블로그 만들기|이전편]]

`Quartz 4`와 `Cloudflare pages`를 통해 구축해둔 블로그를 `Quartz 5`로 마이그레이션하는 방법을 소개합니다. [공식 가이드](https://quartz.jzhao.xyz/getting-started/migrating)를 그대로 따라갔으며, 혹시 처음부터 시작하신다면 [공식 시작 가이드](https://quartz.jzhao.xyz/getting-started/)를 참조하시길 바랍니다.

# 0. 기존 컨텐츠 백업하기
마이그레이션을 위해 브랜치를 `v4`에서 `v5`로 전환해야 합니다. 하지만 이 과정에서 기존 `content` 폴더가 유실되므로 다른 곳으로 백업해두어야 합니다.
# 1. v5 브랜치 가져오기
우선 자신의 저장소로 이동합니다. 만약 사용 중인 데스크탑에 해당 저장소가 없다면 `clone` 후 진행합니다.
```bash
# 공식 저장소를 upstream remote로 등록합니다.
git remote add upstream https://github.com/jackyzha0/quartz.git
 
# 공식 저장소에서 v5 브랜치 정보를 가져옵니다.
git fetch upstream v5
 
# v5 브랜치로 전환합니다.
git checkout -b v5 upstream/v5
 
# 종속성을 설치합니다.
npm i
 
# v5 브랜치를 자신의 저장소로 push 합니다.
git push -u origin v5
```
# 2. 사이트 설정하기
대화형 설정 도구를 이용해 사이트를 구성하고 기존 컨텐츠를 다시 가져옵니다.
```bash
npx quartz create
```
해당 명령어를 실행하면 다음과 같은 정보를 설정해주어야 합니다.
- 템플릿: `default`, `obsidian`, `ttrpg`, `blog` 중에 선택합니다. 우리는 `obsidian`을 사용하니 이것을 선택합니다.
- 기존 콘텐츠 경로: 0번 단계에서 백업해둔 콘텐츠의 절대 경로를 입력합니다.
설정이 끝나면 새롭게 생성된 설정 파일에서 사용된 모든 플러그인을 설치합니다.
```bash
npx quartz plugin install --from-config
# 만약 에러가 나면 아래 커맨드를 시도하면 됩니다. 저는 에러가 나서 이렇게 해결했습니다.
npx quartz plugin install --latest
```
# 3. 설정 파일 구성하기
`Quartz 5`가 되면서 기존 `quartz.config.ts`, `quartz.layout.ts`로 구성된 설정 파일이 `quartz.config.yaml` 하나로 합쳐졌습니다. 플러그인 설정도 모두 해당 파일로 옮겨져서 설정하기는 편해졌습니다만, 기존과 달라서 적응이 좀 필요합니다. 몇 가지 설정을 해보겠습니다.
## 1) Footer 설정하기
`Quartz 4`에서는 `Footer` 링크를 `quartz.layout.ts`에서 아래와 같이 설정했습니다.
```ts title="quartz.layout.ts" {9-10}
//...
// components shared across all pages
export const sharedPageComponents: SharedLayout = {
  head: Component.Head(),
  header: [],
  afterBody: [],
  footer: Component.Footer({
    links: {
      GitHub: "https://github.com/binn328",
      About: "https://binn328.com/about",
    },
  }),
}
//...
```
`Quartz 5`에서는 `quartz.config.yaml`에서 아래와 같이 설정할 수 있습니다.
```ts title="quartz.config.yaml" {6-7}
//...
- source: github:quartz-community/footer
    enabled: true
    options:
        links:
            GitHub: https://github.com/binn328
            About: https://binn328.com/about
//...
```
해당 구문에 대해 추가적인 설명을 하면 이렇습니다.
- `source`: 사용된 플러그인입니다. 파일을 살펴보면 여러 플러그인을 찾을 수 있습니다.
- `enabled`: 플러그인의 활성화 여부입니다. 사용하지 않는 플러그인은 해당 부분을 `false{:ts}`로 설정해주면 됩니다.
- `options`: 부가적인 플러그인의 설정을 지정하는 곳입니다. 
	- `links`: `footer` 에 들어갈 링크를 추가하는 곳입니다. 위 코드처럼 `보여질 이름: 주소` 형식으로 작성해주면 됩니다.

설정을 완료하면 아래 사진처럼 링크가 표시됩니다.
![[attachments/Pasted image 20260627200333.png]]
## 2) 테마 설정하기
`quartz-themes` 플러그인을 이용해 옵시디언에서 사용하던 테마를 `Quartz`에 적용할 수 있습니다. 하지만 저는 꺼둔 기능입니다. 기본값이 제일 예쁜 것 같더라구요. 

설정 방법은, 해당 플러그인을 활성화하고 사용할 테마를 지정해주면 됩니다.
```js title="quartz.config.yaml" {6, 8}
//...
- source:
    name: quartz-themes
    repo: github:saberzero1/quartz-themes
    subdir: plugin
  enabled: false // 사용하려면 true로 변경
  options:
    theme: things // things 테마를 사용합니다. 기본값은 default로 되어있습니다.
//...
```
사용 가능한 테마 목록은 [해당 플러그인의 저장소](https://github.com/saberzero1/quartz-themes#supported-themes) 에서 확인할 수 있습니다. 미리보기도 지원합니다.
## 3) 댓글 기능 추가하기
`Quartz`는 `github`의 [Giscus](https://github.com/apps/giscus)를 통해 댓글 기능을 지원합니다. 설정 방법도 그리 어렵지 않습니다.
### 요구 사항
댓글 기능을 활성화하려면 `Quartz` 저장소가 아래 요구사항을 충족해야합니다.
#### (1) 저장소가 공개 저장소여야 합니다.
저장소가 비공개면 방문자가 해당 저장소의 토론에 접근할 수가 없어서 댓글을 달거나 볼 수 없습니다.
#### (2) [Giscus](https://github.com/apps/giscus) 앱이 설치되어 있어야 합니다.
![[attachments/Pasted image 20260627202439.png]]
해당 앱을 설치하고, **Only select repositories** 옵션에서 `Quartz` 저장소를 지정해주면 됩니다.
#### (3) 저장소에 토론 기능이 활성화되어있어야 합니다.
저장소 - Settings - General - Features 에서 **Discussions**를 활성화해주면 됩니다.
![[attachments/Pasted image 20260627202641.png]]
### Giscus 설정하기
이제 [Giscus 설정 페이지](https://giscus.app/ko)에 접속해서 저장소 정보와 카테고리 정보를 입력하면 댓글 기능에 사용할 설정을 가져올 수 있습니다.
#### (1) 저장소 정보
`아이디/저장소이름` 형식으로 입력해주면 됩니다.
![[attachments/Pasted image 20260627202913.png]]
#### (2) Discussion 카테고리
`Announcements` 카테고리를 설정해주면 됩니다.
![[attachments/Pasted image 20260627202949.png]]
#### (3) 생성된 정보를 설정 파일에 반영하기
이제 아래로 내려가면 다음과 같은 정보를 얻을 수 있습니다.
![[attachments/Pasted image 20260627203240.png]]
해당 정보를 설정 파일에 반영해주면 끝입니다.
```yaml title="quartz.config.yaml" {3, 7-11}
//...
  - source: github:quartz-community/comments
    enabled: true
    options:
      provider: giscus
      options:
        repo: binn328/binn328.com
        repoId: //repo-id
        category: Announcements
        categoryId: //category-id
        lang: ko
    layout:
      position: afterBody
      priority: 10
//...
```
설정을 마치고 `npx quartz build --serve{:bash}` 명령어를 입력해 미리보기를 띄워보면 아래처럼 보입니다.
![[attachments/Pasted image 20260627203649.png]]

# 4. 기본 브랜치 변경 및 빌드 설정
설정을 마쳤으니 이제 반영할 시간입니다. 우선 저장소의 기본 브랜치를 변경해주어야 합니다.
![[attachments/Pasted image 20260627203931.png]]
저는 현재 변경을 해두어서 `v5` 브랜치가 **default**로 설정이 되어있습니다. 아마 변경한 적이 없다면 `v4`로 지정이 되어있을겁니다. 기본 브랜치는 *저장소 - Settings - General - Default branch* 에서 변경할 수 있습니다.
![[attachments/Pasted image 20260627204114.png]]

변경을 마쳤다면 이제 [Cloudflare](https://dash.cloudflare.com)의 Pages 탭에 들어가 설정을 만져줄 차례입니다. **빌드 구성**과 **분기 제어**를 건드려야합니다.
![[attachments/Pasted image 20260627204256.png]]

**빌드 구성**에서 빌드 명령을 아래와 같이 설정해줍니다.
```bash
git fetch --unshallow && npx quartz plugin install && npx quartz build
```
![[attachments/Pasted image 20260627204354.png]]**분기 제어**는 프로덕션 분기를 `v5`로 변경해줍니다.
![[attachments/Pasted image 20260627204501.png]]
여기까지 설정을 마쳤다면 마지막으로 저장소에 그동안 설정한 파일들을 반영해주면 끝입니다. 아래 명령어를 사용하면 됩니다.
```bash
npx quartz sync
```
해당 명령어를 사용하면 변경사항을 저장소에 반영하고, **Cloudflare**가 변경사항을 감지하고 갱신을 시작합니다.
![[Pasted image 20260627204819.png]]
# 마무리
![[Pasted image 20260627204909.png]]
짜잔, 마이그레이션과 설정을 완료했습니다. 하지만 매번 반영을 위해 터미널을 켜고 `npx quartz sync{:bash}`를 입력해주기는 솔직히 좀 많이 귀찮습니다. 그렇다고 변경사항을 감지할 때마다 자동으로 실행되게 하면 **Cloudflare Pages**의 500회 한계에 걸리게 되고요. 그래서 다음엔 `Cron`을 이용해 주기적으로 갱신을 시도하는 동작을 만들어보겠습니다. 

읽어주셔서 감사합니다.