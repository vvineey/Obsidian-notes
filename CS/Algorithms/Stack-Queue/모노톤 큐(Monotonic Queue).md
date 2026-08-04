---
tags:
  - cs/algorithm
  - type/algorithm-note
  - algorithm/stack-queue
  - algorithm/sliding-window
  - data-structure/deque
  - platform/programmers
importance: high
status: seed
aliases:
  - Monotonic Queue
  - Monotonic Deque
  - 모노토닉 큐
---

# 모노톤 큐(Monotonic Queue)

## 한 줄 정의

덱 내부의 값을 단조 증가 또는 단조 감소 상태로 유지해, 이동하는 구간의 최솟값이나 최댓값을 빠르게 구하는 자료구조 패턴이다.

## 언제 사용하는가

- 길이 `k`인 모든 연속 구간의 최댓값 또는 최솟값을 구할 때
- 매 구간을 순회하는 `O(nk)` 풀이가 시간 제한을 넘을 때
- 우선순위 큐의 `O(n log k)`보다 더 빠른 `O(n)` 풀이가 필요할 때
- 윈도우를 벗어난 원소와 더 이상 최댓값·최솟값 후보가 될 수 없는 원소를 제거할 수 있을 때

## 핵심 아이디어

구간 최댓값을 구하는 감소 모노톤 큐는 다음 두 조건을 유지한다.

1. 덱에는 현재 윈도우 안에 있는 인덱스만 보관한다.
2. 해당 인덱스의 값은 덱의 앞에서 뒤로 갈수록 감소한다.

새 값보다 작거나 같은 기존 값은 앞으로도 최댓값이 될 수 없다. 새 값이 더 크고 더 늦게 만료되기 때문이다. 따라서 덱의 뒤에서 제거한다.

덱의 맨 앞에는 항상 현재 윈도우의 최댓값 인덱스가 남는다.

## 동작 과정

길이 `k`인 윈도우의 최댓값을 구하는 경우:

1. 덱 앞에서 현재 윈도우를 벗어난 인덱스를 제거한다.
2. 덱 뒤에서 현재 값보다 작거나 같은 값의 인덱스를 제거한다.
3. 현재 인덱스를 덱 뒤에 넣는다.
4. 윈도우가 완성되면 덱 앞의 값을 정답에 기록한다.
5. 배열 끝까지 반복한다.

## 예시

배열이 `[1, 3, -1, -3, 5]`, `k = 3`인 경우:

| 윈도우 | 감소 덱이 보관하는 값 | 최댓값 |
| --- | --- | --- |
| `[1, 3, -1]` | `[3, -1]` | `3` |
| `[3, -1, -3]` | `[3, -1, -3]` | `3` |
| `[-1, -3, 5]` | `[5]` | `5` |

덱에는 윈도우의 모든 값이 아니라 앞으로 최댓값이 될 가능성이 있는 후보만 남는다.

## 시간 복잡도

| 구현 방식 | 시간 복잡도 | 공간 복잡도 | 특징 |
| --- | --- | --- | --- |
| 구간마다 직접 탐색 | `O(nk)` | `O(1)` | 큰 입력에서 비효율적 |
| 우선순위 큐 | `O(n log k)` | `O(k)` | 만료 원소 처리가 필요 |
| 모노톤 큐 | `O(n)` | `O(k)` | 각 인덱스가 한 번 삽입·삭제 |

## Java 예시 코드

```java
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public int[] solution(int[] numbers, int k) {
        if (k <= 0 || k > numbers.length) {
            return new int[0];
        }

        int[] answer = new int[numbers.length - k + 1];
        Deque<Integer> deque = new ArrayDeque<>();
        int answerIndex = 0;

        for (int i = 0; i < numbers.length; i++) {
            // 현재 윈도우의 왼쪽 경계를 벗어난 인덱스 제거
            while (!deque.isEmpty() && deque.peekFirst() <= i - k) {
                deque.pollFirst();
            }

            // 현재 값보다 작거나 같은 값은 최댓값 후보에서 제거
            while (!deque.isEmpty()
                    && numbers[deque.peekLast()] <= numbers[i]) {
                deque.pollLast();
            }

            deque.addLast(i);

            // 길이 k의 첫 윈도우가 완성된 뒤부터 결과 기록
            if (i >= k - 1) {
                answer[answerIndex++] = numbers[deque.peekFirst()];
            }
        }

        return answer;
    }
}
```

## 코드 해설

- 덱에는 값이 아니라 인덱스를 저장해야 윈도우 이탈 여부를 판단할 수 있다.
- 덱의 앞에서는 범위를 벗어난 인덱스를 제거한다.
- 덱의 뒤에서는 현재 값보다 작거나 같은 후보를 제거한다.
- 같은 값도 제거하고 최신 인덱스를 남기면 더 늦게 만료되어 관리하기 쉽다.
- 각 인덱스는 덱에 한 번 들어가고 최대 한 번 제거되므로 전체 시간 복잡도는 `O(n)`이다.
- Java에서는 `LinkedList`보다 `ArrayDeque`가 일반적인 덱 구현에 적합하다.

## 실수하기 쉬운 지점

- 값을 저장해 만료된 원소의 위치를 판단하지 못하는 것
- 최댓값을 구하면서 증가 덱을 만드는 등 단조 방향을 반대로 잡는 것
- 덱 앞의 만료 원소를 제거하지 않는 것
- 윈도우가 완성되기 전에 결과를 기록하는 것
- `i - k`와 `i - k + 1` 경계를 혼동하는 것
- 모노톤 큐가 정렬된 전체 데이터를 보관한다고 오해하는 것
- `Deque<Integer>`의 박싱 비용이 큰 상황에서 성능 차이를 고려하지 않는 것

## 관련 자료구조 / 개념 링크

- [[슬라이딩 윈도우(Sliding Window)]]
- [[덱(Deque)]]
- [[모노톤 스택(Monotonic Stack)]]
- [[우선순위 큐(Priority Queue)]]

## 관련 문제(프로그래머스)

| 난이도 | 문제 | 핵심 아이디어 | 상태 | 링크 |
| --- | --- | --- | --- | --- |
| Lv.3 | 징검다리 건너기 | 길이 `k` 구간의 최댓값 중 최솟값을 감소 덱으로 계산 | 추천 | [Programmers](https://school.programmers.co.kr/learn/courses/30/lessons/64062) |
| Lv.2 | 뒤에 있는 큰 수 찾기 | 모노톤 스택으로 단조성 유지 연습 | 추천 | [Programmers](https://school.programmers.co.kr/learn/courses/30/lessons/154539) |
| Lv.2 | 주식가격 | 아직 답이 정해지지 않은 인덱스를 단조 스택으로 관리 | 추천 | [Programmers](https://school.programmers.co.kr/learn/courses/30/lessons/42584) |
| Lv.2 | 큰 수 만들기 | 작은 값을 뒤에서 제거하는 모노톤 스택·그리디 연습 | 추천 | [Programmers](https://school.programmers.co.kr/learn/courses/30/lessons/42883) |

`징검다리 건너기`가 모노톤 큐의 직접 연습 문제다. 나머지 세 문제는 덱 뒤에서 지배당한 후보를 제거한다는 감각을 익히기 위한 모노톤 스택 선행 문제다.

## 문제 풀이 접근법

1. 각 윈도우의 최댓값·최솟값을 매번 다시 찾고 있는지 확인한다.
2. 최댓값이면 감소 덱, 최솟값이면 증가 덱을 선택한다.
3. 덱에는 값 대신 인덱스를 저장한다.
4. 앞에서는 만료된 인덱스, 뒤에서는 지배당한 후보를 제거한다.
5. 덱의 앞이 현재 윈도우의 정답이라는 불변식을 확인한다.

## 마지막 요약

| 구분 | 핵심 |
| --- | --- |
| 사용 조건 | 이동 구간의 최댓값 또는 최솟값을 반복 계산할 때 |
| 핵심 자료구조 | `ArrayDeque<Integer>` |
| 시간 복잡도 | `O(n)` |
| 공간 복잡도 | `O(k)` |
| 대표 실수 | 값 저장, 만료 제거 누락, 단조 방향 혼동 |
| 대표 문제 유형 | 슬라이딩 윈도우 최댓값·최솟값 |

## 복습 로그

- [ ] 감소 덱에서 작은 값을 제거해도 되는 이유를 설명할 수 있다.
- [ ] 덱에 값이 아닌 인덱스를 저장해야 하는 이유를 설명할 수 있다.
- [ ] 모노톤 큐와 모노톤 스택의 차이를 설명할 수 있다.
