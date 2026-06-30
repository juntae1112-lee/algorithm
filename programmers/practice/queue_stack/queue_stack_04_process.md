# Queue/Stack 04 - Process

## Problem

- Platform: Programmers
- Category: Queue / Stack
- Title: Process
- Korean title: 프로세스

## Problem Summary

운영체제가 실행 대기 큐에 있는 프로세스를 우선순위에 따라 실행한다.

대기 큐의 가장 앞 프로세스를 꺼냈을 때, 큐 안에 더 높은 우선순위의 프로세스가 남아 있으면 꺼낸 프로세스를 다시 큐 뒤에 넣는다.

더 높은 우선순위가 없다면 그 프로세스를 실행하고, 한 번 실행된 프로세스는 다시 큐에 들어가지 않는다.

목표는 `location` 위치에 있던 프로세스가 몇 번째로 실행되는지 구하는 것이다.

## Constraints

- `priorities`의 길이는 1 이상 100 이하이다.
- 우선순위는 1 이상 9 이하이다.
- 숫자가 클수록 우선순위가 높다.
- `location`은 찾고 싶은 프로세스의 처음 위치이다.

## My Solving Approach

이 문제는 `queue`와 `priority_queue`를 같이 쓰면 된다.

처음에는 우선순위 큐가 알아서 가장 높은 프로세스를 앞으로 보내주는 것처럼 생각할 수 있지만, 실제 대기 순서는 일반 queue가 관리해야 한다.

그래서 역할을 나눈다.

- `queue<pair<int, int>>`: 실제 대기 큐 순서를 관리한다.
- `priority_queue<int>`: 현재 남아 있는 프로세스 중 가장 높은 우선순위를 확인한다.

`pair`에는 원래 위치와 우선순위를 같이 저장한다.

- `first`: 원래 인덱스
- `second`: 우선순위

이렇게 해야 프로세스가 queue 안에서 계속 뒤로 이동해도, 처음 `location`에 있던 프로세스인지 확인할 수 있다.

## Queue Thinking

예시:

- 프로세스: `A B C D`
- 우선순위: `2 1 3 2`
- priority queue 상태: `3 2 2 1`

현재 최대 우선순위는 `3`이다.

대기 큐 앞에서부터 확인한다.

- `A(2)`는 현재 최대 우선순위 `3`보다 낮으므로 다시 뒤로 보낸다.
- `B(1)`도 현재 최대 우선순위 `3`보다 낮으므로 다시 뒤로 보낸다.
- `C(3)`은 현재 최대 우선순위와 같으므로 실행한다.

이제 실행 순서는 `C`가 첫 번째이다.

다음 최대 우선순위는 `2`이다.

- 현재 queue는 `D A B`이다.
- `D(2)`는 현재 최대 우선순위와 같으므로 실행한다.
- 다음 최대 우선순위도 `2`이다.
- `A(2)`가 실행된다.
- 마지막으로 `B(1)`이 실행된다.

결과적으로 실행 순서는 `C D A B`가 된다.

## Key Point

`priority_queue`는 실행 순서를 직접 바꾸는 자료구조가 아니다.

이 문제에서 `priority_queue`는 현재 실행 가능한 가장 높은 우선순위를 알려주는 역할만 한다. 실제 프로세스 순서는 일반 `queue`에서 앞 프로세스를 꺼내고, 실행하지 못하면 다시 뒤에 넣는 방식으로 관리한다.

또한 찾고 싶은 프로세스는 `location`으로 주어지므로, 우선순위만 저장하면 안 되고 원래 인덱스도 같이 저장해야 한다.

## Accepted Solution

```cpp
#include <string>
#include <vector>
#include <queue>
#include <utility>
#include <iostream>

using namespace std;

int solution(vector<int> priorities, int location) {
    int answer = 0;
    int size = priorities.size();

    queue<pair<int, int>> q;
    priority_queue<int> pq;

    // q에는 원래 인덱스와 우선순위를 같이 저장한다.
    // pq에는 현재 남은 프로세스 중 가장 높은 우선순위를 확인하기 위해 우선순위만 저장한다.
    for (int i = 0; i < size; i++) {
        int priority = priorities.at(i);
        q.push({i, priority});
        pq.push(priority);
    }

    int cnt = 0;
    while (!pq.empty()) {
        int top = pq.top();

        // 현재 최우선순위와 같은 프로세스가 나올 때까지 queue를 돌린다.
        while (!q.empty()) {
            int idx = q.front().first;
            int priority = q.front().second;

            if (priority == top) {
                // 현재 프로세스가 최우선순위이면 실행한다.
                q.pop();
                cnt++;

                // 실행한 프로세스의 원래 위치가 location이면 정답이다.
                if (location == idx) {
                    answer = cnt;
                }

                cout << idx << ", " << priority << ", " << cnt << endl;
                break;
            } else {
                // 더 높은 우선순위가 남아 있으므로 현재 프로세스는 queue 뒤로 보낸다.
                auto temp = q.front();
                q.pop();
                q.push(temp);
            }
        }

        // 방금 실행한 최우선순위 프로세스를 priority queue에서도 제거한다.
        pq.pop();
    }

    return answer;
}
```

## Notes

- 제출할 때는 디버깅용 `cout`은 제거하는 것이 좋다.
- `queue`는 실제 순서 관리용이다.
- `priority_queue`는 현재 최대 우선순위 확인용이다.
- `pair`를 사용해서 원래 인덱스와 우선순위를 함께 들고 다닌다.
