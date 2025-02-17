---
layout: single
title:  "싱글 머신에서의 K8s 기반 DistDGL 분산 학습 환경 구축 - 2편, NAT 네트워크 구축"
categories: DistributedLearning
tag: [distributed_learning, docker, k8s, gnn, dgl]
toc: true
toc_sticky: true
author_profile: true
---

# NAT 네트워크 구축
## 개요
- 아래와 같이 네트워크 환경을 구축할 것임

![net_diagram](/images/2025-02-16-DistDGL_on_Docker_2/network_diagram.png)

## 1. net-tools 설치
- `ifconfig`를 통한 네트워크 어댑터와 ip 파악을 위함

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install net-tools
```

![net_tools_install](/images/2025-02-16-DistDGL_on_Docker_2/net_tools_install.png)

## 2. VirtualBox NAT Network 구성
### 2-1. NAT Network란?
#### NAT(Network Address Translation)
- 사설 IP를 공인 IP로 변환해주는 주소 변환 서비스
- 라우터 등의 장비를 사용하여 다수의 사설 IP를 하나의 공인 IP 주소로 변환
- NAT Network를 구축하는 이유는 다음과 같다.
    - VirtualBox NAT:
        - Guest OS 간의 네트워크 통신 불가
        - Guest OS에서의 인터넷 접속 가능
        - Host OS에서의 포트포워딩을 통해 Guest OS 접속 가능
    - **VirtualBox NATNetwork**:
        - Guest OS 간의 네트워크 통신 가능
        - Guest OS에서의 인터넷 접속 가능
        - Host OS에서의 포트포워딩을 통해 Guest OS 접속 가능
        - 설정에 번거로움이 발생
    - VirtualBox Host-Only:
        - 인터넷 사용 불가
        - Guest OS 간의 네트워크 통신 가능
        - Host OS와 Guest OS 간 통신 가능
    - VirtualBox Internal Network:
        - VirtualBox 내부에서만 통신 가능
        - Host OS에서 접근 불가
    - Bridged Adapter:
        - 인터넷 사용 가능
        - Guest OS, Host OS 간 통신 가능
        - 실제 네트워크에 직접 연결되어 IP를 할당받는 방식
        - 실제 공유기 설정을 다루어야 하는 번거로움 발생

### 2-2. VirtualBox NATNetwork 구성
- `도구 > 네트워크` 이동

![vbox_net_setting_1](/images/2025-02-16-DistDGL_on_Docker_2/vbox_network_setting_1.png)

- `NAT 네트워크` 클릭
- `만들기` 클릭
- `IPv4 접두사`: `10.0.2.0/24`
- `DHCP 활성화` 해제
- `IPv6 활성화` 해제

![vbox_net_setting_2](/images/2025-02-16-DistDGL_on_Docker_2/vbox_network_setting_2.png)

- `설정` 클릭

![vbox_net_setting_3](/images/2025-02-16-DistDGL_on_Docker_2/vbox_network_setting_3.png)

- `Expert` 클릭
- 좌측 `네트워크` 클릭
- `어댑터 1`의 `다음에 연결됨`을 `NAT 네트워크`로 변경

![vbox_net_setting_4](/images/2025-02-16-DistDGL_on_Docker_2/vbox_network_setting_4.png)

## 3. Windows 인터넷 공유 설정 변경
- `win + R` → `실행`에서 `control` 입력

![windows_net_setting_1](/images/2025-02-16-DistDGL_on_Docker_2/windows_network_setting_1.png)

- 우측 상단 검색창에 `네트워크 연결 보기` 입력 및 이동

![windows_net_setting_2](/images/2025-02-16-DistDGL_on_Docker_2/windows_network_setting_2.png)

- 인터넷과 연결된 어댑터 우클릭 후, 속성 클릭

![windows_net_setting_3](/images/2025-02-16-DistDGL_on_Docker_2/windows_network_setting_3.png)

- `공유` 클릭 후, `다른 네트워크 사용자가 이 컴퓨터의 인터넷 연결을 통해 연결할 수 있도록 허용` 선택
- `홈 네트워킹 연결`은 VirtualBox 어댑터로 설정
    - 아래 예시에서는 `이더넷`이 VirtualBox 어댑터
    - 경우에 따라, `이더넷2`와 같은 이름일 수 있음
- `확인`

![windows_net_setting_4](/images/2025-02-16-DistDGL_on_Docker_2/windows_network_setting_4.png)

## 4. Ubuntu Network 설정
- 네트워크를 찾지 못해 부팅에 시간이 다소 소요됨
- 2분 정도 기다리자

![ubuntu_net_setting_1](/images/2025-02-16-DistDGL_on_Docker_2/ubuntu_network_setting_1.png)

- ifconfig를 통해 네트워크 인터페이스 파악

```bash
ifconfig
```

![ifconfig](/images/2025-02-16-DistDGL_on_Docker_2/ifconfig.png)

- vi 등의 편집기를 사용해 `/etc/netplan/00-installer-config.yaml`파일 수정
- **주의!: 경우에 따라, `enp0s3`가 아닌 `eth0`과 같은 이름일 수 있음**

![ubuntu_net_setting_3](/images/2025-02-16-DistDGL_on_Docker_2/ubuntu_network_setting_3.png)

```bash
sudo vi /etc/netplan/00-installer-config.yaml
```

```yaml
network:
  version: 2
    renderer: networkd
    ethernets:
      enp0s3: # 확인된 네트워크 인터페이스명 입력
        addresses: [10.0.2.102/24] # 여기서는 100을 Master Node, 101은 Worker Node 1, 102는 Worker Node 2로 사용
        routes:
          - to: default
            via: 10.0.2.1
        nameservers:
          addresses: [8.8.8.8, 8.8.4.4]
```

- nameserver(dns)는 아래 중, 마음에 드는 것을 선택
- 아무거나 상관 없다
    - Google: [8.8.8.8, 8.8.4.4]
    - SKT: [219.250.36.130, 210.220.163.82]
    - KT: [168.126.63.1, 168.126.63.2]
    - LG: [164.124.101.2, 203.248.252.2]

![ubuntu_net_setting_4](/images/2025-02-16-DistDGL_on_Docker_2/ubuntu_network_setting_4.png)

- `/etc/netplan/00-installer-config.yaml` 파일의 접근 권한 변경

```bash
sudo chmod 600 /etc/netplan/00-installer-config.yaml
```

- 변경한 설정 적용

```bash
sudo netplan apply
```

- 네트워크 테스트

```bash
ping google.com
ping 10.0.2.100 # Master Node의 설정을 완료했다면
ping 10.0.2.101 # Worker Ndoe1의 설정을 완료했다면
ping 10.0.2.102 # Worker Node2의 설정을 완료했다면
```

![ubuntu_net_setting_5](/images/2025-02-16-DistDGL_on_Docker_2/ubuntu_network_setting_5.png)

## 5. SSH 접속 설정 (Port Forwarding)
### 5-1. Port Forwarding
- SSH 접속 기본 포트는 22
- Host OS에서 Guest OS로 접속할 때 사용 가능한 IP는 `192.168.137.1` 뿐이다.
- Master Node로 접속할 때 사용할 SSH IP:Port: `192.168.137.1:20022`
- Worker Node1으로 접속할 때 사용할 SSH IP:Port: `192.168.137.1:20122`
- Worker Node2로 접속할 때 사용할 SSH IP:Port: `192.168.137.1:20222`

### 5-2. Virtual Box Port Forwarding Setting
- `도구 > 네트워크` 이동
- `속성 > NAT 네트워크 > 포트 포워딩` 이동
- 규칙 추가

![vbox_port_forwarding](/images/2025-02-16-DistDGL_on_Docker_2/vbox_port_forwarding_1.png)

- 프로토콜: TCP
- 호스트 IP: 빈칸
- 호스트 포트
    - Master Node: 20022
    - Worker Node1: 20122
    - Worker Node2: 20222
- 게스트 IP
    - Master Node: 10.0.2.100 (Master Node에 부여한 IP에 따라)
    - Worker Node1: 10.0.2.101 (Worker Node1에 부여한 IP에 따라)
    - Worker Node2: 10.0.2.102 (Worker Node2에 부여한 IP에 따라)
- 게스트 포트: 22

### 5-3. SSH Connection
- Virtual Box Ethernet Adapter IP 확인

![ssh_1](/images/2025-02-16-DistDGL_on_Docker_2/ssh_1.png)

- PowerShell 터미널 실행
- 아래 명령어 실행

```powershell
ssh vboxuser@192.168.137.1 -p 20022
```

![ssh_2](/images/2025-02-16-DistDGL_on_Docker_2/ssh_2.png)