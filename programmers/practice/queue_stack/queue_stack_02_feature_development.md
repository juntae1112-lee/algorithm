# Queue/Stack 01 - Feature Development

## Problem

- Platform: Programmers
- Category: Queue / Stack
- Title: Feature Development
- Korean title: 기능개발

## Problem Summary

기능들은 주어진 순서대로 배포되어야 한다.

각 기능은 진도가 100%가 되어야 배포할 수 있지만, 뒤에 있는 기능이 먼저 완성될 수도 있다. 이 경우 뒤 기능은 앞 기능이 배포될 때까지 기다렸다가 함께 배포된다.

따라서 핵심은 각 기능이 완성되는 날짜를 구한 뒤, 앞 기능의 배포 날짜를 기준으로 뒤 기능들을 함께 묶는 것이다.

## Quick Summary

- 각 기능이 완료되기까지 걸리는 날짜를 먼저 계산한다.
- 앞 기능의 완료 날짜가 현재 배포 묶음의 기준이 된다.
- 기준 날짜보다 작거나 같은 완료 날짜는 같은 배포에 포함한다.
- 기준 날짜보다 큰 완료 날짜가 나오면 기존 묶음을 결과에 넣고 새 묶음을 시작한다.

## Constraints

- 작업 개수는 100개 이하이다.
- 작업 진도는 100 미만의 자연수이다.
- 작업 속도는 100 이하의 자연수이다.
- 배포는 하루에 한 번, 하루의 끝에 이루어진다.
- 예를 들어 95%에서 하루 4%씩 진행되면, 1일 뒤에는 99%라 아직 배포할 수 없고 2일 뒤에 배포할 수 있다.

## My Approach

나는 코드를 바로 떠올리기보다, 먼저 말로 흐름을 정리하고 그걸 코드로 옮기는 방식이 편하다.

먼저 각 기능마다 남은 진도율을 계산한다.

- 현재 진도율: `95 90 99 99 80 99`
- 남은 진도율: `5 10 1 1 20 1`
- 각 기능이 끝나는 데 걸리는 날짜: `5 10 1 1 20 1`

이제 앞에서부터 순서대로 보면서 큐에 같은 배포 묶음이 될 기능들을 담는다고 생각한다.

## Thinking Process

첫 번째 기능의 완료 날짜가 현재 배포 기준이 된다.

- step 1: queue `{5}`
- step 2: 다음 값 `10`은 queue first인 `5`보다 크다.
- 따라서 기존 queue를 비우고, 지금까지 묶인 기능 개수를 결과에 넣는다.
- 이 시점의 배포 개수는 `현재 기능 1개 + queue에 있던 기능 수`로 생각한다.

다음 배포 기준은 `10`이 된다.

- step 3: queue `{1}`
- step 4: queue `{1, 1}`
- step 5: 다음 값 `20`은 queue first 기준보다 크다.
- 따라서 기존 queue를 비우고, 지금까지 묶인 기능 개수를 결과에 넣는다.
- 이 시점의 배포 개수는 `3`이다.

마지막 배포 기준은 `20`이 된다.

- step 6: queue `{1}`
- step 7: 더 이상 비교할 다음 값이 없으므로 마지막 묶음을 결과에 넣는다.
- 이 시점의 배포 개수는 `1`이다.

## Key Point

뒤 기능이 먼저 완성되어도 앞 기능보다 먼저 배포할 수 없다.

그래서 현재 배포 기준 날짜보다 작거나 같은 완료 날짜들은 전부 같은 배포 묶음으로 센다. 현재 기준보다 큰 완료 날짜가 나오면, 그때 새로운 배포 묶음이 시작된다.

## Accepted Solution

```cpp
#include <string>
#include <vector>
#include <queue>
#include <iostream>

using namespace std;

void printQ(queue<int> q) {
    cout << "[";
    while (!q.empty()) {
        cout << q.front() << " ";
        q.pop();
    }
    cout << "]" << endl;
}

vector<int> solution(vector<int> progresses, vector<int> speeds) {
    vector<int> answer;

    int size = progresses.size();
    if (speeds.size() != size) {
        return answer;
    }

    int completeProgress = 100;
    vector<int> times;

    // 각 기능이 완료되기까지 걸리는 날짜를 먼저 계산한다.
    for (int i = 0; i < size; i++) {
        int time = (completeProgress - progresses.at(i)) / speeds.at(i);
        int temp = (completeProgress - progresses.at(i)) % speeds.at(i);

        // 나머지가 있으면 하루가 더 필요하므로 올림 처리한다.
        if (0 < temp) {
            time++;
        }

        times.push_back(time);
    }

    queue<int> q;

    // q는 현재 같은 날 배포될 수 있는 기능 묶음이다.
    for (auto time : times) {
        if (q.empty()) {
            // 새 배포 묶음의 첫 기능을 넣는다.
            q.push(time);
        } else {
            // 현재 기능이 q의 첫 기능보다 늦게 끝나면,
            // 기존 묶음은 먼저 배포되고 새 묶음을 시작해야 한다.
            if (time > q.front()) {
                int task = q.size();
                answer.push_back(task);

                queue<int> q2;
                q2.push(time);
                swap(q, q2);
            } else {
                // q의 첫 기능보다 빨리 끝나거나 같은 날 끝나면 함께 배포된다.
                q.push(time);
            }
        }
    }

    // 마지막으로 남아 있는 배포 묶음을 결과에 추가한다.
    answer.push_back(q.size());

    return answer;
}
```

## Notes

- `[93, 30, 55]`, `[1, 30, 5]` -> `[2, 1]`
- `[95, 90, 99, 99, 80, 99]`, `[1, 1, 1, 1, 1, 1]` -> `[1, 3, 2]`
