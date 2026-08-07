---
date: 2026-08-07
course: SSAFY AI
title: 11 Dataset과 DataLoader
status: approved
repository: machine-learning-and-pytorch
tags:
  - course/ssafy-ai
  - type/concept-note
  - repo/machine-learning-and-pytorch
  - ai/pytorch
  - data/pipeline
concepts:
  - Dataset
  - TensorDataset
  - DataLoader
  - Batch
  - Shuffle
related:
  - "[[10 PyTorch 텐서와 Autograd]]"
  - "[[12 MLP 학습과 평가 파이프라인]]"
---

# 11 Dataset과 DataLoader

> [!summary] 핵심 정리
> Dataset은 한 샘플을 어떤 방식으로 꺼낼지 정의하고, DataLoader는 Dataset을 배치로 묶어 반복 가능한 학습 입력으로 제공한다. 데이터 자체와 데이터를 공급하는 정책을 분리하면 batch size, shuffle과 병렬 로딩을 모델 코드와 독립적으로 바꿀 수 있다.

## 목차

- [[#1. Dataset]]
- [[#2. TensorDataset]]
- [[#3. DataLoader]]
- [[#4. train과 evaluation loader]]
- [[#5. 첫 배치 검증]]
- [[#6. 반드시 구분할 개념]]
- [[#주요 개념 연결]]

## 학습 목표

- Dataset과 DataLoader의 책임을 구분한다.
- feature와 label Tensor를 `TensorDataset`으로 묶는다.
- train loader의 shuffle 필요성을 설명한다.
- 첫 배치로 shape와 dtype 계약을 검증한다.

## 1. Dataset

Dataset은 전체 데이터를 저장하는 형식이면서 인덱스 하나로 샘플 하나를 반환하는 인터페이스다. 일반적인 반환값은 `(feature, label)`이다.

## 2. TensorDataset

이미 Tensor로 변환된 feature와 label은 `TensorDataset`으로 묶을 수 있다. 첫 번째 차원의 길이는 서로 같아야 한다.

```python
train_ds = TensorDataset(X_train_t, y_train_t)
```

인덱스 `i`를 사용하면 `X_train_t[i]`와 `y_train_t[i]`가 하나의 샘플로 함께 반환된다.

## 3. DataLoader

DataLoader는 Dataset을 반복하면서 여러 샘플을 하나의 배치로 묶는다.

```python
train_loader = DataLoader(train_ds, batch_size=32, shuffle=True)
```

batch size가 32라면 MLP 입력은 보통 `(32, 64)`, 라벨은 `(32,)`가 된다. 마지막 배치는 전체 샘플 수에 따라 더 작을 수 있다.

## 4. train과 evaluation loader

- Train: 매 epoch의 샘플 순서를 섞어 특정 순서에 대한 편향을 줄인다.
- Validation/Test: 일반적으로 shuffle하지 않아 평가와 오류 추적을 단순하게 한다.
- Validation: 모델 선택과 early stopping에 사용한다.
- Test: 모든 선택이 끝난 뒤 최종 일반화 성능을 한 번 평가한다.

## 5. 첫 배치 검증

전체 학습 전에 첫 배치를 꺼내 다음을 확인한다.

```python
xb, yb = next(iter(train_loader))
print(xb.shape, xb.dtype)
print(yb.shape, yb.dtype)
```

이 검사는 모델 forward보다 앞서 데이터 계약 오류를 발견하게 해준다.

## 6. 반드시 구분할 개념

| 개념 | 책임 |
|---|---|
| Dataset | 샘플 저장과 인덱싱 |
| DataLoader | 배치, shuffle과 반복 |
| Batch | 한 번의 forward/backward에 사용하는 샘플 묶음 |
| Epoch | train Dataset 전체를 한 번 순회한 단위 |
| Iteration | 배치 하나를 처리한 학습 단계 |

## 주요 개념 연결

- [[10 PyTorch 텐서와 Autograd]]: DataLoader가 공급하는 값의 자료형과 device 계약을 확인한다.
- [[12 MLP 학습과 평가 파이프라인]]: loader의 반복 결과가 학습·평가 루프의 입력이 된다.
- [[01 손글씨 숫자 분류 MLP 구현 가이드]]: TODO 2에서 TensorDataset과 DataLoader를 직접 구성한다.

