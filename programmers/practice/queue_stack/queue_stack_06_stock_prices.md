# Queue/Stack 06 - Stock Prices

## Problem

- Platform: Programmers
- Category: Queue / Stack
- Title: Stock Prices
- Korean title: 주식가격

## Problem Summary

초 단위로 기록된 주식가격 배열 `prices`가 주어진다.

각 시점의 가격이 이후 몇 초 동안 떨어지지 않았는지 구해야 한다.

가격이 한 번이라도 더 낮은 가격을 만나면, 그 시점까지의 시간이 정답이 된다. 끝까지 더 낮은 가격을 만나지 않으면 마지막 시점까지 버틴 시간이 정답이 된다.

## Constraints

- `prices`의 각 가격은 1 이상 10,000 이하이다.
- `prices`의 길이는 2 이상 100,000 이하이다.

## My Solving Approach

이 문제는 stack을 사용해서 풀 수 있다.

stack에는 아직 가격이 떨어지는 순간을 만나지 못한 시점들을 저장한다.

각 원소는 다음처럼 저장한다.

```text
(시점, 가격)
```

새 가격이 들어왔을 때, stack top에 있는 이전 가격보다 작으면 그 이전 가격은 지금 가격에서 떨어진 것이다.

그래서 현재 시점에서 이전 시점을 빼서 정답을 기록하고 pop한다.

## Stack Thinking

예시:

```text
prices = [1, 2, 3, 2, 3]
```

각 시점별 생각은 다음과 같다.

```text
1초 시점 가격은 항상 뒤의 가격보다 작거나 같다.
2초 시점 가격도 항상 뒤의 가격보다 작거나 같다.
3초 시점 가격은 4초 시점 가격보다 크다.
4초 시점 가격은 항상 뒤의 가격보다 작거나 같다.
5초 시점 가격은 마지막이므로 0초이다.
```

stack 흐름:

```text
1초 [(1 1)] [, , , , ]
2초 [(1 1) (2 2)] [, , , , ]
3초 [(1 1) (2 2) (3 3)] [, , , , ]
4초 [(1 1) (2 2) (4 2)] [, , 1, , ]
    -> 새로 들어오는 4초 가격 2가 기존 3초 가격 3보다 작아서 pop
5초 [(1 1) (2 2) (4 2) (5 3)] [, , 1, , ]
남아 있으면 현재시간에서 들어간 시간을 뺀다.
결과: [4, 3, 1, 1, 0]
```

## Last Price Observation

마지막 가격은 자기 자신의 유지 시간에는 영향을 주지 않는다.

마지막 시점은 뒤에 비교할 가격이 없으므로 항상 `0`초이다.

처음에는 마지막 값이 바뀌면 앞의 값에도 영향을 줄 수 있는지 확인해봤다.

예를 들어 마지막 값이 `3`이 아니라 `1`이어도:

```text
5초 [(1 1) (5 1)] [, 3, 1, 1, ]
결과: [4, 3, 1, 1, 0]
```

이 케이스에서는 결과가 같다.

마지막 값은 자기 자신의 시간에는 의미가 없고, 앞의 값들도 마지막 시점에서 떨어지든 끝까지 버티든 계산되는 시간은 결국 마지막 시점까지의 시간이다.

## Key Point

새로 들어오는 가격이 stack top의 가격보다 작으면, 이전 가격은 지금 시점에서 떨어진 것이다.

그리고 한 번만 pop하는 것이 아니라, 새 가격보다 큰 이전 가격들이 stack 위에 계속 남아 있다면 작거나 같은 가격을 만날 때까지 반복해서 pop해야 한다.

정리하면:

- 이전 가격이 현재 가격보다 크면 가격이 떨어진 것이다.
- 떨어진 가격은 현재 시점과 들어간 시점의 차이를 정답으로 기록한다.
- 현재 가격보다 작거나 같은 가격을 만날 때까지 stack에서 pop한다.
- 끝까지 남은 값들은 마지막 시점까지 가격이 떨어지지 않은 것이다.

## Accepted Solution

```cpp
#include <string>
#include <vector>
#include <stack>
#include <utility>
#include <iostream>

using namespace std;

vector<int> solution(vector<int> prices) {
    int size = prices.size();
    vector<int> answer(size);
    stack<pair<int,int>> s;

    for (int i = 0; i < prices.size(); i++) {
        int price = prices.at(i);
        int cur_sec = i + 1;

        while ((false == s.empty()) && ((price < s.top().second) || (i == size - 1))) {
            int top_sec = s.top().first;
            answer[top_sec - 1] = cur_sec - top_sec;
            s.pop();
        }

        s.push({cur_sec, price});
    }

    return answer;
}
```

## Notes

- stack에는 아직 가격이 떨어지지 않은 시점들을 저장한다.
- 현재 가격이 이전 가격보다 낮으면 이전 가격의 유지 시간이 확정된다.
- 마지막 시점에서는 남아 있는 값들을 현재 시점 기준으로 정리한다.
- 마지막 값의 정답은 항상 `0`이다.

## Complexity

처음 보면 `for` 안에 `while`이 있어서 `O(n^2)`처럼 보일 수 있다.

하지만 각 가격은 stack에 한 번 들어가고, 많아야 한 번 나온다.

즉 전체 과정에서:

- `push`는 최대 `n`번
- `pop`도 최대 `n`번

발생한다.

따라서 전체 시간복잡도는 `O(n)`이다.

공간복잡도는 정답 배열과 stack을 사용하므로 `O(n)`이다.
