# Heap 01 - More Spicy

## Problem

- Platform: Programmers
- Category: Heap
- Title: More Spicy
- Korean title: 더 맵게

## Problem Summary

모든 음식의 스코빌 지수를 `K` 이상으로 만들어야 한다.

한 번 섞을 때마다 가장 맵지 않은 음식 두 개를 꺼내서 새 음식을 만든다.

```text
새로운 스코빌 지수 = 가장 맵지 않은 음식 + (두 번째로 맵지 않은 음식 * 2)
```

이 과정을 모든 음식의 스코빌 지수가 `K` 이상이 될 때까지 반복한다.

만약 모든 음식을 `K` 이상으로 만들 수 없다면 `-1`을 반환한다.

## Quick Summary

- 매번 가장 작은 스코빌 지수 두 개를 꺼내야 한다.
- 전체 정렬 상태가 필요한 것이 아니라 현재 최솟값만 빠르게 필요하다.
- C++에서는 min heap 형태의 `priority_queue`를 사용한다.
- 가장 작은 값이 이미 `K` 이상이면 모든 음식이 조건을 만족하므로 종료한다.

## Thinking Process

### Heap And Priority Queue

이 문제는 힙 문제지만, C++에서는 보통 `priority_queue`를 사용해서 푼다.

관계는 다음처럼 이해하면 된다.

```text
heap = 우선순위가 높은 값을 빠르게 꺼내기 위한 자료구조
priority_queue = heap을 이용해서 만든 C++ 자료구조
```

C++의 기본 `priority_queue<int>`는 max heap이다.

```cpp
priority_queue<int> pq;
```

이 경우 `pq.top()`은 항상 가장 큰 값이다.

하지만 이 문제는 가장 작은 음식 두 개를 계속 꺼내야 하므로 min heap이 필요하다.

```cpp
priority_queue<int, vector<int>, greater<int>> pq;
```

이렇게 만들면 `pq.top()`은 항상 가장 작은 값이 된다.

### Min Heap Syntax

```cpp
priority_queue<int, vector<int>, greater<int>> pq;
```

각 부분의 의미는 다음과 같다.

- `int`: queue에 넣을 값의 타입
- `vector<int>`: 내부에서 값을 저장하는 컨테이너
- `greater<int>`: 큰 값의 우선순위를 낮추는 비교 기준

즉 `greater<int>`를 쓰면 큰 값이 뒤로 밀리고, 작은 값이 `top()`에 오게 된다.

정리하면:

```text
less<int>    -> 큰 값이 top -> max heap
greater<int> -> 작은 값이 top -> min heap
```

### Heap Structure

힙은 보통 완전 이진 트리로 이해할 수 있다.

min heap의 규칙:

```text
부모 <= 자식
```

그래서 맨 위, 즉 root에는 항상 가장 작은 값이 온다.

max heap의 규칙:

```text
부모 >= 자식
```

그래서 root에는 항상 가장 큰 값이 온다.

중요한 점은 힙이 전체 정렬을 보장하지는 않는다는 것이다.

예를 들어 min heap 내부가 다음처럼 저장될 수 있다.

```text
[1, 3, 2, 7, 5, 4]
```

이 배열은 정렬된 배열은 아니다.

하지만 부모가 자식보다 작거나 같은 heap 조건은 만족한다. 따라서 보장되는 것은 전체 순서가 아니라 `top()`에 있는 값이다.

### Priority Queue Vs Sorting

우선순위 큐는 정렬과 다르다.

정렬은 전체 원소의 순서를 알 수 있다.

```text
[1, 2, 3, 4, 5, 7]
```

하지만 heap / priority queue는 현재 가장 우선순위가 높은 값만 빠르게 알 수 있다.

- min heap: 현재 가장 작은 값만 보장
- max heap: 현재 가장 큰 값만 보장

이 문제에서는 전체 정렬 상태가 필요한 것이 아니라, 매번 가장 작은 값 두 개만 필요하다.

그래서 정렬보다 min heap을 사용하는 것이 자연스럽다.

## My Approach

처음에 모든 스코빌 지수를 min heap에 넣는다.

그 다음 가장 작은 값이 `K` 이상인지 확인한다.

- 가장 작은 값이 이미 `K` 이상이면 모든 음식이 조건을 만족하므로 종료한다.
- 가장 작은 값이 `K`보다 작으면 가장 작은 음식 두 개를 꺼낸다.
- 두 음식을 섞어서 새 스코빌 지수를 만든다.
- 새 스코빌 지수를 다시 min heap에 넣는다.
- 섞은 횟수를 1 증가시킨다.

더 이상 두 개를 꺼낼 수 없는데 `K` 미만인 음식이 남아 있다면 `-1`이다.

## Key Point

우선순위 큐의 활용법을 알면 쉽게 풀 수 있는 문제다.

이 문제의 핵심은 매번 “가장 작은 값 두 개”가 필요하다는 점이다.

min heap을 사용하면:

- 가장 작은 값 확인: `top()`
- 가장 작은 값 제거: `pop()`
- 새 값 추가: `push()`

를 효율적으로 처리할 수 있다.

## Accepted Solution

```cpp
#include <string>
#include <vector>
#include <queue>
#include <iostream>

using namespace std;

void printPq(priority_queue<int, vector<int>, greater<int>> pq) {
    cout << "[";
    while (false == pq.empty()) {
        cout << pq.top() << " ";
        pq.pop();
    }
    cout << "]" << endl;
}

int solution(vector<int> scoville, int K) {
    int answer = 0;

    priority_queue<int, vector<int>, greater<int>> pq;
    for (auto i: scoville) {
        pq.push(i);
    }

    while (2 <= pq.size()) {
        int scoville_a = pq.top();

        // 제일 작은 스코빌이 K를 만족한다면 더 돌 필요가 없다.
        if (K <= scoville_a) {
            break;
        }
        pq.pop();

        int scoville_b = pq.top();
        pq.pop();

        int scoville_c = scoville_a + (scoville_b * 2);
        pq.push(scoville_c);
        answer++;
    }

    while (false == pq.empty()) {
        int top = pq.top();
        pq.pop();
        if (top < K) {
            answer = -1;
        }
    }

    return answer;
}
```

## Complexity

음식 개수를 `n`이라고 하면, 각 음식은 heap에 들어가고 섞일 때마다 다시 들어갈 수 있다.

`priority_queue`의 주요 연산은 다음과 같다.

- `top()`: `O(1)`
- `push()`: `O(log n)`
- `pop()`: `O(log n)`

따라서 전체 시간복잡도는 대략 `O(n log n)`으로 볼 수 있다.

공간복잡도는 모든 스코빌 지수를 우선순위 큐에 저장하므로 `O(n)`이다.

## Notes

- 힙은 전체 정렬이 아니라 top만 보장한다.
- min heap에서는 top이 항상 가장 작은 값이다.
- max heap에서는 top이 항상 가장 큰 값이다.
- `greater<int>`를 쓰면 큰 값의 우선순위가 낮아져서 작은 값이 먼저 나온다.
- `pop()`은 값을 반환하지 않으므로, 값을 쓰려면 먼저 `top()`으로 꺼내야 한다.
