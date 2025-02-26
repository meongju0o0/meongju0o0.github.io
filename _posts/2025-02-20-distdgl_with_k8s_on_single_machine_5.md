---
layout: single
title:  "싱글 머신에서의 K8s 기반 DistDGL 분산 학습 환경 구축 - 5편, DistDGL 분산 학습"
categories: DistributedLearning
tag: [distributed_learning, docker, k8s, gnn, dgl]
toc: true
toc_sticky: true
author_profile: true
---

# K8s 클러스터에서의 DistDGL 분산학습
## 1. 단일 컨테이너에서의 DistDGL 학습
### 1-1. WSL2 환경 구축
- WSL2 Ubuntu 24.04 LTS 환경을 기반으로 단일 노드에서 작동하는 DistDGL dockerfile을 작성할 것이다
- 관련 문서는 아래 홈페이지를 참고하자
- [wsl2에-ubuntu-24-04-lts-설치하기](https://junorionblog.co.kr/wsl2에-ubuntu-24-04-lts-설치하기/)

### 1-2. WSL2에 docker 설치
- 아래 홈페이지를 참고하여 WSL2에 docker를 설치하자
- docker만 설치하면 된다
- [docker-설치](https://meongju0o0.github.io/distributedlearning/distdgl_with_k8s_on_single_machine_3/#1-docker-설치)

### 1-3. 소스코드 다운로드
- 깃 레포지토리를 클론한다

```bash
cd ~

git clone https://github.com/meongju0o0/distdgl-on-docker

cd distdgl-on-docker
```

![git_clone](/images/2025-02-26-DistDGL_on_Docker_5/git_clone.png)

### 1-4. 도커 이미지 빌드 및 컨테이너 실행
- 도커 이미지 빌드

```bash
docker build -t distdgl-cpu .
```

![docker_build](/images/2025-02-26-DistDGL_on_Docker_5/docker_build.png)

- 빌드된 이미지 실행

```bash
docker run -it --rm -p 2222:22 distdgl-cpu
```

![docker_run](/images/2025-02-26-DistDGL_on_Docker_5/docker_run.png)

### 1-5. 실행 중인 컨테이너로 SSH 접속 및 학습 수행
- WSL2 터미널 세션 추가 생성
- 컨테이너 SSH 접속
- 비밀번호: DistDGL

```bash
ssh root@localhost -p 2222
```

![container_ssh](/images/2025-02-26-DistDGL_on_Docker_5/container_ssh.png)

### 1-6. 그래프 데이터 다운로드 및 파티셔닝
- `/workspace/graphsage` 디렉토리로 이동

```bash
cd /workspace/graphsage
```

- 파티셔닝 및 그래프 메타데이터 생성

```bash
python partition_graph.py --dataset citeseer --num_parts 1 --part_method metis --balance_train --balance_edges
```

![partition_graph](/images/2025-02-26-DistDGL_on_Docker_5/partition_graph.png)

### 1-7. DistDGL 기반 GraphSAGE 학습 수행
- `/workspace` 디렉토리로 이동

```bash
cd /workspace
```

- 학습 수행

```bash
python launch.py --workspace /workspace/graphsage/ --num_trainers 1 --num_samplers 0 --num_servers 1 --part_config data/citeseer.json --ip_config ip_config.txt "python node_classification.py --graph_name citeseer --ip_config ip_config.txt --num_epochs 70"
```

![node_classification](/images/2025-02-26-DistDGL_on_Docker_5/node_classification.png)

## 2. 멀티 노드 및 컨테이너에서의 DistDGL 학습
### 2-1. NFS(Network File System) 서버 파드 생성
