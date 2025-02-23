---
layout: single
title:  "싱글 머신에서의 K8s 기반 DistDGL 분산 학습 환경 구축 - 4편, K8s 클러스터 구성"
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

![swapoff](/images/2025-02-20-DistDGL_on_Docker_4/swapoff.png)

- `/etc/fstab` 파일 수정
- `/swap.img`로 시작하는 라인 주석 처리

```bash
sudo vi /etc/fstab
```

![swapoff_edit_1](/images/2025-02-20-DistDGL_on_Docker_4/swapoff_edit_1.png)

```
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-7iRSDgELhIC3qGrechDxHZm688AAiIJu6ZihYnbzIs7KtrinT2igDgTc2OtK4r5N / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/8a895e5b-79c6-4914-96bc-6aac65aa0eb8 /boot ext4 defaults 0 1
# /boot/efi was on /dev/sda1 during curtin installation
/dev/disk/by-uuid/A39C-96A5 /boot/efi vfat defaults 0 1
# /swap.img     none    swap    sw      0       0 # 주석처리!
```

![swapoff_edit_2](/images/2025-02-20-DistDGL_on_Docker_4/swapoff_edit_2.png)

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

![firewall_setting](/images/2025-02-20-DistDGL_on_Docker_4/firewall_setting.png)

## 3. containerd 설정 수정 (전체 노드)
### 3-1. containerd SystemdCgroup 설정 수정
- 기본 설정 파일 생성

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

![cri_activate_1](/images/2025-02-20-DistDGL_on_Docker_4/cri_activate_1.png)

- `config.toml` 수정

```bash
sudo vi /etc/containerd/config.toml
```

![cri_activate_2](/images/2025-02-20-DistDGL_on_Docker_4/cri_activate_2.png)

- `SystemdCgroup`을 `true`로 설정

```ini
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true
```

### 3-2. containerd 접근 권한 수정
- containerd-fix-permissions 서비스 정의 파일 생성

```bash
sudo vi /etc/systemd/system/containerd-fix-permissions.service
```

![systemd_service_file_gen](/images/2025-02-20-DistDGL_on_Docker_4/systemd_service_file_gen.png)

```
[Unit]
Description=Fix permissions for containerd.sock
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/bin/bash -c 'sleep 5; chmod 666 /var/run/containerd/containerd.sock'
ExecStartPost=/bin/bash -c 'chown root:docker /var/run/containerd/containerd.sock'
Type=oneshot
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

![containerd_fix_permissions](/images/2025-02-20-DistDGL_on_Docker_4/containerd_fix_permissions.png)

- `containerd-fix-permissions` 서비스 시작

```bash
sudo systemctl daemon-reload
sudo systemctl enable containerd-fix-permissions.service
```

- `containerd` 재시작

```bash
sudo systemctl restart containerd
sudo systemctl status containerd
```

![containerd_restart](/images/2025-02-20-DistDGL_on_Docker_4/containerd_restart.png)

- 시스템 재시작

```bash
sudo reboot now
```

![reboot](/images/2025-02-20-DistDGL_on_Docker_4/reboot.png)

## 4. K8s 마스터 노드 설정 (마스터 노드)
### 4-1. kubeadm을 활용한 클러스터 초기화
- kubeadm init

```bash
sudo kubeadm reset -f
sudo kubeadm init --pod-network-cidr=192.168.1.0/16
```

![kubeadm_init](/images/2025-02-20-DistDGL_on_Docker_4/kubeadm_init.png)

- 성공시, 아래와 같은 메시지 출력

```
Your Kubernetes control-plane has initialized successfully!
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
```

![kubeadm_init_success](/images/2025-02-20-DistDGL_on_Docker_4/kubeadm_init_success.png)

### 4-2. kubectl 설정

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

![kubectl_setting](/images/2025-02-20-DistDGL_on_Docker_4/kubectl_setting.png)

### 4-3. CNI(Container Network Interface, calico) 설치

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

![calico_install](/images/2025-02-20-DistDGL_on_Docker_4/calico_install.png)

### 4-4. CNI Pod 확인
- 현재 실행 중인 모든 Pod가 `Running`이어야 한다.

```bash
kubectl get pods -n kube-system
```

![kubectl_pod_chk](/images/2025-02-20-DistDGL_on_Docker_4/kubectl_cni_chk.png)

## 5. K8s 워커 노드 설정 (워커 노드)
### 5-1. 마스터 노드에서 토큰 생성
- 마스터 노드로의 워커 노드 조인을 위한 토큰을 생성한다

```bash
kubeadm token create --print-join-command
```

- 마스터 노드에서 생성한 join 명령어를 확인한다

- 생성된 join 명령어 예시는 아래와 같다

```bash
sudo kubeadm join 10.0.2.100:6443 --token ia6lda.jy9a25mml713f1be --discovery-token-ca-cert-hash sha256:a7fdc543810f3852ed2dfccd4315329a4615122014c76b0ae9618cc7d6bcafca
```

![kubeadm_token](/images/2025-02-20-DistDGL_on_Docker_4/kubeadm_token_create.png)

### 5-2. 워커 노드에서 join 명령어 실행
- 워커 노드에서 join 명령어를 실행한다

```bash
sudo kubeadm join 10.0.2.100:6443 --token ia6lda.jy9a25mml713f1be --discovery-token-ca-cert-hash sha256:a7fdc543810f3852ed2dfccd4315329a4615122014c76b0ae9618cc7d6bcafca
```

![kubeadm_join](/images/2025-02-20-DistDGL_on_Docker_4/kubeadm_join_1.png)
![kubeadm_join](/images/2025-02-20-DistDGL_on_Docker_4/kubeadm_join_2.png)

### 5-3. 마스터 노드에서 노드 목록 확인
- 마스터 노드로 돌아가서 워커 노드가 정상적으로 조인되었는지 확인한다.

```bash
kubectl get nodes
```

- 정상적으로 조인된 경우, 결과는 아래오 같다.

![kubectl_get_nodes](/images/2025-02-20-DistDGL_on_Docker_4/kubectl_get_nodes.png)