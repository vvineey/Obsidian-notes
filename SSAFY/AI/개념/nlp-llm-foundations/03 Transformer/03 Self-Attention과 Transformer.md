---
date: 2026-08-18
course: SSAFY AI
title: 03 Self-Attention과 Transformer
status: approved
repository: nlp-llm-foundations
tags:
  - course/ssafy-ai
  - type/concept-note
  - repo/nlp-llm-foundations
  - exam/ai-theory-2026-08-20
  - ai/transformer
concepts:
  - "[[02 자연어 생성과 Attention]]"
  - "[[04 사전학습 기반 언어 모델]]"
related:
  - "[[00 자연어처리 기본 인덱스]]"
  - "[[00 2026-08-20 AI 이론 2회차 평가 대비 목차]]"
sources:
  - SSAFY 강의 슬라이드 — Transformer
  - Chapter 2-1 토큰화·임베딩 심화 과제
---

# 03 Self-Attention과 Transformer

> [!summary] 오늘의 핵심
> Self-Attention은 같은 시퀀스의 각 토큰이 다른 모든 토큰을 얼마나 참고할지 계산한다. Transformer는 이 연산에 위치 정보, Multi-Head Attention, Feed-Forward Network, residual connection과 Layer Normalization을 결합하여 순환 없이 문맥 표현을 만든다.

## 목차

- [[#1. Self-Attention]]
- [[#2. Scaled Dot-Product Attention]]
- [[#3. Multi-Head Attention]]
- [[#4. 위치 정보와 마스킹]]
- [[#5. Transformer 블록]]
- [[#6. Encoder와 Decoder]]
- [[#7. RNN과 Transformer 비교]]
- [[#8. 평가 대비 핵심]]
- [[#주요 개념 연결]]

## 학습 목표

- Query, Key, Value의 역할과 행렬 shape을 설명한다.
- dot product를 $\sqrt{d_k}$로 나누는 이유를 설명한다.
- Multi-Head Attention, positional encoding, causal mask의 필요성을 설명한다.
- Transformer encoder와 decoder의 정보 접근 범위를 비교한다.

## 1. Self-Attention

Self-Attention은 한 시퀀스 안에서 각 토큰이 문맥을 만들기 위해 다른 토큰을 참고하는 연산이다. 입력 행렬이 $X\in\mathbb{R}^{T\times d_{model}}$이면 학습 가능한 투영으로 Q, K, V를 만든다.

$$Q=XW_Q,\qquad K=XW_K,\qquad V=XW_V$$

- **Query**: 현재 토큰이 찾는 정보
- **Key**: 각 토큰이 어떤 정보를 가졌는지 나타내는 검색 표지
- **Value**: 실제로 섞어 가져올 내용

Q와 K가 attention weight를 결정하고, 그 weight로 V를 가중합한다. Q·K·V는 고정된 역할의 단어 목록이 아니라 입력에서 학습된 서로 다른 투영이다.

## 2. Scaled Dot-Product Attention

$$\operatorname{Attention}(Q,K,V)=\operatorname{softmax}\left(\frac{QK^\top}{\sqrt{d_k}}+M\right)V$$

행렬 흐름은 다음과 같다.

| 단계 | shape | 의미 |
|---|---|---|
| $Q$ | $T_q\times d_k$ | query 표현 |
| $K$ | $T_k\times d_k$ | key 표현 |
| $QK^\top$ | $T_q\times T_k$ | 모든 query–key 점수 |
| softmax 결과 | $T_q\times T_k$ | 각 query가 key를 참고하는 비율 |
| 결과 | $T_q\times d_v$ | value의 문맥 가중합 |

$d_k$가 커지면 내적의 분산이 커져 softmax가 지나치게 뾰족해질 수 있다. $\sqrt{d_k}$로 나누면 점수 규모를 안정화해 gradient가 너무 작아지는 현상을 완화한다.

## 3. Multi-Head Attention

하나의 attention만으로는 한 종류의 관계에 집중할 수 있다. Multi-Head Attention은 서로 다른 투영 공간에서 attention을 병렬 계산한다.

$$head_i=\operatorname{Attention}(QW_i^Q,KW_i^K,VW_i^V)$$

$$\operatorname{MHA}(Q,K,V)=\operatorname{Concat}(head_1,\dots,head_h)W^O$$

각 head는 문법적 관계, 장거리 참조, 위치적 패턴 등 서로 다른 관계를 학습할 가능성이 있다. 그러나 특정 head가 반드시 사람이 이름 붙인 관계 하나를 학습한다고 보장되지는 않는다.

> [!warning] 오개념
> head 수를 늘리면 각 head 차원은 보통 $d_{model}/h$로 줄어든다. head 수만큼 모델 전체 출력 차원이 무조건 커지는 것은 아니다.

## 4. 위치 정보와 마스킹

### Positional Encoding

Self-Attention 자체는 입력 순서를 바꾸면 출력도 같은 방식으로 순열만 바뀌는 성질이 있어 순서를 스스로 알지 못한다. 따라서 token embedding에 위치 표현을 더한다.

$$Z_0=E_{token}+E_{position}$$

위치 표현은 고정 sinusoidal 방식 또는 학습 가능한 임베딩 방식으로 만들 수 있다.

### Padding Mask

길이를 맞추기 위한 padding 토큰이 실제 내용처럼 참고되지 않게 해당 점수에 큰 음수를 더한다.

### Causal Mask

자기회귀 decoder는 미래 정답을 보면 안 된다. 위치 $t$가 $t$ 이후 key를 참고하지 못하도록 상삼각 영역을 가린다. softmax 전 $-\infty$에 가까운 값을 더하면 해당 확률은 0에 가까워진다.

## 5. Transformer 블록

대표적인 블록은 다음 구성 요소를 반복한다.

```text
입력
 ├─ Multi-Head Attention ─┐
 └────────────────────────┴→ Add & LayerNorm
                              ├─ Feed-Forward ─┐
                              └────────────────┴→ Add & LayerNorm → 출력
```

### Position-wise Feed-Forward Network

각 토큰 위치에 동일한 두 층 MLP를 독립적으로 적용한다.

$$\operatorname{FFN}(x)=W_2\phi(W_1x+b_1)+b_2$$

Attention이 **토큰 사이 정보 교환**을 담당한다면 FFN은 **각 토큰 표현의 비선형 변환**을 담당한다.

### Residual Connection

$$y=x+F(x)$$

입력을 우회 경로로 더해 정보와 gradient 흐름을 돕는다. 입력과 $F(x)$의 shape은 같아야 한다.

### Layer Normalization

각 토큰 표현의 feature 차원을 정규화해 깊은 네트워크 학습을 안정화한다. Batch Normalization처럼 배치 전체 통계에 의존하지 않는다는 점이 중요하다.

## 6. Encoder와 Decoder

| 구조 | Attention 범위 | 대표 목적 |
|---|---|---|
| Encoder | 양방향 self-attention | 입력 이해·표현 |
| Decoder | causal self-attention | 다음 토큰 생성 |
| Encoder–Decoder | encoder self-attention + decoder causal attention + cross-attention | 입력 조건부 생성 |

Cross-Attention에서 query는 decoder 상태에서, key와 value는 encoder 출력에서 온다. Self-Attention에서는 Q·K·V의 원천이 같은 시퀀스다.

## 7. RNN과 Transformer 비교

| 구분 | RNN/LSTM | Transformer |
|---|---|---|
| 순서 처리 | 상태를 시간축으로 전달 | 위치 표현 추가 |
| 토큰 관계 | 여러 시점을 순차 통과 | 한 층에서 직접 연결 |
| 학습 병렬화 | 시간축 병렬화 어려움 | 시퀀스 위치 병렬 처리 가능 |
| 긴 관계 경로 | 길이에 비례 | attention 한 번으로 접근 |
| 긴 시퀀스 비용 | 순차 계산 | 기본 attention은 $O(T^2)$ 메모리·연산 |

Transformer의 병렬 처리는 **학습 시 한 시퀀스의 여러 위치**를 동시에 계산할 수 있다는 뜻이다. 자기회귀 생성에서는 다음 토큰이 이전 생성 결과에 의존하므로 출력 토큰 자체는 순차적으로 생성한다.

## 8. 평가 대비 핵심

> [!question] 예상 문제
> 1. Q, K, V의 역할을 검색 비유와 수식으로 설명하라.
> 2. $\sqrt{d_k}$ scaling이 필요한 이유는 무엇인가?
> 3. 위치 임베딩과 causal mask는 각각 어떤 문제를 해결하는가?
> 4. Attention과 FFN의 역할을 비교하라.
> 5. Transformer가 RNN보다 병렬화에 유리하지만 긴 입력에서 비싼 이유를 설명하라.

서술형 핵심 흐름: **임베딩+위치 → QKV 투영 → 점수와 scaling → mask와 softmax → value 가중합 → multi-head 결합 → residual·normalization → FFN**.

## 주요 개념 연결

- [[02 자연어 생성과 Attention]]: encoder–decoder Attention의 병목 해결 원리를 잇는다.
- [[04 사전학습 기반 언어 모델]]: Transformer 블록을 어떤 mask와 학습 목표로 사전학습하는지 비교한다.
- [[01 워드 임베딩과 순환신경망]]: 순환 상태 전달과 직접적인 token-to-token 관계 계산을 비교한다.
