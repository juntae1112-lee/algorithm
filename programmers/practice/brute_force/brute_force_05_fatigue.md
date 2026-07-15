# Brute Force 05 - Fatigue

## Problem

- Platform: Programmers
- Category: Brute Force
- Title: Fatigue
- Korean title: 피로도

## Problem Summary

현재 피로도 `k`와 여러 던전 정보 `dungeons`가 주어진다.

각 던전은 `[최소 필요 피로도, 소모 피로도]` 형태이다.

현재 피로도가 최소 필요 피로도 이상이면 던전을 탐험할 수 있고, 탐험 후 소모 피로도만큼 피로도가 줄어든다.

탐험할 수 있는 던전 수의 최댓값을 구해야 한다.

## Quick Summary

- 던전은 방문 순서에 따라 결과가 달라진다.
- 던전 개수는 최대 8개라 모든 순서를 재귀로 탐색할 수 있다.
- `selected`로 이미 선택한 던전을 표시한다.
- 현재 피로도로 던전을 갈 수 있으면 피로도를 줄이고 `cnt`를 증가시킨다.
- 모든 경우 중 가장 큰 `cnt`를 답으로 반환한다.
- 보완 풀이에서는 현재 갈 수 없는 던전은 재귀에 넣지 않고, for문으로 다른 던전 후보는 계속 확인한다.

## Constraints

- `k`는 1 이상 5,000 이하이다.
- 던전 개수는 1 이상 8 이하이다.
- 각 던전은 `[최소 필요 피로도, 소모 피로도]`이다.
- 최소 필요 피로도는 항상 소모 피로도보다 크거나 같다.
- 서로 다른 던전의 정보가 같을 수 있다.

## My Approach

재귀로 던전 방문 순서를 모두 만들어 보았다.

선택 여부는 `vector<bool> selected`로 관리했다.

```cpp
vector<bool> selected(dungeons.size(), false);
```

각 재귀 호출에서 아직 선택하지 않은 던전을 하나 고른다.

```cpp
if (true == selected[i]) {
    continue;
}
```

던전을 선택한 뒤, 현재 피로도 `k`가 최소 필요 피로도 이상이면 탐험 가능한 것으로 보고:

```text
피로도 감소
탐험 횟수 증가
```

를 처리했다.

```cpp
if (new_k >= dungeon[0]) {
    new_k -= dungeon[1];
    new_cnt++;
}
```

이후 재귀로 다음 던전 선택을 이어간다.

모든 선택을 확인한 뒤에는 `selected[i] = false`, `temp.pop_back()`으로 상태를 되돌렸다.

## Thinking Process

이 문제는 던전의 순서가 중요하다.

예시:

```text
k = 80
dungeons = [[80,20], [50,40], [30,10]]
```

순서가 다음과 같으면:

```text
0번 -> 1번 -> 2번
```

진행은 다음과 같다.

```text
0번 탐험: k = 80 -> 60
1번 탐험: k = 60 -> 20
2번 탐험: 최소 필요 30, 현재 20이므로 불가능
```

탐험 수는 2개이다.

하지만 순서를 바꾸면:

```text
0번 -> 2번 -> 1번
```

진행은 다음과 같다.

```text
0번 탐험: k = 80 -> 60
2번 탐험: k = 60 -> 50
1번 탐험: k = 50 -> 10
```

탐험 수는 3개이다.

따라서 특정 기준으로 정렬해서 풀기 어렵고, 가능한 순서를 모두 확인해야 한다.

던전 개수는 최대 8개이므로 순열 완전탐색이 가능하다.

```text
최대 경우의 수 = 8! = 40320
```

## Mistakes And Lessons

큰 실수 없이 재귀 완전탐색으로 풀었다.

이제 다음 패턴은 익숙해지고 있다.

```text
selected 배열
하나 선택
재귀 호출
선택 해제
```

다만 내 풀이에는 의미상 보완할 점이 있다.

현재 코드는 선택한 던전을 탐험할 수 없더라도 재귀를 계속 진행한다.

```cpp
if (new_k >= dungeon[0]) {
    new_k -= dungeon[1];
    new_cnt++;
}

new_cnt = explore_dungeons(dungeons, selected, new_k, new_cnt, temp);
```

즉 “탐험하지 못한 던전도 순서에 넣고 넘어가는” 형태이다.

이 방식도 모든 순서를 확인하기 때문에 답은 구할 수 있다.

하지만 문제 의미로 보면 탐험할 수 없는 던전은 선택하지 않고, 갈 수 있는 던전만 재귀로 들어가는 편이 더 명확하다.

이렇게 해도 되는 이유는 피로도가 증가하지 않고 항상 감소하기 때문이다.

```text
현재 피로도로 못 가는 던전
-> 다른 던전을 먼저 가면 피로도는 더 줄어듦
-> 나중에도 그 던전은 갈 수 없음
```

즉 현재 못 가는 던전을 재귀 후보에서 제외해도 답을 놓치지 않는다.

다만 그 던전 하나만 제외하는 것이고, `for`문은 계속 돌면서 현재 갈 수 있는 다른 던전은 모두 확인한다.

그 경우에는 더 이상 갈 수 있는 던전이 없을 때 현재 `cnt`가 답 후보가 되므로, `max_cnt`를 `cnt`로 시작해야 한다.

```cpp
int max_cnt = cnt;
```

또 하나의 보완점은 인자 복사이다.

내 코드는 다음 값들이 재귀마다 복사된다.

```cpp
vector<vector<int>> dungeons
vector<bool> selected
vector<int> temp
```

제한이 작아서 통과는 가능하지만, `dungeons`는 바뀌지 않으므로 `const vector<vector<int>>&`로 넘기는 편이 좋다.

`temp`는 디버깅용 경로 기록이라 정답 풀이에는 없어도 된다.

## Key Point

이 문제는 순서가 결과에 영향을 준다.

따라서 가능한 방문 순서를 모두 확인해야 한다.

핵심 백트래킹 구조는 다음이다.

```text
for each dungeon
    이미 선택했으면 skip
    현재 피로도로 갈 수 없으면 skip
    선택
    피로도 감소, 개수 증가
    재귀 호출
    선택 해제
```

던전 개수가 최대 8개이므로 순열 완전탐색이 가능하다.

## Accepted Solution

내가 작성한 방식이다.

던전 순서를 모두 만들어 보면서 탐험 가능한 경우에는 `cnt`를 증가시킨다.

```cpp
#include <string>
#include <vector>
#include <iostream>

using namespace std;

int explore_dungeons(vector<vector<int>> dungeons, vector<bool> selected, int k, int cnt, vector<int> temp) {
    if (temp.size() == dungeons.size()) {
        return cnt;
    }

    int max_cnt = 0;
    for (int i = 0; i < dungeons.size(); i++) {
        if (true == selected[i]) {
            continue;
        }

        temp.push_back(i);
        selected[i] = true;

        vector<int> dungeon = dungeons[i];
        int new_k = k;
        int new_cnt = cnt;
        if (new_k >= dungeon[0]) {
            new_k -= dungeon[1];
            new_cnt++;
        }

        new_cnt = explore_dungeons(dungeons, selected, new_k, new_cnt, temp);
        if (new_cnt > max_cnt) {
            max_cnt = new_cnt;
        }

        selected[i] = false;
        temp.pop_back();
    }

    return max_cnt;
}

int solution(int k, vector<vector<int>> dungeons) {
    vector<bool> selected(dungeons.size(), false);
    int cnt = 0;
    int answer = -1;
    vector<int> temp;

    answer = explore_dungeons(dungeons, selected, k, cnt, temp);

    return answer;
}
```

## Alternative Solution

보완한 방식이다.

탐험할 수 없는 던전은 재귀로 들어가지 않는다.

또한 `dungeons`는 변경하지 않으므로 참조로 넘기고, 디버깅용 `temp`는 제거한다.

```cpp
#include <string>
#include <vector>

using namespace std;

int explore(const vector<vector<int>>& dungeons, vector<bool>& selected, int k, int cnt) {
    int max_cnt = cnt;

    for (int i = 0; i < dungeons.size(); i++) {
        if (selected[i]) {
            continue;
        }

        int required = dungeons[i][0];
        int cost = dungeons[i][1];

        if (k < required) {
            continue;
        }

        selected[i] = true;
        int result = explore(dungeons, selected, k - cost, cnt + 1);
        if (result > max_cnt) {
            max_cnt = result;
        }
        selected[i] = false;
    }

    return max_cnt;
}

int solution(int k, vector<vector<int>> dungeons) {
    vector<bool> selected(dungeons.size(), false);
    return explore(dungeons, selected, k, 0);
}
```

이 풀이가 더 명확한 이유는 다음과 같다.

```text
탐험할 수 있는 던전만 선택한다.
탐험한 경우에만 cnt가 증가한다.
더 갈 수 없으면 현재 cnt가 답 후보이다.
현재 못 가는 던전은 나중에도 못 가므로 제외해도 안전하다.
```

## Complexity

던전 개수를 `n`이라고 하면 가능한 순서를 탐색한다.

최악의 경우 시간복잡도는 `O(n!)`에 가깝다.

하지만 `n <= 8`이므로 충분히 가능하다.

방문 여부 배열 `selected`를 사용하므로 공간복잡도는 `O(n)`이다.

재귀 호출 스택도 최대 깊이 `n`까지 쌓인다.

## Notes

- 던전 순서가 결과에 영향을 주므로 완전탐색이 필요하다.
- `selected` 배열을 사용하는 재귀 백트래킹 패턴이다.
- 내 풀이도 통과하는 맞는 풀이이다.
- 보완 풀이에서는 갈 수 없는 던전은 아예 재귀에 넣지 않는다.
- 이때 다른 던전 후보는 같은 for문에서 계속 확인하므로 유효한 선택지를 놓치지 않는다.
- 피로도는 증가하지 않기 때문에 현재 못 가는 던전은 나중에도 갈 수 없다.
- `dungeons`처럼 바뀌지 않는 값은 참조로 넘기는 편이 좋다.
