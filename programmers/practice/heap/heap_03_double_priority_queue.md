# Heap 03 - Double Priority Queue

## Problem

- Platform: Programmers
- Category: Heap
- Title: Double Priority Queue
- Korean title: 이중 우선순위 큐

## Problem Summary

이중 우선순위 큐는 다음 연산을 처리해야 한다.

```text
I 숫자  -> 숫자를 삽입한다.
D 1    -> 최댓값을 삭제한다.
D -1   -> 최솟값을 삭제한다.
```

모든 연산을 처리한 뒤:

- 큐가 비어 있으면 `[0, 0]`
- 비어 있지 않으면 `[최댓값, 최솟값]`

을 반환한다.

## Quick Summary

- 최댓값 삭제와 최솟값 삭제가 모두 필요하므로 priority queue 하나로는 부족하다.
- max heap과 min heap을 둘 다 사용한다.
- 같은 원소를 두 heap에 넣고, `id`로 같은 원소임을 연결한다.
- 한쪽 heap에서 삭제된 원소는 `deleted_vec[id] = true`로 표시한다.
- 반대쪽 heap에서는 삭제된 원소가 top에 올라왔을 때 lazy하게 제거한다.

## Thinking Process

### First Idea

처음에는 최댓값용 priority queue와 최솟값용 priority queue를 따로 두고, 삭제할 때마다 반대쪽 queue를 복사해서 동기화하면 된다고 생각했다.

예시 흐름:

```text
최댓값 큐      최솟값 큐
[16 -5643]   [-5643 16]

[16 -5643]   [16]
[16]         [16]   -> 작은 큐로 복사

[]           [16]
[]           []     -> 작은 큐로 복사

[]           []

[123]        [123]

[123]        []
[]           []
```

### Failed Point

이 방식의 문제는 queue 복사 비용이다.

priority queue를 복사하면 내부 원소 개수만큼 복사하므로 `O(n)`이 걸린다.

삭제 연산이 나올 때마다 반대쪽 queue를 복사해서 동기화하면 최악의 경우 다음처럼 된다.

```text
O(n) 복사 * O(n) 연산 = O(n^2)
```

`operations` 길이는 최대 1,000,000이므로 이 방식은 시간초과가 날 가능성이 크다.

따라서 heap을 통째로 복사하면서 동기화하는 방식은 부적합하다.

### Why The Name Is Confusing

문제 이름이 이중 우선순위 큐라서 C++의 `priority_queue`만 사용해야 할 것처럼 보인다.

하지만 여기서 말하는 이중 우선순위 큐는 자료구조 개념이다.

```text
최댓값도 우선적으로 꺼낼 수 있고
최솟값도 우선적으로 꺼낼 수 있는 큐
```

일반 `priority_queue`는 한 방향만 쉽다.

```text
max heap -> 최댓값 조회/삭제가 쉬움
min heap -> 최솟값 조회/삭제가 쉬움
```

이 문제는 두 방향이 모두 필요하므로, 단일 priority queue만으로는 부족하다.

## My Approach

priority queue를 직접 조합해서 구현한다.

```text
max heap + min heap + lazy deletion
```

### Current Realization

구조체를 사용하면 priority queue만으로도 이중 우선순위 큐를 직접 구현할 수 있다.

다만 구조체 하나만으로 되는 것은 아니다.

값이 중복될 수 있으므로 각 원소에 고유 id를 붙여야 한다.

```cpp
struct Node {
    int value;
    int id;
};
```

그리고 같은 원소를 max heap과 min heap에 둘 다 넣는다.

```text
maxHeap: 최댓값 삭제용
minHeap: 최솟값 삭제용
```

삭제 여부는 `deleted_vec[id]` 배열로 공유한다.

```text
deleted_vec[id] = false -> 아직 삭제되지 않음
deleted_vec[id] = true  -> 이미 삭제됨
```

### Lazy Deletion

한쪽 heap에서 원소를 삭제해도, 반대쪽 heap 안에는 같은 원소가 아직 남아 있다.

예를 들어 max heap에서 `(16, id0)`을 삭제해도 min heap에는 `(16, id0)`이 남아 있을 수 있다.

이때 반대쪽 heap을 즉시 뒤져서 삭제하면 비효율적이다.

대신 삭제 여부만 기록한다.

```text
deleted_vec[id] = true
```

그리고 나중에 해당 heap의 top으로 올라왔을 때 제거한다.

이 방식이 lazy deletion이다.

정리 로직은 다음과 같다.

```text
heap의 top이 이미 삭제된 id라면 pop한다.
삭제되지 않은 id가 top에 올 때까지 반복한다.
```

## Mistakes And Lessons

이번 문제에서 현재까지의 핵심 실수와 깨달음은 다음이다.

- heap을 복사해서 양쪽을 동기화하려고 한 것은 시간복잡도 때문에 부적합하다.
- 이중 우선순위 큐는 C++ `priority_queue` 하나를 쓰라는 뜻이 아니다.
- 최댓값과 최솟값을 모두 다뤄야 하므로 양방향 처리가 필요하다.
- priority queue로 직접 구현하려면 max heap, min heap, id, deleted_vec 배열이 필요하다.
- 중복 값이 있으므로 value만으로 삭제 여부를 관리하면 안 된다.

## Key Point

직접 구현한다면 다음 구조로 간다.

```text
struct Node {
    int value;
    int id;
}

maxHeap: value 큰 순
minHeap: value 작은 순
deleted_vec[id]: 삭제 여부
```

연산 흐름:

```text
I x:
  id 발급
  deleted_vec[id] = false
  maxHeap.push({x, id})
  minHeap.push({x, id})

D 1:
  maxHeap top에 이미 죽은 원소가 있으면 정리
  삭제되지 않은 top이 있으면 deleted_vec[id] = true 후 pop

D -1:
  minHeap top에 이미 죽은 원소가 있으면 정리
  삭제되지 않은 top이 있으면 deleted_vec[id] = true 후 pop

마지막:
  maxHeap, minHeap 모두 죽은 top 정리
  비었으면 [0, 0]
  아니면 [maxHeap.top().value, minHeap.top().value]
```

시간복잡도는 전체적으로 `O(n log n)`으로 볼 수 있다.

### Accepted Direction

최종적으로는 `priority_queue` 두 개와 `deleted_vec`을 사용해서 직접 구현했다.

핵심 구조:

```text
max_q: 최댓값 조회/삭제용
min_q: 최솟값 조회/삭제용
deleted_vec[id]: 해당 id의 원소가 이미 삭제되었는지 기록
```

각 삽입 연산마다 `Node`를 만들고, 같은 `Node`를 두 heap에 모두 넣는다.

```cpp
struct Node {
    int val;
    int id;
};
```

`id`는 operations의 index를 사용했다. 삽입 연산이 아닌 삭제 연산에도 id가 생기지만, 실제 heap에는 삽입 명령에서만 들어가므로 삭제 여부 배열의 크기를 `operations.size()`로 잡아도 충분하다.

## Accepted Solution

```cpp
#include <string>
#include <vector>
#include <queue>
#include <iostream>

using namespace std;

struct Node {
    int val;
    int id;
};

struct CompareMax {
    bool operator()(const Node& a, const Node& b) {
        return a.val < b.val;
        // return true 될 경우 a의 우선순위가 낮아진다, 뒤로 간다.
    }
};

struct CompareMin {
    bool operator()(const Node& a, const Node& b) {
        return a.val > b.val;
    }
};

template <typename Compare>
void printpq(priority_queue<Node, vector<Node>, Compare> pq) {
    cout << '[';
    while (false == pq.empty()) {
        Node top = pq.top();
        cout << '(' << top.val << " " << top.id << ')';
        pq.pop();
    }
    cout << ']' << endl;
}

vector<int> solution(vector<string> operations) {
    vector<int> answer;
    priority_queue<Node, vector<Node>, CompareMax> max_q;
    priority_queue<Node, vector<Node>, CompareMin> min_q;

    int size = operations.size();
    vector<bool> deleted_vec(size);
    for (int i = 0; i < size; i++) {
        auto str = operations.at(i);
        char cmd = str[0];
        int val = stoi(str.substr(2));

        Node n;
        n.val = val;
        n.id = i;
        if ('I' == cmd) {
            max_q.push(n);
            min_q.push(n);
        } else {
            // max_q 삭제
            if (1 == val) {
                while (false == max_q.empty()) {
                    Node max = max_q.top();
                    max_q.pop();
                    bool deleted = deleted_vec[max.id];
                    if (false == deleted) {
                        deleted_vec[max.id] = true;
                        break;
                    }
                }
            } else { // min_q 삭제
                while (false == min_q.empty()) {
                    Node min = min_q.top();
                    min_q.pop();
                    bool deleted = deleted_vec[min.id];
                    if (false == deleted) {
                        deleted_vec[min.id] = true;
                        break;
                    }
                }
            }
        }
    }

    int max = 0;
    while (false == max_q.empty()) {
        Node max_node = max_q.top();
        max_q.pop();
        bool deleted = deleted_vec[max_node.id];
        if (false == deleted) {
            max = max_node.val;
            break;
        }
    }

    int min = 0;
    while (false == min_q.empty()) {
        Node min_node = min_q.top();
        min_q.pop();
        bool deleted = deleted_vec[min_node.id];
        if (false == deleted) {
            min = min_node.val;
            break;
        }
    }

    answer.push_back(max);
    answer.push_back(min);

    return answer;
}
```

## Alternative Solution

이 문제는 `multiset`을 이용한 풀이도 가능하다.

```cpp
#include <string>
#include <vector>
#include <set>

using namespace std;

vector<int> solution(vector<string> operations) {
    multiset<int> ms;

    for (auto operation : operations) {
        char cmd = operation[0];
        int val = stoi(operation.substr(2));

        if ('I' == cmd) {
            ms.insert(val);
        } else {
            if (true == ms.empty()) {
                continue;
            }

            if (1 == val) {
                ms.erase(prev(ms.end()));
            } else {
                ms.erase(ms.begin());
            }
        }
    }

    if (true == ms.empty()) {
        return {0, 0};
    }

    return {*prev(ms.end()), *ms.begin()};
}
```

`multiset`은 자동 정렬되고 중복을 허용하므로 최솟값은 `begin()`, 최댓값은 `prev(end())`로 접근할 수 있다.

## Complexity

각 삽입은 max heap과 min heap에 한 번씩 들어가므로 `O(log n)`이다.

삭제도 heap top을 정리하면서 진행되며, 각 원소는 전체 과정에서 많아야 한 번씩 pop된다.

따라서 전체 시간복잡도는 `O(n log n)`으로 볼 수 있다.

두 heap과 `deleted_vec`을 사용하므로 공간복잡도는 `O(n)`이다.

## Notes

이번 문제에서 실제로 구현한 핵심은 다음이다.

- `Node` 구조체로 값과 id를 묶었다.
- max heap과 min heap을 직접 만들었다.
- 같은 원소를 두 heap에 모두 넣었다.
- 한쪽 heap에서 삭제된 원소를 `deleted_vec[id]`로 표시했다.
- 반대쪽 heap에서는 이미 삭제된 원소가 top에 올라왔을 때 버렸다.
- 이것이 lazy deletion 방식이다.

조언을 받았지만, 실제로 구조체, comparator, lazy deletion, 결과 정리까지 직접 코드로 옮겼다.

이 문제에서 기억해야 할 패턴:

```text
하나의 priority_queue로 양방향 삭제는 어렵다.
두 heap을 쓰되, 같은 원소를 id로 연결한다.
한쪽에서 삭제한 흔적은 deleted_vec 배열로 공유한다.
반대쪽 heap에서는 top에 올라왔을 때 lazy하게 버린다.
```
