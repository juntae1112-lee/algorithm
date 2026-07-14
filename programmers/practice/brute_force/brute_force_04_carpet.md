# Brute Force 04 - Carpet

## Problem

- Platform: Programmers
- Category: Brute Force
- Title: Carpet
- Korean title: 카펫

## Problem Summary

갈색 격자 수 `brown`과 노란색 격자 수 `yellow`가 주어진다.

카펫은 중앙이 노란색이고, 바깥 테두리 한 줄이 갈색이다.

전체 카펫의 가로, 세로 길이를 `[가로, 세로]` 순서로 반환해야 한다.

## Quick Summary

- 갈색은 바깥 테두리 한 줄이다.
- 노란색은 테두리를 제외한 내부 영역이다.
- 내 풀이는 `brown`을 가로 테두리 몫과 세로 테두리 몫으로 나눠 보면서 가로/세로를 복원했다.
- 가로 후보와 세로 후보를 만들고, 전체 면적이 `brown + yellow`와 같은지 확인했다.
- 더 의미가 직접 드러나는 검증식은 내부 영역인 `(row - 2) * (col - 2) == yellow`이다.

## Constraints

- `brown`은 8 이상 5,000 이하이다.
- `yellow`는 1 이상 2,000,000 이하이다.
- 카펫의 가로 길이는 세로 길이와 같거나 세로보다 길다.

## My Approach

처음에는 갈색 테두리 개수를 직접 나눠서 생각했다.

카펫의 갈색 영역은 다음처럼 나눌 수 있다.

```text
위쪽 가로 테두리
아래쪽 가로 테두리
왼쪽 세로 테두리
오른쪽 세로 테두리
```

예를 들어 `brown = 24`, 정답이 `[8, 6]`인 경우를 보면:

```text
--------
-      -
-      -
-      -
-      -
--------
```

가로 테두리는 위, 아래 각각 8칸이다.

```text
위쪽 8칸 + 아래쪽 8칸 = 16칸
```

세로 테두리는 좌, 우 각각 내부 높이만큼 들어간다.

이미 모서리는 위/아래 가로 테두리에 포함되어 있으므로, 좌우 세로 테두리는 각각 `height - 2`칸이다.

```text
왼쪽 4칸 + 오른쪽 4칸 = 8칸
```

그래서:

```text
brown = 16 + 8 = 24
```

이 구조를 코드로 옮겼다.

## Thinking Process

`i`를 세로 테두리의 내부 높이라고 생각했다.

```cpp
int row_a = brown - i * 2;
int col_a = i * 2;
```

여기서:

```text
col_a = i * 2
```

는 좌우 세로 테두리의 갈색 개수이다.

그리고:

```text
row_a = brown - i * 2
```

는 남은 갈색, 즉 위쪽과 아래쪽 가로 테두리의 합이다.

가로 길이는 위/아래 테두리 중 한 줄의 길이이므로:

```cpp
int row = row_a / 2;
```

세로 길이는 내부 높이 `i`에 위/아래 테두리 2줄을 더한 값이므로:

```cpp
int col = col_a / 2 + 2;
```

예를 들어 `brown = 24`이고 `i = 4`라면:

```text
row_a = 24 - 4 * 2 = 16
col_a = 4 * 2 = 8

row = 16 / 2 = 8
col = 8 / 2 + 2 = 6
```

따라서 후보는 `[8, 6]`이다.

이때 전체 칸 수가 `brown + yellow`와 같으면 정답이다.

```cpp
if (row * col == (brown + yellow)) {
    answer.push_back(row);
    answer.push_back(col);
}
```

## Mistakes And Lessons

큰 실수는 없었다.

내 풀이는 `brown`의 테두리 구조를 직접 해석해서 가로/세로를 복원한 방식이다.

다만 조건을 확인할 때:

```cpp
row * col == brown + yellow
```

로 확인하면 전체 면적 기준이라 통과는 가능하지만, 문제 의미를 더 직접 드러내지는 않는다.

카펫 문제의 핵심은 노란색이 내부 영역이라는 점이므로 다음 조건이 더 의미가 명확하다.

```cpp
(row - 2) * (col - 2) == yellow
```

즉 내 풀이는 맞지만, 검증식을 내부 노란 영역 기준으로 쓰면 문제 구조가 더 잘 드러난다.

## Key Point

카펫에서 테두리 한 줄을 제외하면 노란색 영역이다.

따라서 가로가 `width`, 세로가 `height`라면:

```text
전체 칸 수 = width * height = brown + yellow
노란색 칸 수 = (width - 2) * (height - 2)
갈색 칸 수 = 전체 칸 수 - 노란색 칸 수
```

약수쌍 풀이에서는 이 식을 그대로 사용한다.

## Accepted Solution

내가 작성한 방식이다.

`brown`을 가로 테두리와 세로 테두리 몫으로 나눠 보면서 가로/세로 후보를 만든다.

```cpp
#include <string>
#include <vector>
#include <iostream>

using namespace std;

vector<int> solution(int brown, int yellow) {
    vector<int> answer;
    int i = 1;

    while (true) {
        int row_a = brown - i * 2;
        int col_a = i * 2;

        int row = row_a / 2;
        int col = col_a / 2 + 2;

        if (row < col) {
            break;
        }

        if (row * col == (brown + yellow)) {
            answer.push_back(row);
            answer.push_back(col);
        }

        i++;
    }

    return answer;
}
```

## Alternative Solution

더 일반적인 풀이는 전체 면적의 약수쌍을 확인하는 방식이다.

전체 카펫 칸 수는 다음과 같다.

```text
total = brown + yellow
```

카펫의 가로와 세로를 곱하면 전체 칸 수가 되어야 한다.

```text
width * height = total
```

따라서 `total`의 약수쌍을 확인하면 가능한 가로/세로 후보를 만들 수 있다.

예를 들어 `brown = 24`, `yellow = 24`라면:

```text
total = 48

48 = 48 * 1
48 = 24 * 2
48 = 16 * 3
48 = 12 * 4
48 = 8 * 6
```

이 중 `[8, 6]`을 확인하면:

```text
(8 - 2) * (6 - 2) = 6 * 4 = 24
```

노란색 개수와 같으므로 정답이다.

코드는 다음처럼 쓸 수 있다.

```cpp
#include <string>
#include <vector>

using namespace std;

vector<int> solution(int brown, int yellow) {
    int total = brown + yellow;

    for (int height = 1; height * height <= total; height++) {
        if (total % height != 0) {
            continue;
        }

        int width = total / height;

        if ((width - 2) * (height - 2) == yellow) {
            return {width, height};
        }
    }

    return {};
}
```

이 풀이에서 `height * height <= total`까지만 보는 이유는 약수쌍이 대칭이기 때문이다.

예를 들어:

```text
48 = 8 * 6
48 = 6 * 8
```

둘은 같은 카펫 크기이고, 문제에서는 가로가 세로보다 크거나 같아야 하므로 작은 쪽을 `height`로 잡고 큰 쪽을 `width`로 계산하면 된다.

## Complexity

내 풀이는 `brown`을 기준으로 가능한 테두리 분할을 하나씩 확인한다.

`brown`은 최대 5,000이므로 충분히 작다.

시간복잡도는 `O(brown)`으로 볼 수 있다.

약수쌍 풀이는 `sqrt(brown + yellow)`까지만 확인하므로 시간복잡도는 `O(sqrt(total))`이다.

추가 자료구조를 사용하지 않으므로 공간복잡도는 `O(1)`이다.

## Notes

- 내 풀이는 갈색 테두리의 구조를 직접 분해한 풀이이다.
- 약수쌍 풀이는 전체 면적에서 가로/세로 후보를 찾는 풀이이다.
- 카펫 문제의 핵심 식은 `(width - 2) * (height - 2) == yellow`이다.
- 통과만 보면 `row * col == brown + yellow`도 가능하지만, 복습 관점에서는 내부 노란 영역 식이 더 명확하다.
