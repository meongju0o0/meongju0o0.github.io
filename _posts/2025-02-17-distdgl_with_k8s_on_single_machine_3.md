---
layout: single
title:  "싱글 머신에서의 K8s 기반 DistDGL 분산 학습 환경 구축 - 3편, Docker, K8s 설치 및 클러스터 구성"
categories: DistributedLearning
tag: [distributed_learning, docker, k8s, gnn, dgl]
toc: true
toc_sticky: true
author_profile: true
---

# Docker, K8s 설치 및 클러스터 구성
## 1. Docker 설치
### 1-1. Docker 패키지 저장소 추가 및 설치
- apt 패키지 매니저 버전 업데이트

```bash
sudo apt update && sudo apt upgrade -y
```

![docker_install_1](/images/2025-02-17-DistDGL_on_Docker_3/docker_install_1.png)

- GPG 키 추가에 필요한 패키지 설치

```bash
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
```

![docker_install_2](/images/2025-02-17-DistDGL_on_Docker_3/docker_install_2.png)

- Docker 공식 GPG 키 추가

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

![docker_install_3](/images/2025-02-17-DistDGL_on_Docker_3/docker_install_3.png)

- Docker 패키지 저장소 추가

```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

![docker_install_4](/images/2025-02-17-DistDGL_on_Docker_3/docker_install_4.png)

- Docker 패키지 설치

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

![docker_install_5](/images/2025-02-17-DistDGL_on_Docker_3/docker_install_5.png)

### 1-2. Docker 실행 및 확인
- Docker 서비스 활성화

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

- Docker 설치 확인

```bash
docker --version
docker run hello-world
```

![docker_version_chk](/images/2025-02-17-DistDGL_on_Docker_3/docker_version_chk.png)

![docker_hello_world](/images/2025-02-17-DistDGL_on_Docker_3/docker_hello_world.png)

## 2. K8s 설치
### 2-1. K8s 패키지 저장소 추가 및 설치
- K8s 패키지 저장소 추가

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /usr/share/keyrings/kubernetes-archive-keyring.gpg
```

```bash
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list > /dev/null
```

![K8s_install_1](/images/2025-02-17-DistDGL_on_Docker_3/k8s_install_1.png)

- K8s 패키지 설치

```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
```

![K8s_install_2](/images/2025-02-17-DistDGL_on_Docker_3/k8s_install_2.png)

- K8s 자동 업데이트 방지

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

![K8s_install_3](/images/2025-02-17-DistDGL_on_Docker_3/k8s_install_3.png)


- K8s 설치 확인

```bash
kubectl version --client && kubeadm version
```

![K8s_install_4](/images/2025-02-17-DistDGL_on_Docker_3/k8s_install_4.png)