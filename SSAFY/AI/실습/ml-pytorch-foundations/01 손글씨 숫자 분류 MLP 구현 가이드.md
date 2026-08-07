---
date: 2026-08-07
course: SSAFY AI
title: 01 손글씨 숫자 분류 MLP 구현 가이드
status: planned
repository: ml-pytorch-foundations
tags:
  - course/ssafy-ai
  - type/practice-guide
  - repo/ml-pytorch-foundations
  - ai/pytorch
  - ai/mlp
  - ai/classification
concepts:
  - "[[08 신경망 모델]]"
  - "[[09 신경망 적합]]"
  - "[[10 PyTorch 텐서와 Autograd]]"
  - "[[11 Dataset과 DataLoader]]"
  - "[[12 MLP 학습과 평가 파이프라인]]"
sources:
  - Chapter_1/Chapter 1-2/Practice_3/(실습-문제) 1-2_MLP 구현.ipynb
prerequisites:
  - "[[03 지도학습 - 회귀와 분류, 평가와 일반화]]"
  - "[[09 신경망 적합]]"
  - "[[10 PyTorch 텐서와 Autograd]]"
---

# 01 손글씨 숫자 분류 MLP 구현 가이드

> [!summary] 실습의 핵심
> 8×8 손글씨 숫자 이미지를 64차원 벡터로 변환하고, PyTorch의 Dataset·DataLoader와 MLP 학습 루프를 연결해 0~9를 분류한다. 핵심 흐름은 **데이터 분할 → train 기준 표준화 → Tensor와 배치 구성 → MLP 순전파 → 역전파와 파라미터 갱신 → validation 기반 모델 선택 → test 평가**다.

## 실습 결과

8×8 손글씨 숫자 이미지를 입력받아 0~9를 분류하는 다층 퍼셉트론을 PyTorch로 구현한다. 데이터 분할부터 배치 구성, 모델 정의, 학습·평가 루프, 조기 종료, 체크포인트, 혼동행렬까지 하나의 재현 가능한 분류 파이프라인을 완성하는 것이 목표다.

## 문제와 데이터

- 데이터: `sklearn.datasets.load_digits`
- 샘플: 1,797개
- 입력: 8×8 이미지를 펼친 64차원 `float32` 벡터
- 타깃: 0~9의 정수 클래스, `int64`
- 분할: train 80%, validation 10%, test 10%
- 출력: 클래스별 확률을 만들기 전의 logits 10개

## 선행학습 판정

### 필수

- NumPy 배열의 shape와 dtype을 확인할 수 있어야 한다.
- train/validation/test의 역할을 구분할 수 있어야 한다.
- scaler를 train에만 fit해야 하는 이유를 설명할 수 있어야 한다.
- 다중분류 logits `(B, 10)`에서 `argmax(dim=1)`로 예측을 구할 수 있어야 한다.

### 권장

- Python 클래스, 상속, `super().__init__()`
- 손실 함수, 경사하강법, 연쇄법칙의 직관
- 1-1 Wine 분류의 표준화와 classification report

### 선택

- PCA, K-Means
- 선형회귀 수학의 상세 유도

## 전체 실습 흐름

```mermaid
flowchart LR
    A["Digits 데이터"] --> B["80/10/10 분할"]
    B --> C["Train 기준 표준화"]
    C --> D["Tensor 변환"]
    D --> E["Dataset과 DataLoader"]
    E --> F["64-128-64-10 MLP"]
    F --> G["Cross Entropy와 Adam"]
    G --> H["Validation과 Early Stopping"]
    H --> I["Test 평가와 혼동행렬"]
```

## 핵심 개념

> [!info] 연결 노트
> [[08 신경망 모델|신경망 구조]]에서 선형층과 활성화 함수의 역할을, [[09 신경망 적합|신경망 적합]]에서 손실·역전파·optimizer의 관계를 먼저 확인한다. 구현 단계에서는 [[10 PyTorch 텐서와 Autograd]], [[11 Dataset과 DataLoader]], [[12 MLP 학습과 평가 파이프라인]]을 함께 참고한다.

### Tensor와 Autograd

Tensor는 NumPy 배열과 유사하지만 device 이동과 자동 미분을 지원한다. `requires_grad=True`인 텐서로 만든 연산 그래프는 `backward()` 호출 시 기울기를 계산한다. PyTorch의 optimizer는 이 기울기를 사용해 파라미터를 갱신한다.

### MLP 구조

모델 계약은 다음과 같다.

| 단계 | 연산 | 출력 shape |
|---|---|---|
| 입력 | flatten된 이미지 | `(B, 64)` |
| 은닉층 1 | Linear + ReLU + Dropout | `(B, 128)` |
| 은닉층 2 | Linear + ReLU + Dropout | `(B, 64)` |
| 출력층 | Linear | `(B, 10)` |

출력층에는 ReLU나 softmax를 붙이지 않는다. Cross entropy가 내부에서 안정적인 log-softmax 계산을 처리한다.

### Dataset과 DataLoader

`TensorDataset`은 feature와 label 텐서를 같은 인덱스로 묶고, `DataLoader`는 이를 배치 단위로 제공한다. Train loader만 shuffle하고 validation/test는 재현 가능한 평가를 위해 일반적으로 shuffle하지 않는다.

### 학습 모드와 평가 모드

- `model.train()`: Dropout을 활성화한다.
- `model.eval()`: Dropout을 비활성화해 일관된 추론을 수행한다.
- `torch.no_grad()`: 평가 시 미분 그래프 생성을 막아 메모리와 계산을 절약한다.

## TODO 지도

1. 두 번의 `train_test_split`으로 80/10/10을 구성하고 train 기준으로 표준화한다.
2. 모든 배열을 올바른 dtype의 Tensor로 바꾸고 Dataset·DataLoader를 만든다.
3. 두 은닉층을 가진 MLP를 `nn.Sequential`로 구현한다.
4. device 이동, forward, gradient 초기화, loss, backward, optimizer step을 포함한 학습 함수를 완성한다.
5. 평가 모드와 no-grad를 적용해 loss, accuracy, 예측, 정답을 반환한다.

## 학습 생명주기

```text
model.train()
→ batch를 device로 이동
→ logits 계산
→ optimizer.zero_grad()
→ cross entropy 계산
→ loss.backward()
→ optimizer.step()
→ 샘플 수로 가중한 loss와 accuracy 누적
```

검증 loss 또는 accuracy가 개선되면 `state_dict`를 체크포인트에 저장한다. 개선이 일정 epoch 동안 없으면 조기 종료하고, 저장된 최적 모델을 복원한 뒤 test를 마지막 한 번만 평가한다.

## 평가 전략

- 기본 지표: loss와 accuracy
- 클래스별 평가: precision, recall, F1
- 오류 분석: confusion matrix
- 과적합 진단: train과 validation 곡선 간 간격
- 모델 선택: validation만 사용
- 최종 보고: test는 모델 선택이 끝난 후 한 번 사용

## 자주 발생하는 오류

| 증상 | 가능한 원인 | 확인 방법 |
|---|---|---|
| Cross entropy dtype 오류 | label이 float | `yb.dtype == torch.int64` 확인 |
| 행렬곱 shape 오류 | 입력이 `(B, 8, 8)` | 모델 입력 전 `(B, 64)` 확인 |
| device mismatch | 모델과 batch 위치 불일치 | 각 `.device` 출력 |
| validation 성능 변동 | `eval()` 누락 | 평가 함수 시작 부분 확인 |
| loss가 줄지 않음 | `backward` 또는 `step` 누락 | 학습 순서 점검 |
| 평가 누수 | 전체 데이터로 scaler fit | scaler fit 대상 확인 |
| 잘못된 loss 평균 | 배치 평균의 단순 평균 | `loss * batch_size` 누적 확인 |

## 실행 계획

- [ ] 환경에서 torch, sklearn, seaborn import 확인
- [ ] 데이터 shape와 클래스 분포 확인
- [ ] split과 표준화 구현
- [ ] 첫 DataLoader 배치의 shape·dtype 확인
- [ ] 임의 배치로 모델 출력 `(B, 10)` 확인
- [ ] 한 배치 학습으로 파이프라인 디버깅
- [ ] 한 epoch 학습·평가 실행
- [ ] 전체 학습과 early stopping 실행
- [ ] 최적 체크포인트 복원
- [ ] test 결과와 혼동행렬 해석

## 실험 기록

| 항목 | 값 |
|---|---|
| Seed | 42 |
| Model | 64-128-64-10 |
| Dropout | 0.2 |
| Optimizer | Adam |
| Learning rate | 1e-3 |
| Weight decay | 1e-4 |
| Batch size | 32 |
| Best epoch |  |
| Validation accuracy |  |
| Test accuracy |  |
| 주요 혼동 클래스 |  |

## 소스와 공개 범위

원본 실습 노트북에는 교육 콘텐츠의 복사·보관·전송·공개를 제한하는 경고가 있다. 퍼블릭 GitHub 저장소에는 원본 문제·정답 노트북을 올리지 않고, 직접 작성한 설명·코드·테스트·실험 결과만 검토 후 추가한다.
