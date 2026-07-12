# Brute Force 02 - Mock Exam

## Problem

- Platform: Programmers
- Category: Brute Force
- Title: Mock Exam
- Korean title: 모의고사

## Problem Summary

정답 배열 `answers`가 주어진다.

수포자 1, 2, 3은 각각 정해진 반복 패턴으로 답을 찍는다.

가장 많은 문제를 맞힌 사람의 번호를 오름차순으로 반환해야 한다.

## Quick Summary

- 각 수포자의 찍기 패턴은 고정되어 있다.
- 문제를 순서대로 보면서 각 수포자의 현재 패턴 값과 정답을 비교한다.
- 패턴 끝에 도달하면 다시 처음으로 돌아간다.
- 가장 높은 점수를 구한 뒤, 그 점수와 같은 사람 번호를 답에 넣는다.
- 사람 번호를 1, 2, 3 순서대로 확인하면 자연스럽게 오름차순이 된다.

## Constraints

- 시험은 최대 10,000문제로 구성된다.
- 문제의 정답은 1, 2, 3, 4, 5 중 하나이다.
- 최고 점수자가 여러 명이면 오름차순으로 모두 반환한다.

## My Approach

각 수포자의 반복 패턴을 배열로 저장했다.

```cpp
vector<int> num_1{1,2,3,4,5};
vector<int> num_2{2,1,2,3,2,4,2,5};
vector<int> num_3{3,3,1,1,2,2,4,4,5,5};
```

그리고 각 수포자의 현재 패턴 위치를 `i`, `j`, `k`로 따로 관리했다.

```text
i -> 1번 수포자의 현재 패턴 위치
j -> 2번 수포자의 현재 패턴 위치
k -> 3번 수포자의 현재 패턴 위치
```

패턴 위치가 배열 길이를 넘으면 다시 0으로 돌렸다.

```cpp
if (i >= num_1.size()) {
    i = 0;
}
```

정답과 현재 패턴 값이 같으면 해당 수포자의 점수를 증가시켰다.

## Thinking Process

이 문제는 가능한 조합을 만들어 보는 완전탐색이라기보다는, 모든 문제를 한 번씩 확인하는 시뮬레이션에 가깝다.

각 문제마다 확인해야 하는 사람은 3명뿐이다.

```text
문제 1개당 비교 3번
전체 문제 수 n
시간복잡도 O(3n) -> O(n)
```

패턴이 반복되므로 핵심은 “현재 문제 번호에서 각 수포자가 어떤 답을 찍는가”를 구하는 것이다.

내 풀이는 인덱스 변수를 직접 증가시키고, 패턴 길이를 넘으면 0으로 되돌렸다.

예를 들어 1번 수포자는 다음처럼 반복된다.

```text
패턴: [1, 2, 3, 4, 5]
문제:  1  2  3  4  5  6  7
답:    1  2  3  4  5  1  2
```

## Mistakes And Lessons

큰 실수는 없었다.

다만 변수 이름 `answer`가 두 번 쓰인다.

```cpp
vector<int> answer;

for (auto answer: answers) {
    ...
}
```

`for`문 안의 `answer`는 정답 배열의 각 원소를 뜻하고, 바깥의 `answer`는 반환 배열이다.

동작에는 문제가 없지만, 복습할 때 헷갈릴 수 있으므로 실제로는 다음처럼 이름을 나누는 편이 좋다.

```cpp
for (auto correct : answers) {
    ...
}
```

또한 패턴 인덱스를 직접 초기화하는 방식도 맞지만, 반복 패턴 문제에서는 `%` 연산자를 쓰면 더 짧게 표현할 수 있다.

## Key Point

반복 패턴에서 현재 위치는 나머지 연산으로 구할 수 있다.

```text
pattern_index = problem_index % pattern_size
```

최고 점수자가 여러 명일 수 있으므로 최고 점수를 먼저 구한 뒤, 1번부터 3번까지 차례대로 확인한다.

```text
1번 점수 == 최고점 -> 1 추가
2번 점수 == 최고점 -> 2 추가
3번 점수 == 최고점 -> 3 추가
```

이 순서로 넣으면 결과는 자동으로 오름차순이 된다.

## Accepted Solution

내가 작성한 방식이다.

각 수포자의 패턴 인덱스를 직접 관리한다.

```cpp
#include <string>
#include <vector>
#include <iostream>
#include <algorithm>

using namespace std;

vector<int> solution(vector<int> answers) {
    vector<int> answer;
    vector<int> num_1{1,2,3,4,5};
    vector<int> num_2{2,1,2,3,2,4,2,5};
    vector<int> num_3{3,3,1,1,2,2,4,4,5,5};
    int i = 0, j = 0, k = 0;
    int x = 0, y = 0, z = 0;

    for (auto answer : answers) {
        if (i >= num_1.size()) {
            i = 0;
        }
        if (j >= num_2.size()) {
            j = 0;
        }
        if (k >= num_3.size()) {
            k = 0;
        }
        if (answer == num_1[i]) {
            x++;
        }
        if (answer == num_2[j]) {
            y++;
        }
        if (answer == num_3[k]) {
            z++;
        }
        i++;
        j++;
        k++;
    }

    int max_cnt = max(x, max(y, z));
    if (x == max_cnt) {
        answer.push_back(1);
    }
    if (y == max_cnt) {
        answer.push_back(2);
    }
    if (z == max_cnt) {
        answer.push_back(3);
    }

    return answer;
}
```

## Alternative Solution

패턴 인덱스를 직접 초기화하지 않고 `%` 연산자로 구할 수 있다.

```cpp
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

vector<int> solution(vector<int> answers) {
    vector<int> answer;

    vector<int> pattern1{1, 2, 3, 4, 5};
    vector<int> pattern2{2, 1, 2, 3, 2, 4, 2, 5};
    vector<int> pattern3{3, 3, 1, 1, 2, 2, 4, 4, 5, 5};
    vector<int> scores(3);

    for (int i = 0; i < answers.size(); i++) {
        if (answers[i] == pattern1[i % pattern1.size()]) {
            scores[0]++;
        }
        if (answers[i] == pattern2[i % pattern2.size()]) {
            scores[1]++;
        }
        if (answers[i] == pattern3[i % pattern3.size()]) {
            scores[2]++;
        }
    }

    int max_score = max(scores[0], max(scores[1], scores[2]));
    for (int i = 0; i < scores.size(); i++) {
        if (scores[i] == max_score) {
            answer.push_back(i + 1);
        }
    }

    return answer;
}
```

## Complexity

문제 수를 `n`이라고 하면 모든 문제를 한 번씩 확인한다.

각 문제마다 3명의 답만 비교하므로 시간복잡도는 `O(n)`이다.

패턴 배열과 점수 변수만 사용하므로 공간복잡도는 `O(1)`이다.

## Notes

- 이 문제는 레벨 1답게 구현 자체는 단순하다.
- 핵심은 반복 패턴을 어떻게 표현하느냐이다.
- 직접 인덱스를 증가시키는 방식도 맞다.
- 반복 패턴은 `%` 연산자를 쓰면 더 간단해진다.
- 반환 배열은 1, 2, 3 순서로 넣으면 오름차순 조건을 따로 처리하지 않아도 된다.
