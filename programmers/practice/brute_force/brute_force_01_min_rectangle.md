# Brute Force 01 - Minimum Rectangle

## Problem

- Platform: Programmers
- Category: Brute Force
- Title: Minimum Rectangle
- Korean title: 최소직사각형

## Problem Summary

여러 명함의 가로, 세로 길이가 주어진다.

명함은 회전해서 넣을 수 있다.

모든 명함을 수납할 수 있는 가장 작은 지갑의 넓이를 구해야 한다.

## Quick Summary

- 각 명함은 그대로 넣거나 회전해서 넣을 수 있다.
- 내 풀이는 현재 지갑 크기에서 새 명함을 넣을 수 있는지 확인한다.
- 못 넣는 경우 두 방향 중 면적이 더 작아지는 방향으로 지갑 크기를 갱신한다.
- 더 단순한 풀이는 각 명함마다 긴 변과 짧은 변으로 정규화하는 것이다.
- 긴 변들의 최댓값과 짧은 변들의 최댓값을 곱하면 답이 된다.

## Constraints

- `sizes`의 길이는 1 이상 10,000 이하이다.
- `sizes[i]`는 `[w, h]` 형식이다.
- `w`, `h`는 1 이상 1,000 이하이다.

## My Approach

처음에는 현재 지갑 크기 `(x, y)`를 유지하면서 명함을 하나씩 확인했다.

새 명함 `(a, b)`가 현재 지갑에 들어갈 수 있으면 넘어간다.

```text
x >= a && y >= b
```

또는 회전해서 들어갈 수 있어도 넘어간다.

```text
y >= a && x >= b
```

둘 다 불가능하면 새 명함을 넣기 위해 지갑 크기를 키워야 한다.

이때 두 가지 방향을 비교했다.

```text
case 1: (a, b) 방향으로 넣기
case 2: (b, a) 방향으로 넣기
```

각 방향으로 넣었을 때의 지갑 면적을 계산하고, 더 작은 면적이 되는 쪽으로 `x`, `y`를 갱신했다.

## Thinking Process

문제를 처음 보면 완전탐색처럼 보인다.

각 명함마다 선택지는 두 개다.

```text
그대로 넣기
회전해서 넣기
```

그래서 재귀로 모든 경우를 확인하는 것도 떠올릴 수 있다.

하지만 `sizes` 길이가 최대 10,000이므로 순수 재귀 완전탐색은 불가능하다.

```text
명함 n개
각 명함 선택지 2개
전체 경우의 수 = 2^n
```

내 풀이는 모든 경우를 재귀로 보지 않고, 현재 지갑에 명함을 넣을 수 있는지 확인한 뒤 필요한 경우에만 두 방향을 비교한다.

결과적으로 이 문제의 핵심은 다음이다.

```text
명함은 회전 가능하다.
그러면 각 명함마다 긴 변을 한쪽 축으로, 짧은 변을 다른 축으로 몰아도 된다.
```

예시:

```text
[60, 50] -> 긴 변 60, 짧은 변 50
[30, 70] -> 긴 변 70, 짧은 변 30
[60, 30] -> 긴 변 60, 짧은 변 30
[80, 40] -> 긴 변 80, 짧은 변 40
```

따라서 필요한 지갑 크기는:

```text
긴 변 최댓값 = 80
짧은 변 최댓값 = 50
정답 = 80 * 50 = 4000
```

## Mistakes And Lessons

이 문제는 완전탐색 카테고리에 있지만, 실제로 모든 경우를 다 탐색할 필요는 없다.

처음에는 현재 지갑 크기 기준으로 두 방향의 면적을 비교하는 방식으로 접근했다.

이 방식도 문제 조건에서는 맞는 방향으로 동작한다.

다만 문제의 구조를 더 잘 보면 각 명함을 독립적으로 정규화할 수 있다.

```text
큰 값은 큰 축으로
작은 값은 작은 축으로
```

즉 이 문제에서 중요한 깨달음은 “모든 회전 경우를 탐색하자”가 아니라 “회전 가능하므로 긴 변/짧은 변으로 정리하자”이다.

## Key Point

명함은 회전 가능하므로 각 명함을 다음처럼 정리할 수 있다.

```text
long_side = max(w, h)
short_side = min(w, h)
```

모든 명함을 담는 지갑은:

```text
max_long_side * max_short_side
```

이다.

## Accepted Solution

내가 작성한 방식이다.

현재 지갑 크기 `(x, y)`를 유지하면서, 새 명함을 넣을 수 없을 때 두 방향의 면적을 비교해서 더 작은 방향으로 갱신한다.

```cpp
#include <string>
#include <vector>

using namespace std;

int case_size(int x, int y, int a, int b) {
    if (x <= a) {
        x = a;
    }

    if (y <= b) {
        y = b;
    }

    return x * y;
}

int solution(vector<vector<int>> sizes) {
    int answer = 0;
    int x = 0;
    int y = 0;

    for (auto size : sizes) {
        int a = size[0];
        int b = size[1];

        if (((x >= a) && (y >= b)) || ((y >= a) && (x >= b))) {
            continue;
        } else {
            int case_1 = case_size(x, y, a, b);
            int case_2 = case_size(x, y, b, a);

            if (case_1 < case_2) {
                if (x <= a) {
                    x = a;
                }
                if (y <= b) {
                    y = b;
                }
            } else {
                if (x <= b) {
                    x = b;
                }
                if (y <= a) {
                    y = a;
                }
            }
        }
    }

    answer = x * y;
    return answer;
}
```

## Alternative Solution

문제 구조를 더 단순하게 보면, 각 명함을 긴 변과 짧은 변으로 정규화하면 된다.

```cpp
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

int solution(vector<vector<int>> sizes) {
    int max_width = 0;
    int max_height = 0;

    for (auto size : sizes) {
        int w = size[0];
        int h = size[1];

        max_width = max(max_width, max(w, h));
        max_height = max(max_height, min(w, h));
    }

    return max_width * max_height;
}
```

## Complexity

내 풀이와 개선 풀이 모두 명함을 한 번씩만 확인한다.

시간복잡도는 `O(n)`이다.

추가 자료구조를 사용하지 않으므로 공간복잡도는 `O(1)`이다.

재귀로 모든 회전 경우를 탐색하면 `O(2^n)`이 되므로 제한 조건에서 사용할 수 없다.

## Notes

- 완전탐색 카테고리여도 항상 모든 경우를 직접 탐색하는 것은 아니다.
- 회전 가능 조건을 보면 긴 변과 짧은 변으로 정규화할 수 있다.
- 내 풀이는 현재 지갑 기준으로 방향을 선택하는 방식이다.
- 더 단순한 풀이는 각 명함을 `max(w, h)`, `min(w, h)`로 바꿔서 최댓값만 구하는 방식이다.
