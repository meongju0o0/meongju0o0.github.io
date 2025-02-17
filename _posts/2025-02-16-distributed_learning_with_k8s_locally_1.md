---
layout: single
title:  "싱글 머신에서의 K8s 기반 DistDGL 분산 학습 환경 구축 - 1편, 가상머신 생성 및 Ubuntu 설치"
categories: DistributedLearning
tag: [distributed_learning, docker, k8s, gnn, dgl]
toc: true
toc_sticky: true
author_profile: true
---

# 가상머신 생성 및 Ubuntu 설치
## 1. Virtual Box 설치
- 아래 홈페이지에서 본인의 호스트 운영체제에 맞는 Virtual Box를 다운 받고 설치한다.
- [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)

![vbox_download](/images/2025-02-16-DistDGL_on_Docker_1/vbox_download.png)

## 2. Ubuntu-24.04 Server 다운로드
- 아래 홈페이지에서 Ubuntu-24.04 Server 이미지를 다운로드한다.
- [https://ubuntu.com/download/server#manual-install](https://ubuntu.com/download/server#manual-install)

![ubuntu_download](/images/2025-02-16-DistDGL_on_Docker_1/ubuntu_download.png)

## 2. Virtual Box 가상 머신 만들기
### 2-1. 개요
- 호스트 OS: Windows 11
- 게스트 OS: Ubuntu-24.04 LTS

- 사용할 가상 머신 수는 총 3개
- 마스터 노드 1개, 워커 노드 2개
    - k8s-master
    - k8s-node1
    - k8s-node2

### 2-1. 가상 머신 추가
![vbox_hello](/images/2025-02-16-DistDGL_on_Docker_1/vbox_hello.png)

### 2-2. 가상 머신 설정
- **이름 및 운영 체제**

![vm_make_1](/images/2025-02-16-DistDGL_on_Docker_1/vm_make_1.png)

- 포스트는 k8s-node3를 기준으로 수행
- 위 이미지에서 **이름** 부분을 적절히 수정하여 생성
- **ISO 이미지**를 다운받은 우분투 이미지로 선택
- **무인 설치 건너뛰기** 체크!
    - 체크 하지 않아도 상관은 없지만,, 어차피 작동하지 않는다.

- **하드웨어**

![vm_make_2](/images/2025-02-16-DistDGL_on_Docker_1/vm_make_2.png)

- **EFI 활성화** 체크
    - 필수는 아니지만, 호스트 시스템과의 하드웨어 드라이버 호환을 위해 권장
    - 호스트 시스템 환경에 따라 다르다
        - 호스트 시스템의 부팅 모드 및 파티션 형식이 UEFI+GPT인 경우 권장
        - 호스트 시스템의 부팅 모드 및 파티션 형식이 BIOS+MBR인 경우, **EFI 활성화** 체크 해제
        - Windows 11인 경우, UEFI 모드이면 무조건 GPT 파티션
    
    - 호스트 시스템 환경 확인
        - Windows (PowerShell)

        - **부팅 모드 확인(UEFI, BIOS)**
        ```powershell
        bcdedit /enum | findstr "path"
        ```
        - `\WINDOWS\system32\winload.efi` → UEFI 모드
        - `\WINDOWS\system32\winload.exe` → BIOS(Legacy) 모드

        - **파티션 형식 확인(GPT, MBR)**
        ```powershell
        diskpart
        list disk
        ```
        - GPT 디스크는 `GPT` 열에 * 표시, MBR은 빈칸

![host_env_chk_win](/images/2025-02-16-DistDGL_on_Docker_1/host_env_chk_windows.png)

## 3. 우분투 설치
### 3-1. 가상 머신 실행
- 설정을 완료한 후, 가상 머신을 실행한다.

![vm_run](/images/2025-02-16-DistDGL_on_Docker_1/vm_run.png)

### 3-2. 우분투 설치
- 언어는 English로 설정

![ubuntu_install_1](/images/2025-02-16-DistDGL_on_Docker_1/ubuntu_install_1.png)

- 업데이트는 하지 않는다.

![ubuntu_install_2](/images/2025-02-16-DistDGL_on_Docker_1/ubuntu_install_2.png)

- 기본 설정으로, 아래 화면이 나올 때까지 `Done` 엔터
- 아래 화면에서는 `Continue` 엔터

![ubuntu_install_10](/images/2025-02-16-DistDGL_on_Docker_1/ubuntu_install_10.png)

- 사용자 프로필은 목적에 따라 적절히 작성

![ubuntu_install_11](/images/2025-02-16-DistDGL_on_Docker_1/ubuntu_install_11.png)

- Ubuntu Pro는 skip

![ubuntu_install_12](/images/2025-02-16-DistDGL_on_Docker_1/ubuntu_install_12.png)

- OpenSSH server는 지금 설치하도록 하자

![ubuntu_install_13](/images/2025-02-16-DistDGL_on_Docker_1/ubuntu_install_13.png)

- 추가적인 소프트웨어나 패키지는 필요할 때 설치할 것이다.
- 모두 체크 해제

![ubuntu_install_14](/images/2025-02-16-DistDGL_on_Docker_1/ubuntu_install_14.png)

- 설치가 완료되면, `Reboot Now` 엔터

![ubuntu_install_15](/images/2025-02-16-DistDGL_on_Docker_1/ubuntu_install_15.png)

- 재부팅 후에, 아래와 같은 화면이 나오면 엔터 2번 입력하자.
- VirtualBox에서 자동으로 cdrom 마운트 해제를 수행한다.

![ubuntu_install_16](/images/2025-02-16-DistDGL_on_Docker_1/ubuntu_install_16.png)

- 유저명과 패스워드를 입력하고 로그인

![ubuntu_install_17](/images/2025-02-16-DistDGL_on_Docker_1/ubuntu_install_17.png)

- 가상머신 종료

```bash
shutdown -h now
```

![ubuntu_shutdown](/images/2025-02-16-DistDGL_on_Docker_1/ubuntu_shutdown.png)