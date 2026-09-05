---
title: Window11 VM 생성하고 갤럭시 폴드 8에서 사용하기
description: Proxmox 서버에서 Window11을 설치하고, Moonlight를 이용해 폴드 8을 서피스 폴드8로 만들어봅시다.
date: 2026-09-05
tags:
  - proxmox
  - passthrough
  - galaxy_fold8
  - apollo
  - artemis
  - moonlight
  - windows11
  - 5500gt
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

# 3. GPU 패스스루 준비
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
[edk2-BaseTools](https://github.com/tianocore/edk2-BaseTools-win32)를 이용해 해당 작업을 할 수 있다. 윈도우의 바탕화면에 터미널을 열고 아래 명령어를 입력해 저장소를 복제해온다.

```bash
git clone https://github.com/tianocore/edk2-BaseTools-win32
cd edk2-BaseTools-win32
```

그리고 아까 기록해둔 `Audio Device`의 장치 ID를 가져와 다음 명령어를 입력해 `efi`파일을 `rom`파일로 변환한다.

```bash
# 참고로 1002는 Vendor ID, 1637은 Device ID이다.
EfiRom.exe -f 1002 -i 1637 -e AMDGopDriver.efi -o AMDGopDriver.rom
```

## 8) 파일을 Proxmox로 옮기기
이렇게 생성한 `vios_1638.dat` 파일과 `AMDGopDriver.rom` 파일을 Proxmox의 `/usr/share/kvm` 디렉터리로 옮겨준다. 나는 공유 폴더 하나를 `nfs`로 연결해 사용하고 있어서 여길 통해 파일을 옮겼다. 만약 그런 환경이 아니라면 `scp`를 이용하거나, 파일을 클라우드 스토리지에 업로드하고 다운로드 링크를 복사해 Proxmox 측에서 `wget`이나 `curl`을 통해 다운로드하는 방법이 있다.

다운로드를 끝내면 우선 윈도우 VM에 들어가 Apollo를 설치하자. GPU 패스스루를 진행하다보면 더 이상 Proxmox 콘솔에서 화면을 확인할 수가 없기 때문이다.

# 4. VM에 Apollo 설치
원격 제어 수단은 `RDP`, `Chrome Remote Desktop` 등으로 다양하지만, 나는 `Moonlight`, 그 중에서도 포크된 버전인 `Apollo`를 사용할 것이다. 이유는 설치만으로 가상 디스플레이 구성을 뚝딱 해주기 때문이다. 그 외에도 안드로이드에서 키보드 마우스를 사용하면 `win + tab` 등의 단축키가 안드로이드에서 사용되는 문제가 있는데, 이를 해결해준다거나, 화면 자동 회전 기능이 있다거나 등의 편의기능을 제공하기에 매우 편리하다.

![[Pasted image 20260905211716.png]]
윈도우 VM에서 [apollo 저장소](https://github.com/ClassicOldSong/Apollo/releases/tag/v0.4.6)에 접속해 설치를 진행한다.

![[Pasted image 20260905211834.png]]
설치를 마치면 우측 하단 트레이에서 하얀 톱니바퀴 모양의 Apollo를 우클릭해 `Open Apollo`를 선택한다.

![[Pasted image 20260905211925.png]]
본인이 대시보드에 로그인할 때 사용할 계정을 입력한다. 

![[Pasted image 20260905212004.png]]
생성을 마치면 잠시 후 로그인 페이지로 이동한다. 생성한 계정으로 로그인한다.

![[Pasted image 20260905212037.png]]
대시보드로 진입하면 우선 `Configuration`에서 `Locale`을 `한국어`로 변경해준다. 이후 하단에서 `Save`, `Apply`를 눌러 적용한다.

그리고 폴드 8에 `Apollo`의 짝꿍인 [Artemis](https://github.com/ClassicOldSong/moonlight-android)를 설치하고 앱을 열면 VM이 보일 것이다. (같은 네트워크에 연결되어 있어야 한다).

![[Pasted image 20260905212529.png]]
접속을 시도하면 4자리 숫자의 PIN이 표시된다. 상단의 `핀 - PIN 페어링`에 들어가 페어링을 해준다.

![[Pasted image 20260905212725.png]]
페어링 후 접속하면 폴드 8에서 보조 디스플레이처럼 동작하게 된다. 폴드 8이 사용 중이지 않은 디스플레이를 비활성화시킨다.

이제 Proxmox 콘솔에서는 화면을 확인할 수가 없으므로 VM 작업은 폴드 8에서 진행해야한다. 미리 배율 등을 설정해 작업하기 편하게 만들어주고 VM을 종료한다.
# 5. GPU 패스스루 진행
이제 GPU 패스스루를 마저 진행하면 된다.

## 1) PCI 디바이스 추가
![[Pasted image 20260905213058.png]]
우선 Proxmox 대시보드에서 VM의 `하드웨어` 메뉴에 들어가 `PCI 디바이스`를 추가해준다. 처음으로 추가할 것은 GPU, 아까 적어둔 IOMMU Group을 확인해 `Raw 디바이스`를 추가하고, `메인 GPU`, `ROM-Bar`, `PCI-Express`를 활성화하여 추가한다.

![[Pasted image 20260905213217.png]]
두 번째로 추가할 것은 `오디오 장치`이다. 역시 아까 적어둔 IOMMU Group을 확인해 `Raw 디바이스`를 추가하고 `ROM-Bar`, `PCI-Express`를 활성화한다. 당연하게도 `메인 GPU`는 활성화하지 않는다.

## 2) vm 설정 파일 수정
이제 VM의 대시보드에서 구성할 수 없는 설정을 수정해야한다.

```bash
# VM 설정을 편집기로 연다. VM ID 확인 후 진행한다.
nano /etc/pve/qemu-server/${VM번호}.conf
```


```bash title="/etc/pve/qemu-server/101.conf" {6, 9, 10}
agent: 1
balloon: 0
bios: ovmf
boot: order=scsi0;ide0;net0;ide1;ide2
cores: 6
cpu: host
efidisk0: local-btrfs:101/vm-101-disk-0.raw,efitype=4m,ms-cert=2023k,pre-enrolled-keys=1,size=528K
hostpci0: 0000:03:00.0,pcie=1,x-vga=1
hostpci1: 0000:03:00.1,pcie=1
ide2: ssd01:iso/virtio-win-0.1.302.iso,media=cdrom,size=856810K
machine: pc-q35-11.0+pve2
memory: 16386
meta: creation-qemu=11.0.3,ctime=1788320680
name: window11
numa: 0
onboot: 1
ostype: win11
scsi0: local-btrfs:101/vm-101-disk-1.raw,discard=on,iothread=1,size=256G,ssd=1
scsihw: virtio-scsi-single
sockets: 1
tpmstate0: local-btrfs:101/vm-101-disk-2.raw,size=4M,version=v2.0
```

```bash title="/etc/pve/qemu-server/101.conf" {6, 9, 10}
agent: 1
balloon: 0
bios: ovmf
boot: order=scsi0;ide0;net0;ide1;ide2
cores: 6
cpu: host,hidden=1
efidisk0: local-btrfs:101/vm-101-disk-0.raw,efitype=4m,ms-cert=2023k,pre-enrolled-keys=1,size=528K
hostpci0: 0000:03:00.0,pcie=1,romfile=vbios_1638.dat,x-vga=1
hostpci1: 0000:03:00.1,pcie=1,romfile=AMDGopDriver.rom
ide2: ssd01:iso/virtio-win-0.1.302.iso,media=cdrom,size=856810K
machine: pc-q35-11.0+pve2
memory: 16386
meta: creation-qemu=11.0.3,ctime=1788320680
name: window11
numa: 0
onboot: 1
ostype: win11
scsi0: local-btrfs:101/vm-101-disk-1.raw,discard=on,iothread=1,size=256G,ssd=1
scsihw: virtio-scsi-single
sockets: 1
tpmstate0: local-btrfs:101/vm-101-disk-2.raw,size=4M,version=v2.0
```

`cpu`에 `hidden=1` 옵션을, 패스스루한 pcie 장치에 롬 파일을 맵핑시켜준다.

그리고 VM을 부팅시켜 확인할 시간이다!
# 6. VM 확인
우선 VM에 들어가 [AMD 드라이버](https://www.amd.com/ko/support/download/drivers.html)를 설치해준다. 이후 재부팅을 하고 장치 관리자를 들어가본다.

![[Pasted image 20260905215228.png]]
이런 식으로 `AMD Radeon(TM) Graphics`가 오류없이 잡혀있다면 성공이다. 

![[Pasted image 20260905215339.png]]
하지만 이런 식으로 `-43 에러`가 표시되고 그래픽 카드가 제대로 잡히지 않았을 수도 있다. 이 경우, 우선 재부팅을 해본다. Window VM이 아닌, **Proxmox** 자체를 재부팅 시켜야 한다. 구성은 올바른데, AMD reset 버그 때문에 생긴 일시적인 현상일 수도 있기 때문이다. 

AMD Reset 버그는 VM이 재부팅되면 AMD 그래픽 카드도 초기화가 되어야 하지만, 버그로 인해 초기화가 되지 않아 VM이 장치를 불러올 수 없는 현상이다.

이걸로 해결이 되었다면 다행이지만, 일단 해결이 안 되었더라도 다음 단계를 진행하도록 하자. 오류없이 잡혀있더라도 다음 단계는 진행을 해야 한다. 그렇지 않으면 VM을 재부팅할 때마다 Proxmox 자체를 재부팅시켜야 그래픽 카드가 정상적으로 잡히는 끔찍한 일을 당하게 된다.

# 7. vendor-reset 패치
아까 전에 `/etc/modules` 파일을 수정할 때, 제일 위에 `vendor-reset`을 넣어두었다. 다시 한 번 강조하지만, **제일 위에 작성해야 한다**. [참고 링크](https://github.com/gnif/vendor-reset/issues/85) 
이제 해당 모듈을 실제로 적용해보자. `-43 에러`와 AMD Reset bug를 해결하기 위해선 반드시 필요하다.

```bash
uname -r
```
우선 현재 사용 중인 커널의 버전을 확인한다.

```bash
apt install pve-headers-${버전}
# ex) apt install pve-headers-7.0.14-4-pve
```
그리고 해당 버전의 커널 소스코드를 가져온다.  나는 현재 `7.0.14-4-pve` 버전을 사용하고 있다.

```bash
# 커널 빌드 도구를 설치한다.
apt install git dkms build-essential

# vendor-reset 모듈을 가져온다.
git clone https://github.com/gnif/vendor-reset
cd vendor-reset

# 커널 빌드 경로에 vendor-reset을 추가한다.
dkms add .

# 커널 빌드를 수행하고
dkms build vendor-reset/0.1.1 -k ${버전}
# ex) dkms build vendor-reset/0.1.1 -k 7.0.14-4-pve

# 빌드된 커널을 설치한다.
dkms install vendor-reset/0.1.1 -k ${버전}
# ex) dkms install vendor-reset/0.1.1 -k 7.0.14-4-pve

# 설치된 커널을 확인한다.
# 목록에 vendor-reset... 이 있다면 성공이다.
dkms status

# 커널 이미지에 수정사항을 반영한다.
update-initramfs -u 
```
커널을 빌드하고 설치한다.

```bash
# 커널 모듈 의존성을 갱신한다.
dpmod -a

# 수동으로 모듈을 로드해 오류가 없는지 확인한다.
modprobe vendor_reset

# 모듈이 로드되었는지 확인한다.
lsmod | grep vendor

# 이런 식으로 나온다면 성공이다.
# vendor_reset          114688  0

# 이제 재부팅한다.
reboot
```
모듈을 수동으로 테스트한 후, 재부팅한다.

![[Pasted image 20260905221239.png]]
재부팅 후 VM을 실행해 그래픽이 잘 잡혔는지 확인한다.

# 8. 소리 설정
오디오 컨트롤러까지 넘겼지만, 현재 윈도우에는 사용할 수 있는 장치가 없다고 표시될 것이다. 이걸 해결하는 방법은 2가지가 있다.
## 1) 가상 오디오 장치 추가하기 (비추천)
![[Pasted image 20260905222034.png]]
`하드웨어` 메뉴에서 오디오 디바이스를 추가해주는 방법이다. 하지만 추천하지는 않는데, 원격 접속 후 유튜브에서 노래를 하나 틀어보면 바로 알 수 있다. 소리가 중간중간 멈추거나, 퍽퍽 거리는 소리가 섞여 들린다. 테스트 용도로는 적합하지만, 실사용은 무리이다.

## 2) 스팀 설치하기
![[Pasted image 20260905222602.png]]
스팀은 설치하면 **SteamLink**에서 사용하기 위해 `Steam Streaming Speakers`를 소리 장치로 추가해준다. 그리고 **Moonlight**는 원격 시에 해당 장치를 이용해 오디오를 클라이언트에 전달할 수 있다. 가상으로 추가된 장치가 아니라, VM에 존재하는 가상 오디오 장치이기 때문에 아까처럼 퍽퍽 튀는 소리 없이, 실제 컴퓨터에서 들리는 소리처럼 매끄럽게 들린다. 비슷한 방법으로 `VB-CABLE`을 사용하는 방법도 있긴 한데, 스팀을 설치하는게 가장 구성이 쉽고 편리하다.

# 9. 이후 기타 설정
## 1) 초기설정 백업하기
![[Pasted image 20260905222837.png]]
우선 문제가 없는 것을 확인했다면, 그 즉시 백업을 하나 해두도록 하자. 나중에 만지다가 망가져도, 윈도우 포맷이 필요해도, 언제든 돌아올 수 있도록.

## 2) 윈도우 AI 기능 제거
[[Remove Windows AI |이전에 쓴 글]] 에서도 언급했지만, [RemoveWindowsAI](https://github.com/zoicware/RemoveWindowsAI)를 이용해 윈도우 11의 AI 기능들을 모두 제거하는 것을 추천한다. 메모리 금값 시대에 가뜩이나 모자란 VM의 램을 갉아먹는 AI 기능들은 하등 쓸모가 없다. 

## 3) Window 10 UI 복원
[Explorer Patcher](https://explorerpatcher.net/)를 통해 매우매우 불편한 윈도우 11의 UI를 윈도우 10의 것으로 되돌릴 수 있다.

![[Pasted image 20260905223311.png]]
가장 중요한 우클릭 메뉴부터, 작업 표시줄을 되돌리고

![[Pasted image 20260905223345.png]]
파일 탐색기의 상단 리본 메뉴도 윈도우 10의 것으로 변경할 수 있다. 

![[Pasted image 20260905223413.png]]
그 외에도 쓸모없는 단축키를 유용한 단축키로 변경하는 등, 다양한 편의기능이 있으니 꼭 사용하는 것을 추천한다.
## 4) 태블릿 웹페이지 이용하기
폴드8의 화면이 크다고는 하지만, 그래도 PC 버전 웹 페이지를 이용하다보면 불편한 점이 있다. 대표적인게 터치에 최적화가 되지 않은 UI가 많다는 점.

이 부분은 `User-Agent`를 변경해주면 해결할 수 있다. **Firefox**에서는 `User-Agent Switcher`라는 확장 프로그램을 이용할 수 있다.

![[Pasted image 20260905225336.png]]
유튜브의 경우 PC UI로는 이렇게 표시되지만

![[Pasted image 20260905225350.png]]
`User-Agent`를 `Android Tablet`으로 변경하면 이렇게 표시된다.

폴드 8의 외부 화면을 사용할 경우 `Android Phone`으로 설정하는 등, 상황에 맞춰 편리하게 이용할 수 있다.
# 마무리
![[20260905_181501 - 복사본 1.jpg]]
(~~S23FE 로 찍은거라 화질이 영 구리다~~)

이게 서피스 듀오가 꿈꾸던 미래가 아니었을까... 서피스 프로를 두 손으로 들고 사용하기는 솔직히 힘든데 이건 침대에서 사용해도 전혀 무리가 없어 매우 만족스럽다.