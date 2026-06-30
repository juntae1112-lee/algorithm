# Queue/Stack 05 - Truck on Bridge

## Problem

- Platform: Programmers
- Category: Queue / Stack
- Title: Truck on Bridge
- Korean title: 다리를 지나는 트럭

## Problem Summary

여러 트럭이 정해진 순서대로 일차선 다리를 건넌다.

다리에는 최대 `bridge_length`대의 트럭이 올라갈 수 있고, 다리 위 트럭들의 총 무게는 `weight`를 넘을 수 없다.

모든 트럭이 다리를 완전히 건너는 데 걸리는 최소 시간을 구해야 한다.

## First Confusing Point

처음에는 `bridge_length`가 단순히 다리에 동시에 올라갈 수 있는 트럭 수처럼 보였다.

하지만 예제에서 `bridge_length = 100`, 트럭이 `[10]` 하나일 때 정답이 `101`인 것을 보면, `bridge_length`는 트럭이 다리를 통과하는 데 걸리는 시간처럼 동작한다.

즉 트럭이 1초에 다리에 올라가고, `bridge_length`초 동안 다리 위에 있다가, 그 다음 시점에 완전히 빠져나간다고 보면 된다.

예를 들어 `bridge_length = 2`라면:

- 1초: 트럭이 다리에 올라감
- 2초: 트럭이 다리 위에 있음
- 3초: 트럭이 다리를 완전히 빠져나감

그래서 트럭 한 대가 완전히 지나가는 시점은 `bridge_length + 1`처럼 보인다.

## My First Simulation Idea

처음에는 다리를 위치별 queue로 표현하려고 했다.

예시:

```text
0초 [] [] [] [7 4 5 6]
1초 [] [] [7] [4 5 6]
2초 [] [7] [] [4 5 6]
3초 [7] [] [4] [5 6]
4초 [7] [4] [5] [6]
5초 [7 4] [5] [] [6]
6초 [7 4 5] [] [6] []
7초 [7 4 5] [6] [] []
8초 [7 4 5 6] [] []
```

이 방식에서는 다리의 각 위치를 따로 두고, 매초 트럭을 한 칸씩 이동시킨다.

## Problem With This Idea

위치별 queue 방식은 이해하기는 쉽지만 비효율적이다.

매초마다 `bridge_length`만큼 순회하면서 트럭을 한 칸씩 이동시켜야 하기 때문이다.

제한 조건을 보면:

- `bridge_length`는 최대 10,000
- 트럭 개수도 최대 10,000

따라서 매초 다리의 모든 칸을 훑는 방식은 타임아웃이 날 수 있다.

## First Attempt - Timeout

첫 번째 시도는 다리 길이만큼 queue를 만들어서, 매초 트럭을 한 칸씩 옮기는 방식이었다.

```text
첫번째: 큐를 다리길이만큼 생성
결과: 테스트 5 시간초과
```

시뮬레이션 흐름은 다음처럼 이해했다.

```text
0초 [] [] [] [7 4 5 6]
1초 [] [] [7] [4 5 6]
2초 [] [7] [] [4 5 6]
3초 [7] [] [4] [5 6]
4초 [7] [4] [5] [6]
5초 [7 4] [5] [] [6]
6초 [7 4 5] [] [6] []
7초 [7 4 5] [6] [] []
8초 [7 4 5 6] [] []
```

이 방식에서 중요한 구현상 문제도 있었다.

앞에서 뒤로 순회하면 방금 옮긴 트럭이 같은 초에 또 이동해버린다. 그래서 위치를 직접 옮기는 방식에서는 뒤에서 앞으로 순회해야 한 초에 한 칸만 이동한다.

하지만 뒤에서 앞으로 고쳐도, 결국 매초 `bridge_length`만큼 확인해야 하는 구조라 시간초과 위험은 남는다.

## Current Realization

이 문제의 핵심은 트럭의 현재 위치를 계속 기록하는 것이 아니다.

더 효율적인 방향은 트럭이 다리에 들어갈 때, 그 트럭이 언제 빠져나갈지를 미리 기록하는 것이다.

즉 다리 위 queue에는 트럭을 이렇게 저장한다.

```text
(트럭 무게, 빠져나갈 시간)
```

예를 들어 `bridge_length = 2`이고, 7kg 트럭이 1초에 들어갔다면:

```text
(7, 3)
```

처럼 저장할 수 있다.

그러면 매초 다리 전체를 순회할 필요가 없다. queue의 front만 보면 가장 먼저 빠져나갈 트럭이 누구인지 알 수 있다.

## Improved Thinking

각 시점마다 해야 할 일은 단순해진다.

- 현재 시간에 빠져나갈 트럭이 있는지 확인한다.
- 빠져나갈 시간이 된 트럭이 있으면 다리 queue에서 제거하고, 현재 다리 무게에서 그 트럭 무게를 뺀다.
- 다음 대기 트럭이 다리에 올라갈 수 있는지 확인한다.
- 올라갈 수 있으면 `(트럭 무게, 현재 시간 + bridge_length)` 형태로 다리 queue에 넣는다.
- 모든 대기 트럭이 사라지고, 다리 위 무게도 0이 되면 종료한다.

두 번째 방향의 시뮬레이션은 다음처럼 볼 수 있다.

```text
두번째: 다리에 넣는 순간 빠져나오는 시간을 같이 넣어서 pop

0초 [] [7 4 5 6]
1초 [(7, 3)] [4 5 6]
2초 [(7, 3)] [4 5 6]
3초 [(4, 5)] [5 6]
4초 [(4, 5), (5, 6)] [6]
5초 [(5, 6)] [6]
6초 [(6, 8)] []
7초 [(6, 8)] []
8초 [] []
```

여기서 `(7, 3)`은 7kg 트럭이 3초에 다리에서 빠져나간다는 뜻이다.

이렇게 하면 매초 다리의 모든 위치를 움직일 필요가 없고, queue의 front에 있는 트럭이 빠져나갈 시간이 되었는지만 보면 된다.

## Key Point

처음에는 다리를 실제 칸처럼 보고, 트럭의 위치를 매초 이동시키려고 했다.

하지만 효율적으로 풀려면 다리를 칸 단위로 시뮬레이션하는 것이 아니라, 다리 위에 있는 트럭들의 “퇴장 시간”만 관리하면 된다.

정리하면:

- 이전 생각: 트럭의 현재 위치를 기록한다.
- 깨달은 방향: 트럭이 빠져나갈 시간을 기록한다.
- 이유: queue의 front만 확인하면 되므로 매초 전체 다리를 순회하지 않아도 된다.

## Implementation Notes

`queue.front()`를 사용할 때는 반드시 queue가 비어 있지 않은지 먼저 확인해야 한다.

처음 구현에서는 `passing.front()`와 `waiting.front()`를 바로 호출해서 문제가 생겼다. 특히 마지막 트럭이 이미 다리에 올라간 뒤에는 `waiting`이 비어 있어도 `passing`이 남아 있기 때문에 while문은 계속 돈다. 이때 `waiting.front()`를 호출하면 비정상 동작이 발생할 수 있다.

그래서 다음 두 조건을 각각 분리해서 확인해야 한다.

- `passing`이 비어 있지 않을 때만 빠져나갈 트럭을 확인한다.
- `waiting`이 비어 있지 않을 때만 다음 트럭을 다리에 올릴 수 있는지 확인한다.

## Accepted Solution

```cpp
#include <string>
#include <vector>
#include <queue>
#include <utility>
#include <iostream>

using namespace std;

void printq(queue<int> q) {
    cout << "[";
    while (false == q.empty()) {
        cout << q.front() << " ";
        q.pop();
    }
    cout << "]" << endl;
}

void printq(queue<pair<int,int>> q) {
    cout << "[";
    while (false == q.empty()) {
        auto front = q.front();
        cout << "(" << front.first << ", " << front.second << ") ";
        q.pop();
    }
    cout << "]" << endl;
}

int solution(int bridge_length, int weight, vector<int> truck_weights) {
    int answer = 0;
    queue<int> waiting;
    for (auto i: truck_weights) {
        waiting.push(i);
    }

    // first : weight, second : pass time
    queue<pair<int,int>> passing;
    int secs = 0;
    int passing_weights = 0;

    while ((false == waiting.empty()) || (false == passing.empty())) {
        secs++;

        if (false == passing.empty()) {
            auto passing_front_truck = passing.front();

            // 나올 시간이 되었으면 다리에서 제거한다.
            if (passing_front_truck.second == secs) {
                passing_weights -= passing_front_truck.first;
                passing.pop();
            }
        }

        if (false == waiting.empty()) {
            // waiting.front()가 passing에 들어갈 수 있는 조건:
            // 현재 다리 위 트럭 무게와 다음 트럭 무게의 합이 weight 이하여야 한다.
            int waiting_front_truck = waiting.front();
            if ((passing_weights + waiting_front_truck) <= weight) {
                // passing에 들어갈 때는 나올 시간을 같이 들고 들어간다.
                passing.push({waiting_front_truck, (secs + bridge_length)});
                passing_weights += waiting_front_truck;
                waiting.pop();
            }
        }
    }

    answer = secs;

    return answer;
}
```

## Final Notes

- `passing`은 현재 다리 위에 있는 트럭 queue이다.
- `waiting`은 아직 다리에 올라가지 않은 트럭 queue이다.
- `passing_weights`는 현재 다리 위 트럭들의 총 무게이다.
- 트럭을 다리에 넣을 때 `(트럭 무게, 빠져나갈 시간)`을 함께 저장한다.
- 디버깅용 `printq`, `cout`은 제출할 때 제거해도 된다.
