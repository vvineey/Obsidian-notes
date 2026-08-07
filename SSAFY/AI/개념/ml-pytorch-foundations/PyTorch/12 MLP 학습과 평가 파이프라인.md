---
date: 2026-08-07
course: SSAFY AI
title: 12 MLP 학습과 평가 파이프라인
status: needs-review
repository: ml-pytorch-foundations
tags:
  - course/ssafy-ai
  - type/concept-note
  - repo/ml-pytorch-foundations
  - ai/mlp
  - ai/model-training
  - ai/model-evaluation
concepts:
  - MLP
  - ReLU
  - Dropout
  - Cross Entropy
  - Optimizer
  - Early Stopping
related:
  - "[[08 신경망 모델]]"
  - "[[09 신경망 적합]]"
  - "[[10 PyTorch 텐서와 Autograd]]"
  - "[[11 Dataset과 DataLoader]]"
---

# 12 MLP 학습과 평가 파이프라인

> [!summary] 핵심 정리
> MLP는 선형변환과 비선형 활성화를 여러 층으로 합성해 입력을 클래스 logits로 변환한다. 학습 루프는 순전파·손실·역전파·갱신을 반복하고, 평가 루프는 파라미터를 바꾸지 않은 채 일반화 성능을 측정한다. Validation은 모델 선택에, test는 최종 보고에 사용한다.

## 목차

- [[#1. MLP의 계산 구조]]
- [[#2. ReLU와 Dropout]]
- [[#3. Cross Entropy와 logits]]
- [[#4. 학습 루프]]
- [[#5. 평가 루프]]
- [[#6. Early Stopping과 체크포인트]]
- [[#7. 지표와 오류 분석]]
- [[#8. 반드시 구분할 개념]]
- [[#주요 개념 연결]]

## 학습 목표

- MLP의 입력·은닉·출력 shape를 추적한다.
- logits와 확률, 예측 클래스를 구분한다.
- 학습 루프와 평가 루프의 상태 변화를 설명한다.
- Validation 기반 early stopping과 test 평가를 구분한다.

## 1. MLP의 계산 구조

Digits 분류 모델은 다음 함수를 학습한다.

```text
(B, 64)
→ Linear(64, 128) → ReLU → Dropout
→ Linear(128, 64) → ReLU → Dropout
→ Linear(64, 10)
→ (B, 10) logits
```

선형층은 표현 차원을 바꾸고, ReLU는 비선형성을 추가한다. 출력층의 10개 값은 숫자 클래스 각각의 상대적인 점수다.

## 2. ReLU와 Dropout

ReLU는 음수를 0으로 만들고 양수는 유지한다. 활성화 함수가 없다면 여러 선형층을 합쳐도 하나의 선형변환과 같아져 복잡한 결정 경계를 학습하기 어렵다.

Dropout은 학습 중 일부 활성화를 무작위로 0으로 만들어 특정 뉴런 조합에 지나치게 의존하는 것을 줄인다. 평가 시에는 모든 뉴런을 사용해야 하므로 `model.eval()`이 필요하다.

## 3. Cross Entropy와 logits

다중분류 cross entropy는 logits와 정수 라벨을 입력으로 받는다. PyTorch 구현은 내부에서 log-softmax를 안정적으로 계산하므로 출력층에서 softmax를 먼저 적용하지 않는다.

예측 클래스는 다음과 같이 구한다.

```python
preds = logits.argmax(dim=1)
```

## 4. 학습 루프

```text
model.train()
→ batch를 device로 이동
→ logits = model(xb)
→ optimizer.zero_grad()
→ loss 계산
→ loss.backward()
→ optimizer.step()
→ 통계 누적
```

`backward()`는 기울기를 계산하고 `step()`은 그 기울기로 파라미터를 갱신한다. 두 단계의 책임은 다르다.

## 5. 평가 루프

```text
model.eval()
→ torch.no_grad()
→ batch를 device로 이동
→ logits와 loss 계산
→ 예측과 정답 누적
```

평가에서는 파라미터를 갱신하지 않는다. `eval()`은 Dropout 같은 레이어의 동작을 바꾸고, `no_grad()`는 계산 그래프 생성을 막는다.

## 6. Early Stopping과 체크포인트

Validation 성능이 개선될 때 모델의 `state_dict`를 저장한다. 지정된 patience 동안 개선이 없으면 학습을 멈추고 최적 체크포인트를 복원한다. Test 결과를 보고 checkpoint를 선택하면 test가 모델 선택에 누출된다.

> [!warning] 개선 조건
> loss와 accuracy를 동시에 추적할 때 “둘 중 하나만 개선돼도 저장”하면 서로 다른 epoch의 최솟값과 최댓값이 한 쌍처럼 기록될 수 있다. 어떤 지표를 우선할지 명시하거나 복합 규칙을 일관되게 적용해야 한다.

## 7. 지표와 오류 분석

- Loss: 모델 출력 분포와 정답의 차이
- Accuracy: 전체 중 맞춘 비율
- Precision·Recall·F1: 클래스별 오류 특성
- Confusion Matrix: 어떤 숫자 쌍을 자주 혼동하는지 확인
- Train/Validation 곡선: 과소적합과 과적합 진단

평균 loss는 `loss.item() * batch_size`를 누적한 뒤 전체 샘플 수로 나눠야 마지막 작은 배치까지 올바르게 가중된다.

## 8. 반드시 구분할 개념

| 구분 | 학습 | 평가 |
|---|---|---|
| 모드 | `model.train()` | `model.eval()` |
| 그래프 | 생성 | `no_grad()`로 비활성화 |
| Dropout | 활성화 | 비활성화 |
| 역전파 | 수행 | 수행하지 않음 |
| 파라미터 갱신 | 수행 | 수행하지 않음 |

## 주요 개념 연결

- [[08 신경망 모델]]: MLP의 층과 비선형 표현을 설명한다.
- [[09 신경망 적합]]: 경사하강법과 역전파의 이론적 흐름을 제공한다.
- [[10 PyTorch 텐서와 Autograd]]: Tensor 계약과 자동 미분을 구현한다.
- [[11 Dataset과 DataLoader]]: 학습·평가 루프에 배치를 공급한다.
- [[01 손글씨 숫자 분류 MLP 구현 가이드]]: 전체 개념을 하나의 Digits 분류 실습으로 연결한다.
