---
date: 2026-08-18
course: SSAFY AI
title: 02 자연어 생성과 Attention
status: approved
repository: nlp-llm-foundations
tags:
  - course/ssafy-ai
  - type/concept-note
  - repo/nlp-llm-foundations
  - exam/ai-theory-2026-08-20
  - ai/nlp
concepts:
  - "[[01 워드 임베딩과 순환신경망]]"
  - "[[03 Self-Attention과 Transformer]]"
related:
  - "[[00 자연어처리 기본 인덱스]]"
  - "[[00 2026-08-20 AI 이론 2회차 평가 대비 목차]]"
sources:
  - SSAFY 강의 슬라이드 — 자연어 생성 모델(Seq2Seq, Attention)
---

# 02 자연어 생성과 Attention

> [!summary] 오늘의 핵심
> 언어 모델은 이전 토큰이 주어졌을 때 다음 토큰의 확률을 학습한다. Seq2Seq는 encoder가 입력을 문맥으로 압축하고 decoder가 출력 시퀀스를 생성하지만, 하나의 고정 길이 벡터는 긴 입력에서 병목이 된다. Attention은 매 출력 시점마다 모든 encoder 상태를 다시 참고하여 필요한 정보의 가중합을 만든다.

## 목차

- [[#1. 언어 모델이란]]
- [[#2. Seq2Seq 구조]]
- [[#3. 학습과 추론]]
- [[#4. 고정 길이 문맥의 병목]]
- [[#5. Attention의 동작]]
- [[#6. 장점과 한계]]
- [[#7. 평가 대비 핵심]]
- [[#주요 개념 연결]]

## 학습 목표

- 자기회귀 언어 모델의 다음 토큰 예측을 설명한다.
- encoder와 decoder의 역할 및 정보 흐름을 구분한다.
- Teacher Forcing을 사용하는 학습과 자기 출력을 재입력하는 추론의 차이를 설명한다.
- Attention의 score, softmax weight, context vector 계산을 설명한다.

## 1. 언어 모델이란

언어 모델은 토큰 시퀀스의 확률을 모델링한다. 자기회귀 모델은 연쇄법칙에 따라 문장 확률을 이전 토큰 조건부 확률의 곱으로 분해한다.

$$P(y_1,\dots,y_T)=\prod_{t=1}^{T}P(y_t\mid y_{<t})$$

각 시점에서 모델은 vocabulary 전체에 대한 logits를 내고 softmax가 확률 분포를 만든다.

$$P(y_t=i)=\frac{e^{z_i}}{\sum_{j=1}^{V}e^{z_j}}$$

가장 큰 확률의 토큰을 항상 선택하는 greedy decoding 외에도 beam search, temperature sampling, top-k, top-p 같은 생성 전략이 있다. 모델이 학습하는 분포와 실제 토큰을 선택하는 decoding 정책은 구분해야 한다.

## 2. Seq2Seq 구조

Seq2Seq는 입력과 출력 길이가 다른 번역·요약 등에 적합한 encoder–decoder 구조다.

```text
입력 x₁, x₂, …, xₙ → Encoder → 문맥 표현 → Decoder → y₁, y₂, …, yₘ
```

### Encoder

입력을 순서대로 읽으며 은닉 상태를 갱신한다. 초기 Seq2Seq에서는 마지막 encoder 상태가 입력 문장 전체를 대표하는 context vector가 된다.

### Decoder

context vector와 이전 출력 토큰을 바탕으로 다음 토큰 분포를 계산한다. `<BOS>`에서 시작하여 `<EOS>`를 생성하거나 최대 길이에 도달할 때까지 반복한다.

## 3. 학습과 추론

### Teacher Forcing

학습에서는 이전 시점의 모델 예측 대신 실제 정답 토큰을 다음 입력으로 제공할 수 있다. 정답 문맥을 사용하므로 학습이 빠르고 안정적이다.

```text
학습: 정답 yₜ₋₁ → decoder → yₜ 예측
추론: 예측 ŷₜ₋₁ → decoder → yₜ 예측
```

학습과 추론의 입력 조건 차이는 **노출 편향(exposure bias)**을 만든다. 추론 중 한 번의 오류가 이후 입력이 되어 오류가 누적될 수 있다.

### 손실

각 출력 위치에서 정답 토큰에 대한 Cross Entropy를 계산하고 합 또는 평균한다. Padding 위치는 실제 정답이 아니므로 mask로 손실에서 제외해야 한다.

## 4. 고정 길이 문맥의 병목

초기 Seq2Seq는 입력 전체를 하나의 고정 길이 벡터에 압축한다. 입력이 길수록 다음 문제가 커진다.

- 앞부분의 세부 정보가 마지막 상태까지 보존되기 어렵다.
- 모든 출력 시점이 동일한 context vector에 의존한다.
- 출력마다 필요한 입력 위치가 달라도 선택적으로 참고하기 어렵다.

이것이 encoder 자체가 정보를 전혀 못 담는다는 뜻은 아니다. 핵심은 **모든 정보를 하나의 병목으로 통과시키는 구조적 부담**이다.

## 5. Attention의 동작

Attention decoder는 하나의 마지막 상태만 받지 않고 encoder의 모든 상태 $h_1,\dots,h_n$을 보관한다.

### 1단계: 관련도 점수

현재 decoder 상태 $s_{t-1}$와 각 encoder 상태 $h_i$의 관련도를 계산한다.

$$e_{t,i}=\operatorname{score}(s_{t-1},h_i)$$

Luong dot attention에서는 두 벡터의 내적을 사용할 수 있다.

$$e_{t,i}=s_{t-1}^{\top}h_i$$

### 2단계: Attention weight

입력 위치 방향으로 softmax를 적용한다.

$$\alpha_{t,i}=\frac{\exp(e_{t,i})}{\sum_j\exp(e_{t,j})}$$

각 $\alpha_{t,i}\ge0$이고 $\sum_i\alpha_{t,i}=1$이다.

### 3단계: 문맥 벡터

$$c_t=\sum_i\alpha_{t,i}h_i$$

$c_t$는 현재 출력에 필요한 입력 정보를 강조한 가중합이다. decoder는 $c_t$와 자신의 상태를 결합해 다음 토큰을 예측한다.

```text
Encoder states: h₁  h₂  h₃  h₄
                 ↘  ↓  ↙
Decoder state sₜ → 점수 → softmax → 가중합 cₜ → 다음 토큰
```

> [!important] 0820 포인트
> Attention은 중요한 토큰 하나만 고르는 hard selection이 아니다. 일반적인 soft attention은 모든 value의 가중합이며, weight는 현재 query에 따라 매 출력 시점 달라진다.

## 6. 장점과 한계

### 장점

- 출력 시점마다 다른 입력 위치를 참고한다.
- 긴 입력의 고정 길이 병목을 완화한다.
- Attention map으로 입력–출력 대응 관계를 어느 정도 관찰할 수 있다.
- 멀리 떨어진 정보에 더 직접적으로 접근한다.

### 한계

- RNN encoder와 decoder를 사용하면 시간축 계산은 여전히 순차적이다.
- 모든 attention weight가 사람에게 충실한 설명을 제공하는 것은 아니다.
- 입력과 출력 길이가 길면 점수 계산량이 증가한다.
- 생성 중 이전 예측에 의존하므로 노출 편향은 별도 문제로 남는다.

## 7. 평가 대비 핵심

| 비교 | 핵심 차이 |
|---|---|
| Encoder / Decoder | 입력 표현 생성 / 조건부 출력 생성 |
| 학습 / 추론 | 정답 이전 토큰 사용 가능 / 모델의 이전 예측 사용 |
| 고정 context / Attention | 하나의 압축 벡터 / 출력마다 동적인 가중합 |
| score / weight / context | 원시 관련도 / 정규화된 비율 / value의 가중합 |

> [!question] 예상 문제
> 1. Seq2Seq에서 encoder와 decoder의 역할을 설명하라.
> 2. Teacher Forcing의 장점과 노출 편향을 설명하라.
> 3. 고정 길이 context vector가 긴 문장에서 병목이 되는 이유는 무엇인가?
> 4. Attention을 `score → softmax → weighted sum` 순서로 설명하라.

## 주요 개념 연결

- [[01 워드 임베딩과 순환신경망]]: Seq2Seq encoder와 decoder의 기본 순환 구조를 이해한다.
- [[03 Self-Attention과 Transformer]]: encoder–decoder attention을 한 시퀀스 내부의 토큰 관계 계산으로 확장한다.
- [[04 사전학습 기반 언어 모델]]: encoder-only, decoder-only, encoder–decoder 사전학습 구조로 연결한다.
