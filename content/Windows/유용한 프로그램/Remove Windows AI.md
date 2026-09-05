---
title: Surface Pro 11에서 AI 기능 제거하기
description: Surface Pro 11에서 쓸모없는 기능을 제거합니다.
date: 2026-07-04
tags:
  - windows11
  - surface
  - ai
  - widget
aliases:
draft: false
permalink:
---
# 16GB로 버틸 수 없는 엄청난 메모리 사용량
올해 초에 서피스 프로 11을 구매하고, 이래저래 설정을 하며 사용을 해왔지만... 
![[스크린샷 2026-05-09 132802.png]]
32GB 모델을 샀어야 했나... 라는 생각이 들 정도의 끔찍한 메모리 사용량... 메인 데스크탑은 32GB로 사용 중이어도 이 정도는 아니었다. 때문에 Window가 제공하는 AI 기능이 문제라 진단을 내리고 AI 기능을 지워버리기로 했다. 어차피 코파일럿, 메모장에 달린 AI 기능, 그림판에 달린 기능 등등 나에겐 하등 쓸모없는 기능이다.
# RemoveWindowsAI
다행히 나만 그렇게 생각한 것이 아니었다. 여기저기 돌아다니다 보니, 이미 누가 개발해둔 저장소를 찾게 되었기 때문이다.

찾게 된 것은 바로 [RemoveWindowsAI](https://github.com/zoicware/RemoveWindowsAI). 윈도우에 존재하는 AI 기능들을 비활성화 및 삭제를 도와주는 스크립트이다. 아래와 같은 동작들 중, 본인이 원하는 것만 선택해 실행할 수가 있었다.
## 기능 목록
- **레지스트리 키 비활성화**
    - 코파일럿(Copilot) 비활성화
    - 리콜(Recall) 비활성화
    - 입력 인사이트 및 타이핑 데이터 수집 비활성화
    - 엣지(Edge) 내 코파일럿 비활성화
    - 그림판 내 AI 실험 프로그램 비활성화
    - AI 패브릭 서비스(AI Fabric Service) 제거
    - AI 액션 비활성화
    - 음성 액세스 비활성화
    - AI 음성 효과 비활성화
    - 설정 검색 내 AI 비활성화
    - 게이밍 코파일럿 비활성화
    - 모든 오피스 앱 내 코파일럿 비활성화
    - 사진 앱 내 AI 기능 비활성화
    - 캡처 도구의 'Click to Do' 비활성화
- **AI 패키지 재설치 방지**
    - CBS(구성 요소 기반 서비스) 저장소에서 AI 패키지가 다시 설치되지 않도록 커스텀 윈도우 업데이트 패키지를 설치합니다.
- **코파일럿 정책 비활성화**
    - `IntegratedServicesRegionPolicySet.json` 파일 내의 코파일럿 및 리콜 관련 정책을 비활성화합니다.
- **AI Appx 패키지 제거**
    - 제거 불가능(`Nonremovable`) 패키지와 윈도우 워크로드(WindowsWorkload)를 포함한 모든 AI Appx 패키지를 제거합니다.
- **리콜 선택적 기능 제거**
- **CBS 내 AI 패키지 제거**
    - CBS 저장소 내에 숨겨져 있거나 잠긴 AI 패키지들을 제거합니다.
- **AI 파일 제거**
    - 남은 모든 AI 설치 파일, 레지스트리 키, 패키지 파일들을 제거하여 전체 시스템을 정리합니다.
- **AI 구성 요소 숨기기**
    - 설정 페이지의 'AI 구성 요소' 항목을 숨깁니다.
- **메모장 AI 다시 쓰기 기능 비활성화**
- **윈도우 AI 작업 제거**
    - 윈도우 AI와 관련된 모든 예약 작업(Scheduled Tasks) 항목을 강제로 제거합니다.
- **업데이트 후 정리 확인**
    - 윈도우 업데이트 여부를 주기적으로 확인하여, 업데이트 후 새로 설치된 AI 기능들을 자동으로 다시 제거하는 예약 작업을 생성합니다.
- #### 클래식 앱 설치
    - AI 기능이 포함된 최신 앱을 구버전(클래식)으로 교체하는 옵션입니다.
    - **옵션**: 메모장, 그림판, 캡처 도구, 사진 뷰어 교체 및 구형 사진 앱(Photos Legacy) 설치
## 사용 방법
사용 방법은 무척이나 간단했따. 그냥 스크립트를 실행하고, 원하는 동작을 선택하고 작업 시작을 누르면 끝이다. 저장소에서 제공하는 실행 스크립트는 다음과 같다.
```powershell
& ([scriptblock]::Create((irm "https://raw.githubusercontent.com/zoicware/RemoveWindowsAI/main/RemoveWindowsAi.ps1")))
```

해당 스크립트를 실행하면 아래와 같은 GUI가 표시된다. 원하는 것을 선택한 후 **Apply**를 눌러주면 스크립트가 실행되고, 작업이 끝나면 재부팅을 하라고 한다.

![[Pasted image 20260704143345.png]]

난 아래와 같은 설정으로 작업을 시작했다.

- [x] Disable Registry Keys
- [ ] Prevent AI Package Reinstall
- [x] Disable Copilot Policies
- [x] Remove AI Appx Packges
- [x] REmove Recall Optional Feature
- [x] Remove AI CBS Packages
- [x] Remove AI Files
- [x] Hide AI Components
- [x] Disable Notepad Rewrite
- [x] Remove WindowsAI Tasks
- [x] Update Cleanup Check
- [x] Install Classic Photoviewer
- [x] Install Classic Mspaint
- [ ] Install Classic SnippingTool
- [ ] Install Classic Notepad
- [x] Install Photos Legacy

## 적용 후 메모리 사용량 
작업이 끝나고 재부팅을 진행한 후, 메모리 사용량을 체크했다.

![[스크린샷 2026-07-04 121549.png]]

부팅 직후의 모습, 아까 전과 다르게 메모리가 굉장히 널널해졌다. 

![[Pasted image 20260704144018.png]]
평소에 사용하는 대로 브라우저, 옵시디언, 디스코드를 켠 후의 메모리 사용량이다. 절반 이상 차지하고 있지만, 이 정도면 사용하는데 문제는 없다.
# 마무리
사용하지도 않는 AI 기능을 제거하고, 서피스가 굉장히 가벼워졌다. AI 기능을 개발하는건 좋은데, 최소한 강제로 깔아두고 성능을 잡아먹는 행태는 좀 사라졌으면 좋겠다는 생각을 해본다. 