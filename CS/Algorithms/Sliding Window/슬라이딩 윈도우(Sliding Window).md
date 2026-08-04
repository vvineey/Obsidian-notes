---
tags:
  - cs/algorithm
  - type/algorithm-note
  - algorithm/sliding-window
  - algorithm/two-pointers
  - platform/programmers
importance: high
status: seed
aliases:
  - Sliding Window
  - 슬라이딩윈도우
---

# 슬라이딩 윈도우(Sliding Window)

## 한 줄 정의

연속된 구간을 이동하면서 새로 들어온 원소는 추가하고 빠져나간 원소는 제거해, 구간 정보를 다시 계산하지 않는 기법이다.

## 언제 사용하는가

- 길이가 `k`인 모든 연속 구간의 합·빈도·최댓값을 구할 때
- 조건을 만족하는 가장 짧거나 긴 연속 구간을 찾을 때
- 문자열이나 배열에서 특정 종류 또는 빈도 조건을 관리할 때
- 구간이 한 칸 이동할 때 이전 계산 결과를 재사용할 수 있을 때

## 핵심 아이디어

슬라이딩 윈도우는 투 포인터의 연속 구간 특화 형태다.

- `right`가 새 원소를 윈도우에 넣는다.
- 조건이 깨지거나 더 줄일 수 있으면 `left`가 기존 원소를 제거한다.
- 현재 구간의 합, 빈도표, 서로 다른 원소 수 등의 상태를 유지한다.

### 고정 크기와 가변 크기

| 종류 | 특징 | 예시 |
| --- | --- | --- |
| 고정 크기 | 윈도우 길이가 항상 `k` | 길이 `k`인 구간의 최대 합 |
| 가변 크기 | 조건에 따라 `left`를 이동 | 합이 `target` 이상인 최소 구간 |
| 빈도 기반 | 맵이나 배열로 구간의 원소 수 관리 | 모든 보석 종류를 포함하는 최소 구간 |

## 동작 과정

### 고정 크기 윈도우

1. 처음 `k`개 원소로 초기 상태를 만든다.
2. 오른쪽 원소 하나를 추가한다.
3. 왼쪽에서 빠지는 원소 하나를 제거한다.
4. 정답을 갱신한다.

### 가변 크기 윈도우

1. `right`를 이동하며 새 원소를 추가한다.
2. 현재 윈도우가 조건을 만족하는지 확인한다.
3. 조건을 만족하는 동안 `left`를 이동해 더 짧게 만들거나 조건을 복구한다.
4. 각 단계에서 정답을 갱신한다.

## 시간 복잡도

| 구현 방식 | 시간 복잡도 | 공간 복잡도 | 특징 |
| --- | --- | --- | --- |
| 고정 길이 합 | `O(n)` | `O(1)` | 합을 증감하여 재사용 |
| 가변 길이 합 | `O(n)` | `O(1)` | 합의 단조성이 필요 |
| 빈도표 기반 | 평균 `O(n)` | `O(U)` | `U`는 서로 다른 값의 수 |
| 매 구간 재계산 | `O(nk)` | 상황에 따라 다름 | 피해야 할 방식 |

## Java 예시 코드

```java
class Solution {
    // 고정 길이 k인 연속 구간 중 최대 합
    public long solution(int[] numbers, int k) {
        if (k <= 0 || k > numbers.length) {
            throw new IllegalArgumentException("유효하지 않은 윈도우 크기");
        }

        long windowSum = 0;

        for (int i = 0; i < k; i++) {
            windowSum += numbers[i];
        }

        long maxSum = windowSum;

        for (int right = k; right < numbers.length; right++) {
            windowSum += (long) numbers[right] - numbers[right - k];
            maxSum = Math.max(maxSum, windowSum);
        }

        return maxSum;
    }

    // 모든 원소가 양수일 때 합이 target 이상인 최소 구간 길이
    public int shortestWindow(int[] numbers, long target) {
        int left = 0;
        int minLength = numbers.length + 1;
        long sum = 0;

        for (int right = 0; right < numbers.length; right++) {
            sum += numbers[right];

            while (sum >= target) {
                minLength = Math.min(minLength, right - left + 1);
                sum -= numbers[left++];
            }
        }

        return minLength == numbers.length + 1 ? 0 : minLength;
    }
}
```

## 코드 해설

- 고정 윈도우는 초기 `k`개 합을 한 번만 계산한다.
- 이후에는 `numbers[right]`를 더하고 `numbers[right - k]`를 빼서 다음 구간을 만든다.
- 가변 윈도우에서는 `right`가 범위를 확장하고, 조건을 만족하는 동안 `left`가 범위를 축소한다.
- 모든 원소가 양수이므로 `left`를 이동하면 합이 감소한다는 단조성이 보장된다.
- 합은 `long`으로 관리해 오버플로우를 방지한다.

## 실수하기 쉬운 지점

- 윈도우가 완성되기 전에 정답을 갱신하는 것
- 빠져나갈 원소의 인덱스를 `right - k + 1`로 잘못 계산하는 것
- 빈도 맵에서 개수가 0이 된 항목을 제거하지 않는 것
- 음수가 섞인 배열에 양수 전용 가변 윈도우를 적용하는 것
- 최소 구간 문제에서 조건을 한 번만 검사하고 `while`로 계속 축소하지 않는 것
- 길이가 같은 정답이 여러 개일 때 시작 인덱스 우선 조건을 놓치는 것

## 관련 자료구조 / 개념 링크

- [[투 포인터(Two Pointers)]]
- [[모노톤 큐(Monotonic Queue)]]
- [[누적합(Prefix Sum)]]
- [[해시 테이블(Hash Table)]]

## 관련 문제(프로그래머스)

| 난이도 | 문제 | 핵심 아이디어 | 상태 | 링크 |
| --- | --- | --- | --- | --- |
| Lv.2 | 할인 행사 | 길이 10의 고정 윈도우에서 상품 빈도 관리 | 추천 | [Programmers](https://school.programmers.co.kr/learn/courses/30/lessons/131127) |
| Lv.2 | 연속된 부분 수열의 합 | 양수 구간 합을 이용한 가변 윈도우 | 추천 | [Programmers](https://school.programmers.co.kr/learn/courses/30/lessons/178870) |
| Lv.2 | 두 큐 합 같게 만들기 | 연결된 수열에서 목표 합을 갖는 구간 탐색 | 추천 | [Programmers](https://school.programmers.co.kr/learn/courses/30/lessons/118667) |
| Lv.3 | 보석 쇼핑 | 모든 종류를 포함하는 최소 가변 윈도우 | 추천 | [Programmers](https://school.programmers.co.kr/learn/courses/30/lessons/67258) |
| Lv.3 | [1차] 추석 트래픽 | 1초 구간에 겹치는 처리 구간의 수 계산 | 추천 | [Programmers](https://school.programmers.co.kr/learn/courses/30/lessons/17676) |

## 문제 풀이 접근법

1. 문제에서 요구하는 대상이 반드시 연속 구간인지 확인한다.
2. 윈도우 길이가 고정인지 조건에 따라 변하는지 구분한다.
3. 윈도우가 보관해야 할 상태를 합·빈도·종류 수 중에서 정한다.
4. 오른쪽 원소를 추가하는 연산과 왼쪽 원소를 제거하는 연산을 대칭적으로 작성한다.
5. 정답 갱신 시점과 동률 조건을 확인한다.

## 마지막 요약

| 구분 | 핵심 |
| --- | --- |
| 사용 조건 | 연속 구간이며 이전 구간의 상태를 재사용할 수 있을 때 |
| 핵심 상태 | `left`, `right`, 합 또는 빈도표 |
| 시간 복잡도 | 보통 `O(n)` |
| 대표 실수 | 제거 연산 누락, 음수가 있는 합 문제에 잘못 적용 |
| 대표 문제 유형 | 고정 구간 집계, 최소·최대 구간, 빈도 조건 |

## 복습 로그

- [ ] 고정 크기와 가변 크기 윈도우의 차이를 설명할 수 있다.
- [ ] 왜 가변 합 윈도우에서 양수 조건이 중요한지 설명할 수 있다.
- [ ] 빈도 맵에서 원소 추가와 제거를 대칭적으로 구현할 수 있다.
