---
layout: single
title:  "싱글 머신에서의 K8s 기반 DistDGL 분산 학습 환경 구축 - 3편, K8s 클러스터 구성"
categories: DistributedLearning
tag: [distributed_learning, docker, k8s, gnn, dgl]
toc: true
toc_sticky: true
author_profile: true
---

# K8s 클러스터 구성
## 1. Swap 비활성화 (전체 노드)
- K8s는 Swap 메모리가 활성화된 상태에서는 정상적으로 작동하지 않음

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

![swapoff](/images/2025-02-17-DistDGL_on_Docker_3/swapoff.png)

## 2. 방화벽 설정 (전체 노드)
- 방화벽 설정

```bash
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

![firewall_setting](/images/2025-02-17-DistDGL_on_Docker_3/firewall_setting.png)

## 3. K8s 마스터 노드 설정 (마스터 노드)
### 3-1. containerd 설정 수정
- 기본 설정 파일 생성

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

![cri_activate_1](/images/2025-02-17-DistDGL_on_Docker_3/cri_activate_1.png)

- `config.toml` 수정

```bash
sudo vi /etc/containerd/config.toml
```

![cri_activate_2](/images/2025-02-17-DistDGL_on_Docker_3/cri_activate_2.png)

- `SystemdCtroup`을 `true`로 설정

```ini
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true
```

- containerd 재시작

```bash
sudo systemctl restart containerd
sudo systemctl status containerd
```

![containerd_restart](/images/2025-02-17-DistDGL_on_Docker_3/containerd_restart.png)

### 3-2. kubeadm을 활용한 클러스터 초기화
- kubeadm init

```bash
sudo kubeadm reset -f
sudo kubeadm init --pod-network-cidr=192.168.1.0/16
```

![kubeadm_init](/images/2025-02-17-DistDGL_on_Docker_3/kubeadm_init.png)

- 성공시, 아래와 같은 메시지 출력

```
Your Kubernetes control-plane has initialized successfully!
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
```

![kubeadm_init_success](/images/2025-02-17-DistDGL_on_Docker_3/kubeadm_init_success.png)

### 3-3. kubectl 설정

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

![kubectl_setting](/images/2025-02-17-DistDGL_on_Docker_3/kubectl_setting.png)

### 3-4. CNI(Container Network Interface, calico) 설치

```bash
![calico_install](/images/2025-02-17-DistDGL_on_Docker_3/calico_install.png)
```

![calico_install](/images/2025-02-17-DistDGL_on_Docker_3/calico_install.png)

### 3-5. CNI Pod 확인

```bash
kubectl get pods -n kube-system
```

![kubectl_pod_chk](/images/2025-02-17-DistDGL_on_Docker_3/kubectl_cni_chk.png)