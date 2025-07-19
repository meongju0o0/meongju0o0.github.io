---
layout: single
title:  "LLM, RAG, KG, RecSys 트렌드 리뷰"
categories: PaperReview
tag: [deep_learning, paper_review, llm, rag, kg]
toc: true
toc_sticky: true
author_profile: true
---

# LLM, RAG, KG, RecSys 트렌드 리뷰
- LLM: Large Language Model
    - [https://ko.wikipedia.org/wiki/대형_언어_모델](https://ko.wikipedia.org/wiki/대형_언어_모델)
- RAG: Retrieval-Augmented Generation
    - [https://ko.wikipedia.org/wiki/검색증강생성](https://ko.wikipedia.org/wiki/검색증강생성)
- KG: Knowledge Graph
    - [https://ko.wikipedia.org/wiki/지식_그래프](https://ko.wikipedia.org/wiki/지식_그래프)
- RecSys: Recommender System
    - [https://ko.wikipedia.org/wiki/추천_시스템](https://ko.wikipedia.org/wiki/추천_시스템)

## Improving Multimodal Social Media Popularity Prediction via Selective Retrieval Knowledge Augmentation
### 논문 정보
- 출판년도: 2025
- 저자: Xovee Xu, Yifan Zhang, Fan Zhou, JingKuan Song
- 기관: University of Electronic Science and Technology of China
- Proceedings of the AAAI Conference on Artifical Intelligence (AAAI)

### 서론
- 온라인 **사용자 생성 콘텐츠(UGC, User Generated Contents)**의 인기도를 예측하는 것은 소셜 미디어 추천, 광고, 마케팅 등에 중요
- 기존 연구는 UGC 자체의 텍스트, 이미지 등 멀티모달 특성만을 기반으로 예측
- 그러나 실제 UGC의 인기도는 다른 **UGC와의 관계성, 사용자 정보, 유행, 알고리즘** 등의 외부 요인에 크게 영향을 받음

### 제안 기법: 지식 검색 기반 멀티모달 인기도 예측 프레임워크
- 멀티모달 + 메타 기반 검색
    - 단순히 의미적으로 유사한 콘텐츠를 검색하는 것이 아닌
    - 이미지와 텍스트의 의미, 사용자 정보 및 게시물 메타데이터 까지
    - 위 피쳐들을 고려하여 다양한 맥락적으로 연관된 UGC들을 검색
- RRCP(Relative Retrieval Contribution to Prediction)
    - 검색된 UGC 중, 예측에 정말 기여하는 콘텐츠만 선택하기 위한 measure
    - 제안 measure를 통해 잡음이나 무관한 콘텐츠를 제거, 모델의 정확도 개선
- Vision-Language Graph Neural Netowrk
    - 최종적으로 선택된 콘텐츠 간의 관계를 그래프 형태로 학습
    - 이미지와 텍스트를 함께 처리하는 GNN을 통해 컨텍스트 정보 강화

## DFA-RAG: Conversational Semantic Router for Large Language Model with Definite Finite Automaton
### 논문 정보
- 출판년도: 2024
- 저자: Yiyou Sun, Junjie Hu, Wei Cheng, Heifeng Chen
- ICML'24: Proceedings of the 41st International Conference on Machine Learning

### 문제 상황
- 기존 LLM은 다양한 상황에서 잘 작동하지만, 감정 상담, 고객 응대 등 정해진 응답 규칙이 필요한 경우에 적절한 대응을 하지 못하는 경우가 많음
- 이처럼 정형화된 응답 흐름이 요구되는 상황에서 LLM이 자율적으로 응답을 생성하면 일관성 부족, 규칙 위반이 발생할 수 있음

### 제안 기법: DFA-RAG
- DFA-RAG는 DFA(Deterministic Finite Automaton)와 RAG(Retrieval-Augmented Generation)을 결합한 방식
- **DFA의 각 노드를 임베딩**하여 **RAG 기법**을 통해 **LLM이 절차에 맞는 응답을 생성**하는 방식

![DFA-RAG](/images/2025-07-20-LLM_RAG_KG_trends/그림1.png)

## MKGL: Mastery of a Three-Word Language
### 논문 정보
- 출판년도: 2024
- 저자: Lingbing Guo, Zhongpu Bo, Zhuo Chen, Yichi Zhang, Jiaoyan Chen, Yarong Lan, Mengshu Sun, Zhiqiang Zhang, Yangyifei Luo, Qian Li, Qiang Zhang, Wen Zhang, Huajun Chen
- 38th Conference on Neural Information Processing Systems (NeurIPS 2024)

### 문제 상황
- LLM은 다양한 자연어 처리 작업에서 뛰어난 성능을 보여주나, 환각 현상과 같은 문제점을 여전히 가지고 있음
- 이러한 문제점을 해결하고자 지식 그래프에서 정보를 추출하여 LLM에 투입하는 RAG 기술들이 제안됨
- 그러나, 사실 기반 구조인 지식 그래프 (Knowledge Graph)와 LLM은 아직 충분히 결합되지 않음
- KG는 보통 (주어, 관계, 목적어) 형식의 트리플(triple) 구조를 가지며, 환각이 거의 없는 신뢰할 수 있는 정보 체계임

### 제안 기법: KGL(Knowledge Grpah Language)
- 이에 논문은 LLM이 KG 기반 정보를 좀 더 잘 다룰 수 있도록 KGL(Knowledge Graph Language)이라는 특수 언어를 제안
- KG의 텍스트 자체를 수정하는 Hard한 방식이 아닌, 제안 모델을 통해 KG에서 추출되는 임베딩 토큰 벡터를 LLM이 더 잘 이해할 수 있는 형태로 수정하는 Soft한 방식으로 유추됨

![MKGL](/images/2025-07-20-LLM_RAG_KG_trends/그림2.png)

## Latent Paraphrasing: Perturbation on Layers Improves Knowledge Injection in Language Models
### 논문 정보
- 출판년도: 2024
- 저자: Minki Kang, Sung Ju Hwang, Gibbeum Lee, Jaewoong Cho
- 38th Conference on Neural Information Processing Systems (NeurIPS 2024)

### 문제 상황
- LLM이 전문 분야에 널리 사용되면서, 지속적으로 변화하는 지식을 신속하고 정확하게 반영하는 것이 중요
- 기존에는 패러프레이징(paraphrasing)된 데이터를 이용해 파인튜닝(fine-tuning)하는 방식이 일반적
- 기존 방식으로는 아래 두 문제점이 존재
    - 높은 계산 비용: 패러프레이징을 위해 외부 모델(예: 다른 LM, LLM)을 반복 사용
    - 제한된 다양성: 생성된 문장 수가 적어 지식 일반화에 한계가 존재

### 제안 기법: LaPael (Latent Paraphrasing on Early Layers)
- 모델 내부에서 입력에 따라 초기 레이어에 노이즈를 가하는 방식으로 페러프레이징
- 이는 잠재 표현(latent representation) 수준에서 동작. 이에 따라 아래의 장점이 존재
    - 입력에 따른 다양하고 일관된 문장 표현 생성 가능
    - 외부 모델 없이 내부에서 직접 패러프레이징
    - 매번 지식이 업데이트 될 때마다 새로운 문장을 다시 만들 필요가 없음

![LaPael](/images/2025-07-20-LLM_RAG_KG_trends/그림3.png)
- 기존 Transformer 구조에 Latent Paraphraser 레이어를 추가
- Latent Paraphraser 레이어를 제안하는 KL Divergence Loss Function으로 잘 학습시키면 적은 양의 데이터로 fine-tuning이 가능하다는 것으로 유추됨
- *Early Layer?? 논문을 다 읽어보진 않았으나.. 초반 Transformer Layer에만 Latent Paraphraser를 추가하는 방식??*

## ChatQA: Surpassing GPT-4 on Conversational QA and RAG

## RankRAG: Unifying Context Ranking with Retrieval-Augmented Generation in LLMs

## ChatQA2: Bridging the Gap to Propriety LLMs in Long Context and RAG Capabilities

## The Elephant in the Room: Rethinking the Usage of Pre-trained Language Model in Sequential Recommendation

## Knowledge-based Multiple Adaptive Spaces for Recommendation

## Heterogeneous Knowledge Fusion: A Novel Approach for Personalized Recommendation via LLM

## Going Beyond Local: Global Graph-Enhanced Personalized News Recommendations
