---
date: 2026-08-07
course: SSAFY AI
title: 10 PyTorch 텐서와 Autograd
status: approved
repository: machine-learning-and-pytorch
tags:
  - course/ssafy-ai
  - type/concept-note
  - repo/machine-learning-and-pytorch
  - ai/pytorch
  - ai/autograd
concepts:
  - Tensor
  - Computational Graph
  - Gradient
  - Autograd
related:
  - "[[09 신경망 적합]]"
  - "[[11 Dataset과 DataLoader]]"
  - "[[12 MLP 학습과 평가 파이프라인]]"
---

# 10 PyTorch 텐서와 Autograd

> [!summary] 핵심 정리
> Tensor는 PyTorch의 다차원 배열이며 NumPy와 유사한 연산에 GPU 이동과 자동 미분 기능을 더한다. Autograd는 Tensor 연산으로 만들어진 계산 그래프를 추적하고 `backward()` 호출 시 손실에서 각 파라미터까지의 기울기를 연쇄법칙으로 계산한다.

## 목차

- [[#1. Tensor의 역할]]
- [[#2. shape와 dtype]]
- [[#3. device]]
- [[#4. 계산 그래프와 Autograd]]
- [[#5. 기울기 누적]]
- [[#6. NumPy 변환]]
- [[#7. 반드시 구분할 개념]]
- [[#주요 개념 연결]]

## 학습 목표

- Tensor의 shape, dtype과 device를 확인한다.
- NumPy 배열과 Tensor를 상호 변환한다.
- `requires_grad`, 계산 그래프와 `backward()`의 관계를 설명한다.
- 기울기가 기본적으로 누적되는 이유와 초기화 필요성을 설명한다.

## 1. Tensor의 역할

Tensor는 스칼라, 벡터, 행렬과 그 이상의 차원을 하나의 자료구조로 표현한다. 신경망에서는 입력 배치, 가중치, 중간 활성화, logits와 손실이 모두 Tensor다.

| 예시 | shape | 의미 |
|---|---|---|
| 스칼라 loss | `()` | 하나의 손실값 |
| class label 배치 | `(B,)` | 각 샘플의 정수 라벨 |
| feature 배치 | `(B, D)` | B개 샘플과 D개 특성 |
| 이미지 배치 | `(B, C, H, W)` | 배치·채널·높이·너비 |

## 2. shape와 dtype

연산 오류를 줄이려면 값보다 먼저 계약을 확인한다.

- 입력 feature: 보통 `torch.float32`
- 다중분류 라벨: 보통 `torch.int64`
- MLP 입력: `(batch_size, input_dim)`
- Cross entropy의 logits: `(batch_size, num_classes)`
- Cross entropy의 target: `(batch_size,)`

> [!warning] 흔한 오류
> 라벨을 `float32`로 만들거나 `(B, 1)`로 유지하면 cross entropy의 입력 계약과 맞지 않을 수 있다.

## 3. device

모델과 입력 Tensor는 같은 device에 있어야 한다. CPU Tensor와 CUDA 모델을 함께 연산하면 device mismatch 오류가 발생한다.

```python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
model = model.to(device)
xb, yb = xb.to(device), yb.to(device)
```

## 4. 계산 그래프와 Autograd

`requires_grad=True`인 Tensor가 연산에 참여하면 PyTorch는 결과가 어떤 연산을 통해 만들어졌는지 기록한다. 최종 스칼라 loss에서 `backward()`를 호출하면 그래프를 거꾸로 따라가며 편미분을 계산하고 leaf Tensor의 `.grad`에 저장한다.

```text
입력과 파라미터 → 순전파 → 예측 → 손실 → backward → 각 파라미터의 grad
```

Autograd가 파라미터를 직접 갱신하는 것은 아니다. 계산된 `.grad`를 optimizer가 읽고 `step()`에서 값을 갱신한다.

## 5. 기울기 누적

PyTorch는 여러 loss의 기울기를 합산하는 사용 사례를 지원하기 위해 `.grad`를 자동으로 덮어쓰지 않고 누적한다. 일반적인 미니배치 학습에서는 매 배치마다 `optimizer.zero_grad()`로 이전 기울기를 지운다.

## 6. NumPy 변환

- NumPy → Tensor: `torch.from_numpy(array)`
- CPU Tensor → NumPy: `tensor.numpy()`
- CUDA 또는 gradient Tensor → NumPy: `tensor.detach().cpu().numpy()`

NumPy와 Tensor가 메모리를 공유할 수 있으므로 한쪽의 변경이 다른 쪽에 영향을 주는지 주의한다.

## 7. 반드시 구분할 개념

| 개념 | 역할 |
|---|---|
| `requires_grad` | 연산 추적과 기울기 계산 대상 표시 |
| `backward()` | 그래프를 역방향으로 미분 |
| `.grad` | 계산된 기울기가 저장되는 위치 |
| `no_grad()` | 그래프 생성을 비활성화하는 평가 컨텍스트 |
| `detach()` | 현재 그래프에서 Tensor를 분리 |

## 주요 개념 연결

- [[09 신경망 적합]]: 기울기와 역전파의 이론을 PyTorch 연산으로 연결한다.
- [[11 Dataset과 DataLoader]]: Tensor를 샘플과 배치 단위로 공급한다.
- [[12 MLP 학습과 평가 파이프라인]]: Autograd 결과를 optimizer step으로 연결한다.
- [[01 손글씨 숫자 분류 MLP 구현 가이드]]: 실제 TODO에서 shape·dtype·device 계약을 적용한다.

