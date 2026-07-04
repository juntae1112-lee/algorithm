# Heap 02 - Disk Controller

## Problem

- Platform: Programmers
- Category: Heap
- Title: Disk Controller
- Korean title: 디스크 컨트롤러

## Problem Summary

각 작업은 다음 두 값을 가진다.

```text
[요청 시각, 작업 소요 시간]
```

하드디스크는 한 번에 하나의 작업만 처리할 수 있다.

목표는 각 작업의 반환 시간 평균을 최소화하는 것이다.

```text
반환 시간 = 작업 종료 시각 - 작업 요청 시각
```

## Core Idea

현재 시각까지 요청된 작업 중에서, 작업 소요 시간이 가장 짧은 작업을 먼저 처리한다.

즉, 두 가지 자료구조가 필요하다.

- 아직 도착하지 않은 작업 목록
- 이미 도착해서 기다리는 작업 우선순위 큐

우선순위 큐는 작업 소요 시간이 짧은 작업이 먼저 나오도록 min heap으로 만든다.

## Quick Summary

이 문제에서 가장 중요한 점은 요청 시간에 맞게 작업을 대기 큐에 넣는 것이다.

아직 요청되지 않은 작업은 실행 후보가 아니다. 따라서 현재 시간 `sec` 기준으로 요청 시간이 `sec` 이하인 작업만 실행 대기 큐에 넣어야 한다.

```text
대기 큐에 넣을 수 있는 조건: ask_time <= sec
```

내 최종 방식은 힙을 두 개 사용한다.

```text
pq  = 아직 대기 큐에 들어가지 않은 작업들
      요청 시간 빠른 순으로 꺼낸다.

pq2 = 현재 시간까지 도착해서 대기 중인 작업들
      소요 시간 짧은 순 -> 요청 시간 빠른 순 -> 작업 번호 작은 순으로 꺼낸다.
```

처리 흐름:

```text
1. 모든 작업을 Task로 변환해서 원래 작업 번호를 저장한다.
2. 요청 시간용 pq에 모든 작업을 넣는다.
3. 현재 시간 sec까지 도착한 작업을 pq에서 pq2로 모두 옮긴다.
4. pq2에서 실행 우선순위가 가장 높은 작업을 꺼낸다.
5. 작업을 실행하고 완료 시점까지 sec를 이동한다.
6. 반환 시간인 완료 시각 - 요청 시각을 sum에 더한다.
7. 모든 작업이 끝나면 sum / 작업 개수를 반환한다.
```

실수하기 쉬운 지점:

- 전체 작업을 처음부터 실행 우선순위 큐에 넣으면 안 된다.
- 정렬하거나 heap에 넣기 전에 원래 작업 번호를 보존해야 한다.
- 현재 시간까지 도착한 작업은 `if`가 아니라 `while`로 전부 옮겨야 한다.
- 실행 중 작업을 별도 queue로 관리할 필요는 없다.
- 대기 큐가 비어 있고 미래 작업만 남아 있으면 다음 요청 시간으로 점프할 수 있다.

## Timeline Example

예를 들어 작업이 다음과 같다고 하자.

```text
[[0, 3], [1, 9], [3, 5]]
```

### 0ms

```text
작업 중: 없음
대기 큐: []
```

0ms에 요청된 `[0, 3]`이 대기 큐에 들어간다.

```text
대기 큐: [(요청 0ms, 소요 3ms)]
```

가장 짧은 작업을 꺼내 실행한다.

```text
작업 중: [0, 3]
현재 시각: 0ms -> 3ms
반환 시간: 3 - 0 = 3
```

### 1ms

1ms에 `[1, 9]`가 요청되지만, `[0, 3]`이 실행 중이므로 바로 처리하지 못하고 기다린다.

```text
대기 큐: [(요청 1ms, 소요 9ms)]
```

### 3ms

3ms에 첫 작업이 끝난다.

이때 `[3, 5]`도 요청되므로 대기 큐에 들어간다.

```text
대기 큐: [(요청 1ms, 소요 9ms), (요청 3ms, 소요 5ms)]
```

소요 시간이 더 짧은 `[3, 5]`를 먼저 실행한다.

```text
작업 중: [3, 5]
현재 시각: 3ms -> 8ms
반환 시간: 8 - 3 = 5
```

### 8ms

남은 작업 `[1, 9]`를 실행한다.

```text
작업 중: [1, 9]
현재 시각: 8ms -> 17ms
반환 시간: 17 - 1 = 16
```

평균 반환 시간은 다음과 같다.

```text
(3 + 5 + 16) / 3 = 8
```

## Why Priority Queue?

매 시점마다 필요한 것은 전체 정렬 결과가 아니다.

필요한 것은 현재까지 요청된 작업 중 가장 짧은 작업 하나다.

그래서 작업이 도착할 때마다 우선순위 큐에 넣고, 다음 작업을 선택할 때 가장 짧은 작업을 꺼내면 된다.

## Failed Attempt - Put All Jobs Into Priority Queue

처음에는 모든 작업을 시작부터 우선순위 큐에 넣고, 소요 시간이 짧은 순서로 꺼내면 될 것 같다고 생각했다.

```cpp
#include <string>
#include <vector>
#include <queue>
#include <iostream>

using namespace std;

struct Task {
    int num;
    int ask_time;
    int process_time;
};

struct Compare {
    // a b 비교시 return true이면 a가 b보다 우선순위가 낮다,
    // return false면 a가 b보다 우선순위가 높다.
    bool operator()(const Task& a, const Task& b) {
        if (a.process_time != b.process_time) {
            return a.process_time > b.process_time;
        }
        if (a.ask_time != b.ask_time) {
            return a.ask_time > b.ask_time;
        }

        return a.num > b.num;
    }
};

int solution(vector<vector<int>> jobs) {
    int answer = 0;

    priority_queue<Task, vector<Task>, Compare> pq;
    for (int i = 0; i < jobs.size(); i++) {
        Task task;
        task.num = i;
        task.ask_time = jobs.at(i).at(0);
        task.process_time = jobs.at(i).at(1);
        pq.push(task);
    }

    int sec = 0;
    vector<int> temp;
    while (!pq.empty()) {
        auto top = pq.top();

        if (sec <= top.ask_time) {
            sec = top.ask_time;
        }

        int complete_time = sec + top.process_time;
        int time = complete_time - top.ask_time;

        sec = complete_time;
        temp.push_back(time);

        pq.pop();
    }

    return answer;
}
```

이 방식은 불가능하다.

이유는 처음부터 모든 작업을 우선순위 큐에 넣으면, 아직 요청 시간이 되지 않은 작업도 실행 후보가 되기 때문이다.

예를 들어 현재 시간이 `0ms`인데 `3ms`에 들어올 작업이 우선순위가 높다는 이유로 먼저 꺼내질 수 있다. 하지만 문제 조건상 아직 요청되지 않은 작업은 대기 큐에 들어온 상태가 아니므로 실행 후보가 될 수 없다.

따라서 우선순위 큐에 넣을 수 있는 작업은 전체 작업이 아니라, 현재 시간 기준으로 이미 요청된 작업뿐이다.

정리하면:

- 전체 작업을 처음부터 priority queue에 넣으면 안 된다.
- 현재 시간에 들어올 수 있는 작업만 대기 큐에 넣어야 한다.
- 아직 요청 시각이 안 된 작업은 후보가 아니다.
- 그래서 요청 시간 기준으로 정렬하거나, 요청 시간 기준 우선순위 큐를 따로 두는 방식이 필요하다.

## Failed Attempt - Sort Jobs Directly And Simulate Time

다음 시도는 `jobs`를 요청 시간 기준으로 정렬하고, 현재 시간마다 들어올 수 있는 작업을 priority queue에 넣는 방식이었다.

```cpp
#include <string>
#include <vector>
#include <queue>
#include <iostream>
#include <algorithm>

using namespace std;

struct Task {
    int num;
    int ask_time;
    int process_time;
    int end_time;
};

struct Compare {
    // a b 비교시 return true이면 a가 b보다 우선순위가 낮다,
    // return false면 a가 b보다 우선순위가 높다.
    bool operator()(const Task& a, const Task& b) {
        if (a.process_time != b.process_time) {
            return a.process_time > b.process_time;
        }
        if (a.ask_time != b.ask_time) {
            return a.ask_time > b.ask_time;
        }

        return a.num > b.num;
    }
};

int solution(vector<vector<int>> jobs) {
    int answer = 0;

    sort(jobs.begin(), jobs.end(), [](const vector<int>& a, const vector<int>& b) {
        // sort는 return true면 a가 앞에 온다.
        // 요청 시간이 작으면 앞에 와야 한다.
        return a.at(0) < b.at(0);
    });

    priority_queue<Task, vector<Task>, Compare> pq;
    queue<Task> q;

    vector<int> temp;
    int sec = 0;
    int i = 0;
    while (temp.size() < jobs.size()) {
        if (false == q.empty()) {
            auto qFront = q.front();
            if (sec == qFront.end_time) {
                temp.push_back(qFront.end_time - qFront.ask_time);
                q.pop();
            }
        }

        if (i < jobs.size()) {
            auto job = jobs.at(i);
            int ask_time = job.at(0);
            int process_time = job.at(1);
            if (ask_time <= sec) {
                Task task;
                task.num = i;
                task.ask_time = ask_time;
                task.process_time = process_time;
                pq.push(task);
                i++;
            }
        }

        if ((true == q.empty()) && (false == pq.empty())) {
            auto pqTop = pq.top();
            pqTop.end_time = sec + pqTop.process_time;
            q.push(pqTop);
            pq.pop();
        }
        sec++;
    }

    int sum = 0;
    for (auto a: temp) {
        sum += a;
    }
    answer = sum / temp.size();
    return answer;
}
```

이 방식도 실패 가능성이 있다.

가장 큰 문제는 `jobs`를 직접 정렬한 뒤 `i`를 작업 번호로 사용한다는 점이다.

```cpp
task.num = i;
```

문제에서 작업 번호는 처음 입력된 `jobs` 배열의 index이다.

하지만 `jobs`를 정렬하면 배열 순서가 바뀐다. 따라서 정렬 후의 `i`는 원래 작업 번호가 아니다. 소요 시간과 요청 시간이 같은 작업들 사이에서 작업 번호가 tie-break 기준이 되는 케이스가 있으면 오답이 날 수 있다.

따라서 정렬 전에 반드시 `Task`로 변환해서 원래 작업 번호를 보존해야 한다.

```text
Task.num = 원래 jobs index
```

그 다음 `vector<Task>`를 요청 시간 기준으로 정렬해야 한다.

두 번째 문제는 한 시점에 도착 가능한 작업을 한 개만 넣는 구조이다.

```cpp
if (i < jobs.size()) {
    ...
    if (ask_time <= sec) {
        pq.push(task);
        i++;
    }
}
```

같은 시간에 여러 작업이 요청될 수 있으므로, 현재 시간까지 도착한 모든 작업을 `while`로 넣어야 한다.

```text
현재 시간까지 들어온 작업을 전부 pq에 넣는다.
```

세 번째 문제는 실행 중인 작업을 `queue<Task> q`로 따로 관리하고, 시간을 `sec++`로 1씩 증가시키는 구조이다.

하드디스크는 한 번 작업을 시작하면 끝까지 그 작업만 수행한다. 따라서 작업을 하나 꺼낸 순간 바로 다음처럼 처리할 수 있다.

```text
현재 시간 += 작업 소요 시간
반환 시간 += 현재 시간 - 요청 시간
```

실행 중 작업을 별도 queue에 넣고 매 ms마다 완료 여부를 확인할 필요가 없다.

정리하면 이 시도의 실패 원인은 다음과 같다.

- 정렬 후 index를 작업 번호로 사용해서 원래 작업 번호가 소실된다.
- 현재 시간까지 도착한 작업을 한 번에 모두 넣지 못한다.
- 시간을 1씩 증가시키며 실행 중 작업 queue를 관리해서 로직이 불필요하게 복잡하다.
- 이 문제는 작업 단위로 시간을 점프시키는 방식이 더 적합하다.

## Accepted But Still Improvable - Three Queue Simulation

다음 시도는 답은 맞았다.

구조는 다음처럼 나눴다.

- `pq`: 요청 시간이 작은 순서로 작업을 꺼내는 queue
- `pq2`: 현재 대기 중인 작업 중 실행 우선순위가 높은 작업을 꺼내는 queue
- `q`: 현재 실행 중인 작업을 저장하는 queue

```cpp
#include <string>
#include <vector>
#include <queue>
#include <iostream>
#include <algorithm>

using namespace std;

struct Task {
    int num;
    int ask_time;
    int proc_time;
    int end_time;
};

struct Compare {
    bool operator()(const Task& a, const Task& b) {
        if (a.ask_time != b.ask_time) {
            return a.ask_time > b.ask_time;
        }

        return a.num > b.num;
    }
};

struct Compare2 {
    // a b 비교시 return true이면 a가 b보다 우선순위가 낮다,
    // return false면 a가 b보다 우선순위가 높다.
    bool operator()(const Task& a, const Task& b) {
        if (a.proc_time != b.proc_time) {
            return a.proc_time > b.proc_time;
        }
        if (a.ask_time != b.ask_time) {
            return a.ask_time > b.ask_time;
        }

        return a.num > b.num;
    }
};

int solution(vector<vector<int>> jobs) {
    int answer = 0;

    // 요청 시간이 작은 순 우선순위 큐
    priority_queue<Task, vector<Task>, Compare> pq;
    for (int i = 0; i < jobs.size(); i++) {
        Task task;
        task.num = i;
        task.ask_time = jobs[i][0];
        task.proc_time = jobs[i][1];
        pq.push(task);
    }

    // 작업 대기 큐
    priority_queue<Task, vector<Task>, Compare2> pq2;
    queue<Task> q;
    int sec = 0;
    int sum = 0;

    while ((false == pq.empty()) || (false == pq2.empty()) || (false == q.empty())) {
        if (false == q.empty()) {
            auto processing = q.front();
            if (sec == processing.end_time) {
                sum += processing.end_time - processing.ask_time;
                q.pop();
            }
        }

        // 현재 시간까지 요청된 작업을 대기 큐에 삽입
        while ((false == pq.empty()) && (pq.top().ask_time <= sec)) {
            auto t = pq.top();
            pq2.push(t);
            pq.pop();
        }

        // 대기 큐가 있고 작업 큐가 비었으면 작업 시작
        if ((false == pq2.empty()) && (true == q.empty())) {
            auto waiting = pq2.top();
            waiting.end_time = sec + waiting.proc_time;
            q.push(waiting);
            pq2.pop();
        }

        sec++;
    }

    answer = sum / jobs.size();
    return answer;
}
```

이 방식은 통과는 했지만 개선점이 보인다.

핵심은 굳이 작업 큐 `q`가 필요 없다는 것이다.

하드디스크는 한 번 작업을 시작하면 그 작업이 끝날 때까지 다른 작업을 하지 않는다. 따라서 `pq2`에서 작업을 하나 꺼낸 순간, 실행 중 상태를 queue에 저장해두고 매 ms마다 확인할 필요가 없다.

바로 다음처럼 계산할 수 있다.

```text
sec += proc_time
sum += sec - ask_time
```

즉 작업을 시작하는 순간 완료 시점까지 시간을 점프하면 된다.

현재 방식은 다음 이유로 더 복잡하다.

- 요청 queue, 대기 queue, 실행 queue까지 상태가 세 개다.
- `sec++`로 1ms씩 증가시키므로 이벤트 단위가 아니라 시간 단위 시뮬레이션이 된다.
- 작업 완료 여부를 매 반복마다 확인해야 한다.

개선 방향은 다음과 같다.

- 요청 시간 기준 자료구조 하나
- 실행 우선순위 기준 priority queue 하나
- 작업을 꺼내면 현재 시간을 완료 시점으로 바로 이동
- 대기 큐가 비어 있으면 다음 요청 시간으로 현재 시간을 이동

## Mistakes And Lessons

이번 문제에서 나온 실수들은 다음에 같은 유형을 풀 때 바로 점검해야 한다.

첫 번째 실수는 문제의 우선순위 조건을 처음에 놓친 것이다.

문제의 우선순위는 다음 순서다.

```text
소요 시간 짧은 것 -> 요청 시각 빠른 것 -> 작업 번호 작은 것
```

요청 시각이 빠른 작업이 항상 먼저 실행되는 것이 아니다. 현재 대기 큐에 이미 들어온 작업들 중에서는 소요 시간이 짧은 작업이 먼저다.

두 번째 실수는 모든 작업을 처음부터 priority queue에 넣으려고 한 것이다.

아직 요청 시간이 되지 않은 작업은 대기 큐에 들어온 작업이 아니다. 따라서 실행 후보가 될 수 없다.

```text
전체 작업 중 우선순위가 높은 작업
```

이 아니라:

```text
현재 시간까지 요청된 작업 중 우선순위가 높은 작업
```

을 골라야 한다.

세 번째 실수는 `jobs`를 직접 정렬하면서 작업 번호를 잃은 것이다.

문제에서 작업 번호는 처음 입력된 `jobs` 배열의 index이다.

```text
jobs[i] = i번 작업
```

따라서 정렬 후의 index를 작업 번호로 쓰면 안 된다. 정렬 전에 `Task`로 변환해서 원래 index를 보존해야 한다.

네 번째 실수는 현재 시간까지 들어온 작업을 한 번에 모두 대기 큐에 넣지 않은 것이다.

같은 시간에 여러 작업이 요청될 수 있고, 작업이 끝난 동안 여러 작업이 이미 도착해 있을 수 있다.

따라서 `if`가 아니라 `while`로 처리해야 한다.

```text
현재 시간까지 들어온 작업을 전부 pq에 넣는다.
```

다섯 번째 실수는 `ask_time`과 `sec` 비교 방향을 반대로 생각했던 것이다.

현재 시간에 대기 큐에 들어올 수 있는 작업은 요청 시간이 현재 시간보다 작거나 같은 작업이다.

```text
ask_time <= sec
```

이어야 한다.

반대로 생각하면 아직 요청되지 않은 미래 작업이 대기 큐에 들어가게 된다.

여섯 번째 실수는 실행 중인 작업을 별도 queue로 관리하면서 시간을 1씩 증가시킨 것이다.

이 문제는 하드디스크가 작업을 시작하면 끝까지 수행한다. 그러므로 매 ms마다 시뮬레이션할 필요가 없다.

작업을 선택한 순간:

```text
현재 시간 += 작업 소요 시간
반환 시간 += 현재 시간 - 요청 시간
```

으로 처리하는 것이 더 단순하다.

다만 처음에 시간을 건너뛰어야 한다고 생각한 방향 자체는 맞았다.

대기 큐에서 작업을 하나 꺼내 실행했다면, 중간 시간을 하나씩 볼 필요 없이 바로 작업 완료 시점으로 이동하면 된다.

또 대기 큐가 비어 있고 아직 요청되지 않은 작업만 남아 있다면, 현재 시간을 다음 작업의 요청 시각으로 이동할 수 있다.

정리하면 이 문제의 핵심 체크리스트는 다음과 같다.

- 아직 요청되지 않은 작업은 실행 후보가 아니다.
- 현재 시간까지 도착한 작업 조건은 `ask_time <= sec`이다.
- 작업 번호는 원래 입력 index를 유지해야 한다.
- 현재 시간까지 도착한 작업은 모두 대기 큐에 넣어야 한다.
- 실행 우선순위는 소요 시간, 요청 시각, 작업 번호 순이다.
- 시간은 1씩 증가시키기보다 이벤트 시점으로 점프하는 것이 낫다.

## My Code Evolution

이번 문제는 다음 순서로 풀이가 발전했다.

### 1. 모든 작업을 처음부터 실행 우선순위 큐에 넣음

처음에는 모든 작업을 `priority_queue`에 넣고, 문제의 우선순위 조건대로 꺼내면 된다고 생각했다.

이때 comparator는 다음 기준으로 만들었다.

```text
소요 시간 짧은 순
요청 시간 빠른 순
작업 번호 작은 순
```

이 comparator 자체는 실행 대기 큐 기준으로는 맞다.

하지만 이 방식은 아직 요청 시간이 되지 않은 작업도 큐에 들어가므로 문제 조건과 맞지 않는다.

실패 원인:

- 전체 작업을 처음부터 실행 후보로 넣었다.
- 아직 요청되지 않은 작업도 우선순위가 높으면 먼저 꺼낼 수 있다.
- 문제에서 실행 후보는 전체 작업이 아니라 현재 시간까지 도착한 작업이다.

### 2. 요청 시간 기준으로 sort한 뒤 시간 단위로 시뮬레이션

다음에는 `jobs`를 요청 시간 기준으로 정렬하고, 현재 시간 `sec`에 들어올 수 있는 작업을 실행 대기 큐에 넣는 방식으로 바꿨다.

이때 중요한 실수가 있었다.

정렬 후의 index를 작업 번호로 사용했다.

```text
task.num = i
```

하지만 문제의 작업 번호는 처음 입력된 `jobs` 배열의 index이다.

따라서 정렬 전에 `Task`로 변환해서 원래 작업 번호를 보존해야 한다.

실패 원인:

- 정렬 후 index를 작업 번호로 사용했다.
- 현재 시간까지 들어온 작업을 한 번에 모두 넣지 못했다.
- 작업 완료를 확인하기 위해 실행 중 작업 queue를 따로 뒀다.
- `sec++`로 1ms씩 증가시키면서 로직이 복잡해졌다.

### 3. 요청 시간용 힙과 실행 대기용 힙을 분리

그 다음에는 힙을 두 개로 나눴다.

```text
pq  = 요청 시간이 빠른 작업을 먼저 꺼내는 힙
pq2 = 현재 대기 중인 작업 중 실행 우선순위가 높은 작업을 꺼내는 힙
```

이 방식에서는 처음부터 `Task`로 변환하므로 작업 번호를 잃지 않는다.

또 요청 시간용 힙 `pq`에서 현재 시간까지 들어온 작업만 `pq2`로 옮긴다.

```text
pq.top().ask_time <= sec
```

이 조건을 만족하는 작업을 `while`로 전부 옮겨야 한다.

이 방식은 논리적으로 맞고, 힙을 연습하는 목적에도 맞다.

### 4. 실행 중 작업 queue 제거

통과한 3큐 방식에서 다시 보니, 실행 중 작업을 저장하는 `queue<Task> q`는 필요 없었다.

하드디스크는 한 번 작업을 시작하면 그 작업이 끝날 때까지 다른 작업을 하지 않는다.

따라서 실행 대기 큐 `pq2`에서 작업을 하나 꺼낸 순간 바로 완료 시점까지 계산할 수 있다.

```text
end_time = sec + proc_time
sum += end_time - ask_time
sec = end_time
```

이렇게 하면 실행 중 작업 queue 없이도 반환 시간을 바로 누적할 수 있다.

최종 구조:

```text
pq  = 아직 대기 큐로 옮기지 않은 작업, 요청 시간 빠른 순
pq2 = 현재 대기 중인 작업, 실행 우선순위 순
sec = 현재 시간
sum = 반환 시간 누적합
```

### Final Direction

최종적으로 두 가지 풀이가 가능하다.

하나는 정석적인 방식이다.

```text
Task 배열로 변환
요청 시간 기준 sort
index로 현재 시간까지 도착한 작업을 pq에 공급
실행 우선순위 pq 하나로 처리
```

다른 하나는 힙 학습에 맞는 방식이다.

```text
요청 시간용 priority_queue
실행 우선순위용 priority_queue
```

요청 시간용 힙은 `sort + index`로 대체할 수 있지만, 힙 연습 목적이라면 힙 두 개를 쓰는 방식도 타당하다.

중요한 것은 입력 index와 요청 시간 사이에는 아무 보장이 없다는 점이다.

따라서 다음 둘 중 하나로 요청 시간 순서를 보장해야 한다.

- 요청 시간 기준으로 정렬한다.
- 요청 시간 기준 priority queue를 사용한다.

## My Final Approach

이 기록에서는 내가 직접 짠 힙 두 개 방식으로 정리한다.

첫 번째 힙은 아직 실행 대기 큐에 들어가지 않은 작업을 요청 시간 순서로 관리한다.

```text
pq = 요청 시간이 빠른 순
```

두 번째 힙은 현재 시간까지 요청되어 대기 중인 작업을 실행 우선순위 순서로 관리한다.

```text
pq2 = 소요 시간 짧은 순 -> 요청 시간 빠른 순 -> 작업 번호 작은 순
```

작업 번호는 처음 입력된 `jobs`의 index이므로, 처음부터 `Task`로 변환해서 보존한다.

## My Accepted Solution

```cpp
#include <string>
#include <vector>
#include <queue>
#include <iostream>
#include <algorithm>

using namespace std;

struct Task {
    int num;
    int ask_time;
    int proc_time;
};

struct Compare {
    bool operator()(const Task& a, const Task& b) {
        if (a.ask_time != b.ask_time) {
            return a.ask_time > b.ask_time;
        }

        return a.num > b.num;
    }
};

struct Compare2 {
    // a b 비교시 return true이면 a가 b보다 우선순위가 낮다,
    // return false면 a가 b보다 우선순위가 높다.
    bool operator()(const Task& a, const Task& b) {
        if (a.proc_time != b.proc_time) {
            return a.proc_time > b.proc_time;
        }
        if (a.ask_time != b.ask_time) {
            return a.ask_time > b.ask_time;
        }

        return a.num > b.num;
    }
};

int solution(vector<vector<int>> jobs) {
    int answer = 0;

    // 요청 시간이 작은 순 우선순위 큐
    priority_queue<Task, vector<Task>, Compare> pq;
    for (int i = 0; i < jobs.size(); i++) {
        Task task;
        task.num = i;
        task.ask_time = jobs[i][0];
        task.proc_time = jobs[i][1];
        pq.push(task);
    }

    // 작업 대기 큐
    priority_queue<Task, vector<Task>, Compare2> pq2;
    int sec = 0;
    int sum = 0;

    while ((false == pq.empty()) || (false == pq2.empty())) {
        // 현재 시간까지 요청된 작업을 전부 대기 큐에 삽입
        while ((false == pq.empty()) && (pq.top().ask_time <= sec)) {
            auto t = pq.top();
            pq2.push(t);
            pq.pop();
        }

        if (false == pq2.empty()) {
            auto waiting = pq2.top();
            int end_time = sec + waiting.proc_time;
            sum += end_time - waiting.ask_time;
            pq2.pop();

            sec = end_time;
        } else {
            sec++;
        }
    }

    answer = sum / jobs.size();
    return answer;
}
```

## Further Improvement

내 최종 코드에서 `pq2`가 비어 있고 `pq`에 미래 작업만 남아 있으면 `sec++`로 1씩 증가시킨다.

이 부분은 다음 요청 시간으로 바로 점프할 수 있다.

```text
sec = pq.top().ask_time
```

이렇게 하면 대기 시간이 긴 입력에서도 불필요한 반복을 줄일 수 있다.

또 요청 시간용 `pq`는 `Task` 배열을 요청 시간 기준으로 정렬한 뒤 index로 공급하는 방식으로 대체할 수 있다.

하지만 이 문제를 힙 학습으로 보면, 요청 시간용 힙과 실행 대기용 힙을 나누는 현재 방식도 타당하다.

## Key Point

이 문제에서 가장 중요했던 점은 요청 시간에 맞게 작업을 대기 큐에 넣는 것이다.

즉, 실행 우선순위 큐에는 아무 작업이나 넣는 것이 아니라 현재 시간 기준으로 실제 요청이 들어온 작업만 넣어야 한다.

이 조건이 깨지면 아직 요청되지 않은 미래 작업이 실행 후보가 되어버린다.

핵심 기준은 다음이다.

- 아직 요청되지 않은 작업은 실행 후보가 아니다.
- 현재 시간까지 요청된 작업만 실행 대기 큐에 들어간다.
- 현재 시간까지 들어온 작업 조건은 `ask_time <= sec`이다.
- 실행 대기 큐 안에서는 소요 시간, 요청 시간, 작업 번호 순으로 우선순위가 정해진다.
- 작업을 시작하면 완료 시점까지 시간을 점프할 수 있다.

## Complexity

작업 개수를 `n`이라고 하면:

- 요청 시간용 priority queue push/pop: `O(log n)`
- 실행 대기용 priority queue push/pop: `O(log n)`

각 작업은 요청 시간용 힙에서 한 번 나오고, 실행 대기용 힙에 한 번 들어갔다가 한 번 나온다.

따라서 전체 시간복잡도는 `O(n log n)`이다.

두 개의 priority queue에 작업을 저장하므로 공간복잡도는 `O(n)`이다.
