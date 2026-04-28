---
layout: single
title:  "GRACEFUL: A Learned Cost Estimator For UDFs"
categories: PaperReview
tag: [rdbms, paper_review]
toc: true
toc_sticky: true
author_profile: true
---

# Rethinking Data Bias: Dataset Copyright Protection via Embedding Class-wise Hidden Bias
## About Paper
- Johannes Wehrstein, Tiemo Bang, Roman Heinrich, Carsten Binnig
- ICDE 2025: IEEE 41st International Conference on Data Engineering
- Technical University of Darmstadt / Microsoft - Gray Systems Lab / DFKI

# 1. Introduction
## 1.1. Abstract
### 1.1.1. User-Defined-Functions (UDFs) are a pivotal feature in modern DBMS...
> UDF는 중요하지만 최적화가 어렵다

- SQL만으로는 표현하기 어려운 사용자/비즈니스 로직을 함수로 생성 가능
    - 문자열 포맷팅
    - 사용자 정의 필터링 조건
    - 특정 도메인 전용 계산
    - 커스텀 aggregation
    - 변환 함수
- 위 예시들은 실제 서비스 환경에서 자주 등장
- 그러나, UDF는 DBMS optimizer 관점에서 다루기 어려운 주제
- UDF 실행에 필요한 cost estimation이 어렵기 때문

### 1.1.2. 기존 cost model의 한계
- 기존 cost model은 보통 아래 연산들에 대해서는 꽤 잘 작동
    - scan, filter, join, sort, aggregation
- 이러한 연산들은 DBMS가 오래 전부터 잘 연구해 온 연산
- cardinality나 selectivity와 같은 통계 정보를 활용하여 대략적인 비용 추산이 가능

- 그러나, UDF는 내부 구조가 너무 다양해서 이런 전통적 hand-crafted cost function으로 다루기 어려움
- 따라서, 기존 optimizer는
    - UDF를 일반 predicate처럼 취급하거나
    - UDF 비용을 일정하다고 가정하거나
    - 아예 제대로 반영하지 못함
- 그 결과 suboptimal plan, 즉 비효율적인 실행 계획을 선택

### 1.1.3. GRACEFUL의 제안
- 따라서, GRACEFUL은 learned cost model을 제안
- 사람이 손으로 UDF 비용 공식을 구하는 대신, 데이터와 실행 결과를 바탕으로 모델이 비용을 학습

- GRACEFUL은 query plan과 UDF의 구조를 함께 보고
    - 이 UDF가 어떤 구조인지
    - 어떤 branch가 얼마나 자주 실행될지
    - 어떤 loop가 얼마나 돌 가능성이 있는지
    - 데이터 분포가 어떤지
- 를 반영하여 실행 비용을 예측

- 위 예측을 바탕으로 optimizer가 더 좋은 결정을 할 수 있음

### 1.1.4 대표적인 효과: filter pull-up / push-down
- 일반적으로 DBMS에서는 filter push-down이 좋은 최적화
    - 가능한 빨리 filter를 적용하여서 row 수를 줄이면 이후 연산이 빨라짐

- UDF filter는 다를 수 있음
    - 어떤 UDF가 매우 비싼 함수라면
    - 초반에 많은 row에 대해 해당 UDF를 사용하는 것은 오히려 손해일 수 있음

- 즉, 
    - 일반 predicate는 빨리 적용하는 것이 좋지만
    - 비싼 UDF predicate는 나중에 적용하는 것이 더 좋을 수도 있음

- GRACEFUL은 이런 경우를 비용 예측으로 판단하여
- pull-up (UDF filter 먼저 적용) / push-down (UDF filter를 나중에 적용) 중 결정
- 그 결과, 약 50배의 속도 개선을 달성

### 1.1.5 데이터셋 공개
- UDF 쿼리 최적화는 데이터셋이 부족한 분야
- 해당 논문에서 9만 개 이상의 synthetic UDF query benchmark를 공개

- 즉, 후속 연구가 쉽게 재현되고 비교 가능한 기반을 제공

## 1.2. Introduction
### 1.2.1. "Modern Databases Increasingly Face UDFs"
- 데이터 규모가 계속 증가
- 처리 요구도 점점 복잡해짐
- 따라서, 계산을 DB 바깥으로 빼기 보다는 데이터 가까이서 처리하는 것이 중요
- 이를 위해 DB 내부에서 custom logic을 실행할 수 있는 UDF가 중요해짐

- 즉, UDF는 단순한 부가 기능이 아니라
- 현대 DBMS에서 실제로 많이 사용되는 실전 기능

- 또한, 저자들은 UDF가 데이터센터에서 매일 수십억 번 실행된다는 것을 언급
- 따라서, UDF 최적화가 실제 시스템 성능에 매운 큰 영향을 미칠 수 있음

### 1.2.2. UDF Optimizations are Crucial
- DBMS의 핵심은 단순히 SQL을 실행시키는 것이 아닌, 효율적인 실행 계획을 고르는 것
- 그러나, UDF는 이 과정에서 문제를 일으킴
- SQL optimizer는 보통 아래를 기준으로 판단
    - 어느 join order가 좋은가
    - filter를 언제 적용할 것인가
    - 어떤 연산을 먼저할 것인가
    - 어떤 index를 사용할 것인가
- 이 모든 것은 cost estimation을 바탕으로 작동
- 그러나, UDF가 개입하면 cost estimation이 불명확해져서 optimizer가 잘못된 선택을 할 수 있음
- 특히, UDF에서는 predicate push-down이 항상 좋은 것이 아님

### 1.2.3. Limited UDF Support in DBMS Cost Estimators
- 사실 UDF를 고려한 query optimization은 오래된 문제
- 과거에도 complex predicate 최적화 문제가 논의됨
- 하지만 보지 못한 UDF의 비용을 일반화해서 예측하는 문제는 아직 해결되지 않았음

- 즉, 기존 연구가 UDF를 빠르게 실행하는 방법이나 특정 상황 최적화는 수행하였더라도
- optimizer가 새로운 UDF를 처음 봤을 때에도 비용을 잘 예측할 수 잇는지에 대한 해법이 부족

### 1.2.4. The impact of pull-up optimization an a SQL query with a UDF
<div align="center">
    <img src="/images/2026-04-22-GRACEFUL_A_Learned_Cost_Estimator_For_UDFs/fig1.png" alt="Fig_1_The_impact_of_pull-up_optimization_an_a_SQL_query_with_a_UDF" width="500">
</div>

- Given Query
    ```SQL
    SELECT * FROM movie_keyword as mk JOIN title as t ON mk.movie_id = t.id
    JOIN movie_info_idx as mi_idx ON t.id = mi_idx.movie_id
    WHERE t.series_years = '1987-1997'
    AND udf(mk.movie_id, mk.keyword_id) <= 26026;
    ```
    - 아래 3개 테이블을 EQUI JOIN
        - movie_keyword as mk, title as t, movie_info_idx as mi_idx
    - EQUI JOIN 조건
        - mk.movie_id = t.id = mi_idx.movie_id
    - '1987-1997' 연도 조건으로 필터링
    - udf(mk.movie_id, mk.keyword_id) <= 26026 조건으로도 필터링
- 일반적으로는 filter를 빨리 적용하는 것이 좋음
- 해당 원칙에 따라, UDF filter도 우선으로 수행하면 오히려 상황이 안좋아짐
- UDF를 너무 일찍 적용하면 4.5 million rows에 대해 UDF를 평가해야함

- 한편, join과 다른 조건들을 먼저 수행한 후에 UDF를 위쪽에서 적용하면
- 69k rows에 대해서만 UDF를 수행
- 즉, UDF가 매우 비싸기 때문에 적은 row에 대해 늦게 평가하는 편이 훨씬 이득

- runtime 비교
    - 잘못된 plan: 21.86s
    - pull-up된 plan: 0.48s

- 위 예시는 아래를 시사
    1. UDF가 있으면 고전적인 optimizer heuristic이 깨질 수 있음
    2. 따라서 UDF 비용을 아는 것이 매우 중요
    3. 정확한 cost estimator만 있으면 엄청난 성능 개선이 가능

### 1.2.5. Our Proposal: Learned Cost Estimator for UDFs
- 논문은 최근 ML 기반 cost estimation이 좋은 성과를 내고 있음을 언급
- 전통적인 hand-crafted optimizer 대신 학습 기반 방법이 database 분야에서 점점 주목을 받음

- 하지만 기존 learned cost model도 주로 일반 SQL 연산 중심
- UDF 내부 구조까지는 잘 다루지 못함
- UDF 그 자체가 하나의 프로그램이기 때문
- 예를 들어 UDF 내에는
    - loop
    - if/else
    - arithmetic
    - string operator
    - library call
- 과 같은 구조가 존재

- 따라서, 단순히 query plan만 볼 것이 아니라
- UDF 자체를 program structure로 표현해야함

- 그래서 GRACEFUL은 query plan + UDF control flow를 함께 보는 모델을 제안

### 1.2.6. Runtime prediction is hard works
- 프로그램 비용 예측은 원래 어려운 문제
- 일반화해서 생각하면, 이는 Turing의 Halting Problem과 유사해 본질적으로 어려움

- 그러나, DBMS의 UDF는 아래의 특성이 있어 상대적으로 다룰 만함
    - UDF는 보통 구조가 크지 않음
        - 몇십~몇백 연산 수준
        - control flow가 비교적 단순
        - loop/branch 수가 제한적
    - DBMS 안에서는 UDF에 들어갈 데이터에 대한 정보가 없음
        - 어떤 테이블에 적용되는지
        - 데이터 분포가 어떠한지
        - branch condition이 얼마나 자주 true인지
        - 이를 database statistics로 어느 정도 추론할 수 있음

- GRACEFUL은 단순히 코드 만을 바탕으로 최적화 하는 것이 아니라 (Compiler Optimization)
- 데이터 특성까지 반영하여 비용을 예측 (GRACEFUL)

### 1.2.7. The Need to Generalize to Unseen UDFs
- UDF는 종류가 매우 다양
- DL 모델 훈련 시점에 본 UDF만 잘 맞추는 모델로는 쓸모가 없음
    - 처음 보는 UDF
    - 처음 보는 SQL 패턴
    - 처음 보는 데이터셋
- 즉, Unseen UDF에도 일반화가 되어야함

- 논문은 GRACEFUL이 아래 세 가지에 대해 generalize할 수 있다고 주장
    1. unseen UDFs: 훈련 때 없던 새로운 함수 코드
    2. unseen SQL workloads: 훈련 때와 다른 질의 패턴
    3. unseen datasets: 훈련 때 없던 데이터셋

### 1.2.8. 핵심 기술: CFG + GNN + 데이터 통계
#### 1.2.8.1. UDF를 Control Flow Graph (CFG)로 표현
- UDF를 단순 텍스트 코드로 부지 않고
- 프로그램의 구조를 나타내는 Control Flow Graph (CFG)로 표현

<div align="center">
    <img src="/images/2026-04-22-GRACEFUL_A_Learned_Cost_Estimator_For_UDFs/cfg1.svg" alt="Control Flow Graph (CFG)" width="300">
</div>

- CFG Representation은 아래를 반여하기에 좋음
    - branch 구조
    - loop 구조
    - execution path
    - operation 유형
- 즉, UDF를 작은 프로그램 그래프로 보는 것

#### 1.2.8.2. Query Plan과 UDF를 joint representation으로 결합
- UDF는 query 바깥에 따로 존재하는 것이 아니라
- query plan 안 특정 위치에서 실행

- 따라서, 
    - query plan 구조
    - UDF 구조
    - plan 내 UDF의 위치
    - 입력 cardinality
- 를 함께 표현해야함

#### 1.2.8.3. GNN으로 비용 예측
- 그래프 구조 데이터이기 때문에 GNN(Grpah Neural Network)를 사용
- GNN은 query plan graph와 CFG 같은 구조적 정보를 반영하기 적합
- 논문은 이를 통해 UDF와 query plan의 joint embedding을 만들고 runtime을 예측

### 1.2.9. Pull-up advisor
- GNN을 통한 비용 예측 정확도를 통해
- 실제 optimizer task의 pull-up advisor를 구축

- 이 비용 모델을 활용해
    - UDF filter를 아래로 내릴지
    - 위로 올릴지
- 를 결정하는 의사결정 모듈을 개발

- 이를 통해 단순 예측 정확도 뿐 아니라
- 실제 optimizer decision에 연결했을 때 end-to-end 성능 이득이 크다는 것을 입증

### 1.2.10. Adaptive Execution is Not to be Preferred
- adaptive execution으로 실행 중간에 plan을 바꾸면
    - adaptive execution은 DBMS에 넣기 복잡
    - execution paradign 자체가 달라질 수 있음
    - 기존 DBMS architecture와 잘 안 맞을 수 있음

- 즉, 현실 시스템 관점에서는
- 기존 optimizer 구조에 자연스럽게 들어가는 cost model 방식이 더 실용적

### 1.2.11. Contributions
1. GRACEFUL 제안
    - UDF가 포함된 query plan의 runtime을 예측할 수 있는
    - GNN 기반 cost estimator를 제안
2. 새로운 UDF 표현 방식
    - UDF 구조와 입력 데이터 분포를 반영하는 transferable representation 제안
    - 특히 database statistics를 이용해 UDF branch hit-ratio를 추정하는 방식이 핵심
3. 실험적 성능 입증
    - 정확도 측면에서 우수
    - 실제 pull-up / push-down 최적화에 큰 성능 향상을 줄 수 있음
4. 코드와 benchmark 공개
    - 9만 개 이상의 query, 20개 데이터베이스에 걸친 benchmark 공개

# 2. Overview of GRACEFUL
![Fig. 2: Cost Estimation Overview for a SQL Query](/images/2026-04-22-GRACEFUL_A_Learned_Cost_Estimator_For_UDFs/fig2.png)

- GRACEFUL의 핵심 구성 4가지
    1. UDF를 CFG로 표현
    2. 그 CFG를 cost estimation에 맞춰 변형하고 annotation을 붙임
    3. 그 UDF 그래프를 query plan 그래프와 합침
    4. 합쳐진 그래프를 GNN에 넣어 runtime을 예측
- 즉, 단순히 SQL plan만 보는 것도 아니고, UDF 코드만 보는 것도 아님
- **Query, UDF, 데이터 통계**를 함께 모델링

## 2.1. Key Ideas of Cost Model
### 2.1.0. CFG Introduction
<div align="center">
    <img src="/images/2026-04-22-GRACEFUL_A_Learned_Cost_Estimator_For_UDFs/udf_to_cfg.png" alt="Generate Control Flow Graph" width="400">
</div>

#### 2.1.0.1. 그래프 구조로 추상화
- 기존 DBMS의 cost model은 대체로 아래 정보를 고려
    - query plan operator 종류
    - cardinality
    - join type
    - scan row 수
    - predicate selectivity
- 그러나, UDF가 개입하면 위 정보만으로는 부족
- UDF 내부에서 발생하는 연산에 따라 runtime에 큰 영향을 주기 때문
    - 단순 덧셈
    - 문자열 파싱
    - 반복문
    - 여러 branch를 가짐
- 위 경우에 따라 비용이 완전 달라짐

- 따라서, GRACEFUL은 runtime estimation을 위해 필요한 대상을 모두 그래프로 추상화
    - **SQL query plan** → query graph
    - **UDF code** → control flow graph
    - **data characteristics / cardinalities** → node/edge annotation
- 즉, 단순한 벡터 feature 몇 개가 아니라
- 구조적 정보(structure)를 가진 그래프로 관찰

#### 2.1.0.2. UDF를 CFG로 표현하는 이유
- CFG(Control Flow Graph)는 프로그램의 실행 흐름을 그래프로 나타낸 것
    - node: 코드 블록 또는 statement
    - edge: 실행 흐름 또는 제어 경로

- e.g., 
    ```python
    if x > 10:
        y += 1
    else:
        y *= 2
    return y
    ```
    - 위 코드를 CFG로 만들면 대략적으로 아래와 같은 형태가 됨

<div align="center">
    <img src="/images/2026-04-22-GRACEFUL_A_Learned_Cost_Estimator_For_UDFs/cfg2.jpg" alt="Control Flow Graph (CFG)" width="300">
</div>

- 이를 활용해 CFG는 프로그램의 branch, loop, 실행 순서를 자연스럽게 담을 수 있음

#### 2.1.0.3. 왜 UDF를 CFG로 보는가?
- UDF는 결국 작은 프로그램
- 따라서 runtime은 다음에 의해 결정됨
    - 어떤 연산이 있는가
    - 몇 개의 branch가 있는가
    - loop가 있는가
    - 어떤 경로가 자주 실행되는가
    - 어떤 연산이 반복되는가
- 위와 같은 정보는 단순 SQL operator feature로는 담아내기 어려움
- CFG로 포현해야 자연스럽게 반영됨

- 즉, UDF는 opque black-box가 아니라, 구조를 가진 code graph로 보자는 제안

#### 2.1.0.4. naive CFG 만으로는 부족하다
- CFG는 코드 블록과 control edge만 표현
- 대용량 데이터를 다루는 DBMS에서는 CFG 만으로 cost estimation이 충분하지 않음
- 어떤 statement가 어떤 데이터에 얼마나 자주 수행되는지가 중요하기 때문

### 2.1.1. Our UDF Representation
<div align="center">
    <img src="/images/2026-04-22-GRACEFUL_A_Learned_Cost_Estimator_For_UDFs/encode_cfg_annotate_branch_selectivity.png" alt="Encode CFG As Graph (DAG), Annotate Branch Selectivities" width="300">
</div>

- 제안 논문은 CFG에 추가적인 변형을 수행
    - exploit loop end 추가
    - single-statement CFG로 변환

#### 2.1.1.1. exploit loop end 추가
- 일반 CFG에서는 loop의 끝이나 반복 경계가 명시적으로 잘 드러나지 않을 수 있음
- 그러나, runtime estimation에서는 loop가 매우 중요
    - loop가 있는가?
    - 몇 번 반복하는가?
    - loop 내부 연산이 무엇인가?
- 위와 같은 것들이 runtime에 직접적인 영향을 미침

- loop end 구조를 명시적으로 드러내면
- 모델이 반복 구조를 더 쉽게 학습할 수 있음

- 즉, 원래 CFG보다 비용 추정에 유리한 형태로 구조를 정리

#### 2.1.1.2. single-statement CFG
- 기존 CFG는 하나의 node가 여러 줄의 code block일 수 있음
- 예를 들어 한 basic block 내에 여러 statement가 들어갈 수 있음
- 그러나, 이런 경우에 node 단위가 너무 뭉뜽그려져서
    - 어느 연산이 비싼지
    - 어느 statement가 branch 직전인지
    - 어느 부분이 loop 안에 있는지
- 를 세밀하게 관찰히가 어려움

- GRACEFUL은 code block을 쪼개서
- statement 하나당 node 하나 수준으로 더 fine-grained 하게 표현

- 이를 통해 GNN이 더 세밀한 연산 구조를 학습할 수 있음
- 즉, **프로그램 구조를 runtime prediction에 적합한 해상도로 분할**하는 것

#### 2.1.1.3. data-flow annotation
- CFG 만으로 runtime을 예측하는 것은 매우 어려움
- 같은 코드이더라도 입력 데이터에 따라 실행 경로와 반복 횟수가 달라지기 때문

- e.g., 
    ```python
    if salary > 10000:
        expensive_func()
    ```
    - 위 코드의 실제 runtime은
        - `salary > 10000`인 row가 몇 개인지
        - expensive_func()가 얼마나 자주 호출되는지
    - 에 따라 달라짐

- 일반적인 실행 환경에서는 위 코드의 runtime을 예측하기 어려우나,
- DBMS 환경에서는 기존 DBMS가 가진 통계 정보를 활용하여 위 UDF 함수의 cost estimation이 가능

#### 2.1.1.4. cardinality
- DBMS cost model에서 cardinality는 가장 핵심적인 정보
- 연산 비용은 대체로 "몇 개의 tuple을 처리하느냐"에 크게 좌우되기 때문

- 예를 들어 join이던 filter이든
    - 입력 row의 수
    - 출력 row의 수
- 가 runtime에 매우 큰 영향을 줌

- 논문은 이 점을 UDF 내부에도 확장
    - query plan operator에도 cardinality가 필요
    - UDF 내부 data flow에도 cardinality 비슷한 정보가 필요

- 예를 들어 UDF 내부 if-branch가 존재하는 경우
    - 몇 %의 row가 true branch로 갈 것인가?
- 는 사실상 UDF branch 내부 연산에 대한 cardinality 정보

#### 2.1.1.5. UDF 내부 cardinality와 hit-ratio estimator
- DBMS는 일반적으로 query plan operator에 대해서는 cardinality estimate를 제시
    - scan 결과 row 수
    - join 결과 row 수
    - filter 후 row 수
- 그러나, UDF 내부에 대해서는 이러한 정보가 없음

- 예를 들어, UDF 내부에
    ```python
    if col_a > 5:
        # do inexpensive something
        # ...
    else:
        # do expensive something
        # ...
    ```
    - 기존 DBMS에서는 UDF에 
        - 몇 개의 row가 if branch로 가는지
        - 몇 개의 row가 else branch로 가는지
    - 알려주지 않음
- 이를 해결하기 위해 논문은 **hit-ratio estimator**를 제안
    - hit ratio: 특정 branch 조건이 참이 될 비율
        - `then` branch가 얼마나 자주 실행되는지
        - `else` branch가 얼마나 자주 실행되는지
    - 따라서, **hit-ratio estimator**를 통해 분기문 runtime에 대한 기대값을 계산할 수 있음

- **hit-ratio**를 어떻게 **estimate**할 것인가?
    - UDF 내부 branch 조건을 그냥 프로그램 조건으로 두지 않음
    - 가능하면 SQL predicate 형태로 변형하여 DBMS cardinality estimator에 정보를 요청
    - 예를 들어, UDF 내부 조건이 테이블 컬럼 기반이면
        ```python
        if a > 10 and b = 'x'
        ```
        - 이를 SQL WHERE clause처럼 바꾸어서
        - DBMS의 기존 통계 기반 caridnality estimator를 활용
    - 즉, 논문은 새로운 branch selectivity 모델을 완전히 새로 만드는 것이 아니라
    - 기존 DBMS의 통계 기반 추정 기능을 재활용

### 2.1.2. Joint Query-UDF Representation
- GRACEFUL이 단순 "UDF cost estimator"가 아니라
- 정확히는 **query plan runtime estimator**라는 점을 보여줌
- 실제 실행 시간은 UDF만으로 결정되지 않기 때문

- runtime은 아래 조건이 모두 영향을 미침
    - query plan 구조
    - join order
    - operator cardinality
    - UDF가 plan의 어디에 위치하는지
    - UDF 앞뒤 연산 결과 row 수
    - UDF 내부 구조와 branch cost

- 따라서 UDF 그래프만 따로 파악하면 안 되며
- query graph와 joint embedding을 해야 함

- 즉,
    - query plan graph는 SQL 실행 구조를 표현
    - UDF CFG는 함수 내부 실행 구조를 표현
    - 둘을 연결하여 하나의 큰 그래프로 만들어 runtime을 찾음

#### 2.1.2.1. query plan operator에도 cardinality annotation이 중요한 이유
- DBMS에서 동일한 join operator이어도
    - 입력이 100 rows 인지
    - 입력이 100M rows 인지
- 에 따라 비용이 완전 다름

- 따라서, opeartor type 뿐 아니라 입출력 cardinality가 필요
- GRACEFUL은 DBMS의 cost estimation 개념을 그대로 활용하되, UDF가 있는 경우 추가 난점이 존재한다고 주장

#### 2.1.2.2. UDF filter가 있으면 output cardinality를 모름
- 일반적으로 filter의 output cardianlity는 selectivity로 계싼
    - e.g., 
    - 입력 1,000,000 rows
    - selectivity 0.1 → 출력 100,000 rows
- 이때, filter predicate가 UDF인 경우 문제가 발생
    - e.g.,
        ```SQL
        WHERE udf(col1, col2) <= 26026;
        ```
    - 위 경우, DBMS는 일반적으로
        - UDF가 어떤 값 분포를 내는지
        - 조건을 몇 %가 만족하는지 (i.e., selectivity)
    - 를 모르기 때문에 output cardinality를 쉽게 계산할 수 없음
    - 즉, plan graph에서 중요한 feature 하나가 비게 됨
    - 이는 단순 cost prediction뿐 아니라 optimizer decision에도 문제를 발생시킴

### 2.1.3. Point estimation of UDF-filter selectivity is unknown
- 일반적인 SQL filter는 예를 들어
    ```SQL
    WHERE age > 30
    ```
- 이러한 경우에 DBMS는 통계(histogram, NDV, min/max 등)을 활용해서
    - 전체 row 중 몇 %가 살아남는지
    - 즉, filter selectivity가 얼마인지
- 를 추정할 수 있음

- 예를 들어,
    - 입력 1,000,000 rows
    - selectivity 0.2
- 이면 출력은 대략 200,000 rows
- 이를 바탕으로 join, aggregation, sort 등의 비용을 계산할 수 있음

- 그러나 UDF filter는 다름
    ```SQL
    WHERE udf(a, b, c) <= 26026
    ```
- 과 같은 경우 DBMS는 아래 사항들을 알지 못함
    - udf 결과값 분포가 어떠한지
    - 몇 %의 row가 조건을 통과하는지
    - 해당 filter를 적용한 뒤 몇 개의 row가 남을지
- 즉, UDF-filter selectivity가 unknown임

- 그러나, optimizer는 이 값을 꼭 알아야함
- selectivity가 달라지면 "push-down이 좋은지, pull-up이 좋은지"가 달라지기 때문

#### 2.1.3.1. Solution: Predict a distribution of the costs
- GRACEFUL's probabilistic approach: 
    - UDF filter selectivity가 여러 값일 가능성을 염두
    - 해당 경우들에 대해 비용을 계산/예측
    - 단일 cost가 아니라 cost distribution을 생성
- 즉, **UDF 때문에 불확실성이 생기면, 그냥 무시하지 말고 확률적 분포로 다룸**

#### 2.1.3.2. selectivity에 따라 달라지는 cost
- 일반적인 cost model
    - $\hat{C}=f(plan_features)$
- 즉, 입력 plan에 따라 **cost 결과 하나**를 출력

- 그러나, UDF filter가 있으면 plan의 핵심 변수가 추가되어야함
    - $s=\text{UDF filter selectivity}$

- 위 비용에 따라 전체 실행 계획 비용이 달라짐
- 즉, cost model은 selectivity라는 종속 변수를 가지고 있음
    - $C=C(s)$

- 예를 들어, 
    - $s=0.001$: 거의 다 걸러짐 → early push-down이 유리할 수 있음
    - $s=0.9$: 거의 안 걸러짐 → expensive UDF를 늦게 적용하는 것이 유리할 수 있음

#### 2.1.3.3. Iterate over the UDF-filter selectivity
- UDF-filter selectivity가 정확히 무엇인지 모르는 상황에서
- 가능한 여러 selectivity에 따른 cost estimation의 집합을 생성
    - e.g., $s \in \{ 0.01, 0.05, 0.1, 0.2, 0.5, 0.8 \}$
        1. 각 값에 대하여 출력 cardinality 계산
        2. 그 cardinality가 후속 operator에 어떤 영향을 주는지 반영
        3. 전체 query plan cost를 각각 예측
    - 즉, 아래와 같은 형식
    - $\{ C(0.01), C(0.05), C(0.1), C(0.2), C(0.5), C(0.8)\}$
        - 위 여러 cost estimation의 집합이 cost distribution의 근사
- **selectivity uncertainty를 discretize해서 여러 시나리오를 평가**

#### 2.1.3.4. Cost Distribution
- cost distribution이 꼭 엄밀한 확률밀도함수(pdf)를 의미하는 것은 아님
    - UDF-filter selectivity가 불확실
    - 가능한 여러 경우(다양한 selectivity)에 대해 cost를 예측
    - 그 결과로 **cost가 어느 범위에 있을 법한지** 관찰

- 예를 들어, 어떤 plan에 대해: 
    - example A: 
        | assumed selectivity | predicted cost |
        | :-----------------: | :------------: |
        | 0.01                |        2.1 sec |
        | 0.05                |        2.5 sec |
        | 0.10                |        3.2 sec |
        | 0.20                |        4.8 sec |
        | 0.50                |        9.7 sec |
        | 0.80                |       14.1 sec |
    - example B: 
        | assumed selectivity | predicted cost |
        | :-----------------: | :------------: |
        | 0.01                |        6.3 sec |
        | 0.05                |        6.4 sec |
        | 0.10                |        6.5 sec |
        | 0.20                |        6.7 sec |
        | 0.50                |        7.0 sec |
        | 0.80                |        7.2 sec |

- 위 distribution을 활용하여 아래 정보를 구할 수 있음
    - cost에 대한 기댓값 (기대 비용)
    - 최악 비용 (maximum predicted cost)
    - 분산/불확실성
    - 특정 selectivity 구간에서의 취약성

#### 2.1.3.5. pull-up / push-down decision using cost distribution
- 2개의 후보 plan이 있다고 하자
    - Plan A: UDF filter pull-up
    - Plan B: UDF filter push-down

- 만약 UDF-filter selectivity를 정확히 모르면, 두 plan 중 어느 것이 좋은지 단정할 수 없음
- 예를 들어: 
    - selectivity가 높으면 pull-up이 좋을 수 있음
        - (비싼 UDF를 초반 대량 row에 적용하는 것이 손해)
    - selectivity가 아주 낮으면 push-down이 좋을 수 있음
        - (초반에 많이 걸러지므로 이후 연산이 줄어듬)

- 따라서, plan 비교는 사실
    - $C_A(s)$ vs. $C_B(s)$
- 를 비교하는 문제

- 즉, selectivity에 따라 우열이 바뀔 수 있음

- UDF는 selectivity 하나를 확정(point estimation)이 불가능 하므로
- Graceful은 두 plan의 cost distribution을 비교

- 이에 따라: 
    - 기대값 기준으로 더 나은 plan
    - worst-case가 더 안전한 plan
    - regret이 더 작은 plan
- 등을 선택할 수 있음

### 2.1.4. Runtime Prediction
- UDF CFG와 SQL query plan을 합친 그래프를 GNN에 입력
- GNN이 생성한 graph embedding을 regression layer가 받아 query runtime을 예측
- 모델은 특정 DB나 특정 UDF에 재학습하지 않아도 zero-shot으로 동작

#### 2.1.4.1. Joined query-UDF graph
1. Query graph
    - SQL query plan을 그래프로 표현
    - e.g., `scan`, `filter`, `join`, `aggregation`과 같은 operator가 node
2. UDF graph
    - UDF 코드를 CFG 형태로 표현
    - e.g., `statement`, `branch`, `loop`, `function call` 등이 node

- 예를 들어, SQL plan에 아래와 같은 filter가 있다고 하자
    ```SQL
    WHERE udf(a, b) < 100
    ```
- query graph의 `filter` node와 UDF CFG가 연결됨
- 즉, 최종 입력은 SQL query plan graph와 UDF control-flow graph가 합쳐진 하나의 그래프
- 이를 **joined query-UDF graph**라고 함

#### 2.1.4.2. Runtime prediction via GNNs
- joined query-UDF graph에는 아래 정보를 포함
    - query operator 간 parent-child 관계
    - UDF 내부 control-flow 관계
    - branch 경로
    - loop 구조
    - UDF가 query plan 어디에 붙어 있는지
    - opeartor cardinality
    - UDF statement feature
- 따라서 일반 MLP보다 GNN이 적합

- GNN은 각 node가 주변 node와 message passing을 하면서
- 전체 그래프 구조를 반영한 representation을 학습
    - 이 UDF는 어떤 구조를 가지고 있으며
    - 이 query plan 안에서 어디에 위치하며
    - 어느 정도 row에 대해 실행되는가?
- 를 GNN이 embedding으로 압축

#### 2.1.4.3. Regression model as final layer
- GNN이 생성한 hidden state vector를 regression model에 입력
- regression model에서 최종 query runtime을 예측
- 추가적으로, 학습 안정화를 위해 log runtime을 예측

- 즉, pull-up / push-down classification이 아닌 run-time regression

#### 2.1.4.4. Does this model predicts only UDF cost?
- UDF가 포함된 query plan 전체 runtime을 예측
- UDF 비용은 query context와 분리해서 생각하면 안되기 때문
- 같은 UDF이어도
    - 1,000 rows에 적용되는지
    - 10,000,000 rows에 적용되는지
    - join 전에 실행하는지
    - join 후에 실행하는지
    - branch가 어느 데이터에서 자주 hit 되는지
- 에 따라 전체 비용이 달라짐

- 따라서, GRACEFUL은 UDF 단독 비용이 아닌
- query plan 내에서 UDF가 실행될 때의 전체 runtime을 예측

#### 2.1.4.5. zero-shot manner
- 새로운 데이터베이스, 새로운 UDF, 새로운 SQL query를 사전에 학습하지 않더라도 바로 runtime을 예측
- 즉, 모델이 특정 UDF를 미리 실행하고 학습하는 방식이 아님

- DBMS optimizer에 들어가려면 zero-shot이 매우 중요
- 만약 어떤 새 UDF가 들어올 때마다
    1. 그 UDF를 여러 query에 대해 실행
    2. runtime 데이터 수집
    3. 모델 재학습
    4. optimizer가 정보 활용
- 위 과정을 거치면 실용성이 매우 떨어짐

- 실제 DBMS에서는 사용자가 임의의 UDF를 계속 만들 수 있음
- 따라서 optimizer는 처음 보는 UDF에 대해서도 즉시 판단해야함
- 즉, **기존에 본 적 없는 UDF structure, dataset, SQL query에 대해서도 코드 구조와 데이터 통계를 활용해 대략적인 비용을 예측**

#### 2.1.4.6. database-independent featurization
- feature가 특정 데이터베이스에 종속되지 않음
- 나쁜 feature의 예시는 아래와 같음
    ```
    table_name = "movie_keyword"
    column_name = "series_years"
    udf_name = "my_udf_17"
    database_name = "imdb"
    ```
    - 위와 같은 feature를 사용하면 모델이 특정 DB나 UDF를 외울 가능성이 큼
    - 새로운 DB에서는 작동하기 어려움

- 반면, database-independent feature (제안 논문)은 아래와 같음

<div align="center">
    <img src="/images/2026-04-22-GRACEFUL_A_Learned_Cost_Estimator_For_UDFs/table1.png" alt="table1_features_of_the_UDF_node_types" width="500">
</div>

## 2.2. Discussion of Scope
### 2.2.1. Scalar Python UDFs
### 2.2.2. Non-UDF Queries

# 3. GRACEFUL Design
## 3.1. UDF Representation
### 3.1.1. Control-Flow Graph as Basis
### 3.1.2. Transforming the CFG
### 3.1.3. Handling Loops
### 3.1.4. Various UDF Node Types
### 3.1.5. Transferable Featurization

## 3.2. UDF Selectivity Annotation
### 3.2.1. Hit-Ratios Estimation / Branch Prediction

## 3.3. Joint Query-UDF Representation

## 3.4. Model Architecture

# 4. Pull-up / Push-down Advisor
## 4.1. Why this Problem?
## 4.2. Regret Optimization
## 4.3. Pull-up/Push-down Decision

# 5. A novel UDF Benchmark
## 5.1. Benchmark Design
## 5.2. A Resource for UDF Cost Models

# 6. Experimental Evaluation
# 7. Related Work