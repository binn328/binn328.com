---
title: Window11 VM에 5500GT GPU 패스스루하기
description: Proxmox 서버에서 GPU를 Window 11 VM에 패스스루 하기
date: 2026-09-05
tags:
  - proxmox
  - passthrough
aliases:
draft: false
permalink:
---
# 개요 
이번에 갤럭시 폴드 8을 구매했다. 기존 태블릿으로 이용하던 Moonlight를 대체할 생각으로 구매했는데, 굉장히 만족스러운 모습을 보여주었다. 하지만 쓸 때마다 WOL로 데스크탑을 부팅시키는 과정이 귀찮아서... 이 참에 서버에 Window11을 올려두고 원할 때마다 접속할 수 있게 만들기로 하였다.
# 준비물
- [ ] [Window11 ISO](https://www.microsoft.com/ko-kr/software-download/windows11)
- [ ] [virtio-win ISO](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.285-1/)
- [ ] [Window 설치 사전 구성 ISO](https://schneegans.de/windows/unattend-generator/)
	- 윈도우 설치 시 가장 귀찮은 설정 부분을 건너뛸 수 있다.
	- 24시간 켜두기 위해서는 윈도우 업데이트를 비활성화해야하는데 여기서 설정할 수 있다.
	- 그 외 쓸모없이 깔리는 프로그램들도 설치되지 않게 만들 수 있다.

# 1. Window 11 VM 생성
![[Pasted image 20260905192726.png]]
준비물이 모두 준비되었다면 Proxmox 에서 VM 생성을 시작한다. 우선 VM의 이름을 정해준다.

![[Pasted image 20260905192817.png]]
준비해둔 Window 11 ISO를 선택하고, 유형은 `Microsot Windows`로, 그리고 VirtIO 드라이버용 드라이브를 추가한다. 이걸 안하면 윈도우를 설치할 디스크를 선택하는 단계에서 막히게 된다.

![[Pasted image 20260905192947.png]]
BIOS는 `OVMF (UEFI)`, `Qemu 에이전트` 활성화, 그리고 `EFI 스토리지`와 `TPM 스토리지`를 선택해준다.

![[Pasted image 20260905193058.png]]
 사용할 디스크의 크기를 정해준다. 그리고 `SSD 에물레이션`, `버리기`를 활성화한다.
 
![[Pasted image 20260905193218.png]]
적당한 갯수의 코어를 주고, CPU 유형을 `host`로 설정한다. 문라이트 사용을 위해 GPU를 패스스루하려면 필수적이다.

![[Pasted image 20260905193335.png]]
메모리는 최소 `8GB` 이상은 주는 것이 좋다. 

![[Pasted image 20260905193416.png]]
네트워크 부분은 그대로 두고

![[Pasted image 20260905193441.png]]
설정값을 확인하고 생성하면 된다. 단, 아직 설정할 것이 남았으니 `생성 후 시작`은 비활성화한다.

![[Pasted image 20260905193550.png]]
이후 생성된 VM의 `하드웨어` 설정에서 준비해둔 `Window 사전 구성 ISO`를 추가해준다. 그리고 VM을 시작하고 Window 11의 설치를 진행하면 된다.

# 2. Window 11 설치
![[Pasted image 20260905193800.png]]
VM을 시작하고 Window11을 설치하려고 보면 아까 추가한 드라이브가 표시되지 않는다. 당황하지 말고 `드라이버 로드(L)`을 선택한다.

![[Pasted image 20260905193839.png]]
`찾아보기`를 누르고 `virtio-win/amd64/w11`을 선택하고 확인을 누른다.

![[Pasted image 20260905193912.png]]
표시된 드라이버를 선택하고 `설치`를 누른다.

![[Pasted image 20260905193936.png]]
이제 드라이브를 선택하고 `다음`을 누르면 된다.

![[Pasted image 20260905194000.png]]
아까 `Window 설치 사전구성 ISO`를 넣어두었다면 바로 Window 설치 화면으로 넘어가고 기다리면 된다. 굳이 국가는 어디고 사용자명은 뭐고 설정할 필요가 없어서 굉장히 편하다.

설치가 끝나면 우선 VM은 종료한다. 이제 GPU 패스스루를 해야 한다.

# 3. GPU 패스스루
우선 GPU 패스스루는 [이 블로그](https://god-logger.tistory.com/188)의 가이드를 많이 참고했다. 자세한 가이드 올려주셔서 감사합니다, 덕분에 3번의 삽질을 딛고 성공할 수 있었어요.

일단 GPU 패스스루가 없어도 Moonlight를 설정하고 사용할 수는 있다. 하지만 GPU 가속이 불가능하기 때문에 **소프트웨어 인코딩**만 사용할 수 있으며, 이로 인해 **인코딩 성능 문제**와 **60FPS 이상을 지원하지 않는 문제**가 생긴다. 즉, 클라이언트 측에서 화면이 버벅인다는 느낌을 받게 된다.

그렇기 때문에 현재 서버에서 GPU를 다른 용도로 사용 중인 것이 아니라면, 가급적 GPU를 VM에 넘겨주는 쪽이 사용성에 있어서 압도적으로 좋다.

그러니 시작해보자.

## 1) Proxmox 부팅 옵션 설정
`GRUB` 설정 파일을 연다.
```bash
nano /etc/default/grub
```

아마 건드린 적이 없다면 아래와 같이 적혀있을 것이다.
```bash title="/etc/default/grub"
GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR=`( . /etc/os-release && echo ${NAME} )`
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX=""
```

`GRUB_CMDLINE_LINUX_DEFAULT` 부분을 다음과 같이 변경한다.
```bash title="/etc/default/grub"
GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR=`( . /etc/os-release && echo ${NAME} )`
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt video=efifb:off,vesafb:off initcall_blacklist=sysfb_init pcie_acs_override=downstream,multifunction"
GRUB_CMDLINE_LINUX=""
```

> [!info] 각 옵션의 의미
> - `amd_iommu=on`: AMD 프로세서의 하드웨어 가상화 IOMMU(AMD-Vi) 기능을 활성화한다.
> - `iommu=pt`: IOMMU를 **PassThrough** 모드로 구동한다. 하이퍼바이저가 직접 사용하는 장치는 IOMMU 변환을 건너뛰게 만들어 호스트의 입출력 오버헤드를 줄이고 성능을 보장한다. VM에 할당된 장치만 IOMMU 격리를 거친다.
> - `video=efifb:off,vesafb:off`: 구형 EFI 및 VESA 기반 프레임버퍼 드라이버를 비활성화하여, 호스트 콘솔이 그래픽 카드를 점유하지 못하도록 막는다.
> - `initcall_blacklist=sysfb_init`: 최신 리눅스 커널에서 EFI 프레임버퍼를 대신해 하드웨어를 선점하는 sysfb 초기화 함수를 차단한다.
> - `pcie_acs_override=downstream,multifunction`: 메인보드나 칩셋 레벨에서 IOMMU 그룹이 여러장치와 뭉쳐있을 때, Access Control Service 검사를 소프트웨어적으로 우회하여, 강제로 각 Pcie 슬롯 및 다기능 장치를 개별 IOMMU 그룹으로 쪼갠다.

추가를 마치면 `grub`을 업데이트한다.
```bash
update-grub
```

## 2) 드라이버 블랙리스트 추가
내장 그래픽 카드와 오디오 드라이버를 호스트가 사용하지 않도록 차단한다. 만약 설정하지 않으면 악명 높은 `-43 에러`를 마주하게 된다.

```bash
# 해당 파일을 편집기로 열어서
nano /etc/modprobe.d/blacklist.conf
```

```bash title="/etc/modprobe.d/blacklist.conf"
# 다음 내용을 추가해준다.
blacklist amdgpu
blacklist snd_hda_intel
blacklist snd_hda_codec_hdmi
```

## 3) vfio 모듈 추가
장치의 제어권을 VM에 넘기기 위해 `vifo` 모듈을 추가한다.

```bash
nano /etc/modules
```

```bash title="/etc/modules"
vendor-reset
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

`vendor-reset`은 AMD 그래픽카드의 악명높은 PCIe Reset 버그(FLR 버그)를 해결하기 위해 추후에 추가할 커널 모듈이다. `vfio`보다 먼저 불러와야 하므로 **반드시 제일 위에 적어줘야 한다**.

## 4) GPU ID 확인
패스스루할 장치들의 ID값을 가져와야한다.

```bash
lspci -D -nnk | grep "AMD/ATI"
```

```bash title="실행 결과"
0000:03:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Cezanne [Radeon Vega Series / Radeon Vega Mobile Series] [1002:1638] (rev c9)
        Subsystem: Advanced Micro Devices, Inc. [AMD/ATI] Device [1002:1636]
0000:03:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Renoir Radeon High Definition Audio Controller [1002:1637]
        Subsystem: Advanced Micro Devices, Inc. [AMD/ATI] Renoir Radeon High Definition Audio Controller [1002:1637]
```

현재 사용 중인 CPU는 `5500GT`이다. 다른 CPU를 사용 중이라면 다르게 나올 것이다. 본인에게 맞춰 다음과 같이 기록해두도록 하자.

|      장치      | IOMMU Group  |    ID     |
| :----------: | :----------: | :-------: |
|     VGA      | 0000:03:00.0 | 1002:1638 |
| Audio Device | 0000:03:00.1 | 1002:1637 |
오디오 장치는 왜? 라고 생각할 수도 있다. 우리는 현재 VM의 바이오스를 `UEFI`로 설정했기 때문에 GPU만 패스스루를 시키면 `-43 에러`를 마주하게 된다. 이를 해결하는 방법이 오디오 장치를 함께 넘겨주는 것이기 때문에 오디오 장치도 기록해두는 것이다. ([출처](https://github.com/isc30/ryzen-gpu-passthrough-proxmox#optional-getting-ovmf-uefi-bios-working-error-43))

## 5) VIFO 바인딩
 `vfio.conf` 파일을 수정해 장치를 바인딩 시켜준다.

```bash
nano /etc/modprobe.d/vfio.conf
```

```bash title="/etc/modprobe.d/vfio.conf"
# 기록해둔 GPU의 ID를 적어준다.
options vfio-pci ids=1002:1638
# 해당 장치의 절전 모드 전환 기능을 비활성화한다.
options vfio-pci disable_idle_d3=1
```

수정을 마친 후, 커널 이미지에 수정사항을 반영하고 재부팅한다.

```bash
update-initramfs -u -k all
reboot
```

## 6) vbios 추출
VM이 호스트가 부팅하면서 건드려 놓은 bios 롬 대신, 전용으로 사용할 수 있는 bios를 추출해 맵핑해주어야 한다. 역시 `-43 에러`와 `reset 버그`를 방지하기 위함이다.

vbios는 현재 사용 중인 메인보드의 바이오스에서 추출할 수 있다. 이를 위해 현재 바이오스 버전을 알아야한다.

```bash
dmidecode -t bios​
```

```bash title="실행 결과" {3}
BIOS Information
        Vendor: American Megatrends International, LLC.
        Version: P2.10
        Release Date: 11/21/2024
        Address: 0xF0000
        Runtime Size: 64 kB
        ROM Size: 16 MB
        Characteristics:
                PCI is supported
                ...
```

현재 X300을 사용 중인데, bios 버전은 `P2.10`이다. 

[X300의 지원 홈페이지](https://www.asrock.com/nettop/AMD/DeskMini%20X300%20Series/index.kr.asp#BIOS)에 들어가 `P2.10` 버전의 바이오스를 다운로드하여 윈도우 PC에 위치시킨다. 혹은, 아까 만들어둔 VM에서 진행하는 방법도 있다.

그리고 [UBU Tool](https://winraid.level1techs.com/t/tool-guide-news-uefi-bios-updater-ubu/30357)을 다운로드한다. 

![[Pasted image 20260905205255.png]]
다운로드한 압축 파일을 해제하고, 받아둔 `bios.bin` 파일을 `UBU` 폴더에 던져넣은 후 `UBU.cmd` 파일을 실행한다.

![[Pasted image 20260905205448.png]]
잠시 기다리면 로딩이 완료되고 메인 메뉴가 표시된다. 우리에게 필요한건 GPU 쪽이니 `2 - Video OnBoard`를 선택한다.

![[Pasted image 20260905205558.png]]
그리고 `X`를 눌러 추출을 시작한다.

![[Pasted image 20260905205650.png]]
추출이 완료되면 `Extracted` 폴더가 생성된다. 해당 폴더를 뒤져 `ADMGopDriver.efi`와 `vbios_1638.dat` 파일을 따로 바탕화면 같은 곳에 꺼내둔다. `efi` 파일을 `rom` 파일로 변환한 다음, 해당 두 파일을 Proxmox에 넣어줄 것이다.

## 7) AMDGopDriver.efi 파일을 Rom 파일로 변환
[edk2-BaseTools](https://github.com/tianocore/edk2-BaseTools-win32)를 이용해 해당 작업을 할 수 있다. 윈도우의 바탕화면에 터미널을 열고 아