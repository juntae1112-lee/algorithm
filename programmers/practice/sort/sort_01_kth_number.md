# Sort 01 - Kth Number

## Problem

- Platform: Programmers
- Category: Sort
- Title: Kth Number
- Korean title: K번째수

## Problem Summary

배열 `array`와 명령어 배열 `commands`가 주어진다.

각 명령어는 `[i, j, k]` 형태이다.

명령어마다 다음 작업을 수행한다.

```text
1. array의 i번째 숫자부터 j번째 숫자까지 자른다.
2. 자른 배열을 정렬한다.
3. 정렬된 배열의 k번째 숫자를 결과에 넣는다.
```

모든 명령어의 결과를 순서대로 담아 반환한다.

## Quick Summary

- 문제의 `i`, `j`, `k`는 1-based index이다.
- C++ `vector` slicing은 iterator 범위로 한다.
- C++ 범위는 `[start, end)`이므로 끝 iterator는 포함되지 않는다.
- 따라서 `i`번째부터 `j`번째까지 자르려면 `array.begin() + (i - 1)`부터 `array.begin() + j`까지 사용한다.
- 자른 배열을 정렬한 뒤 `k - 1` index를 answer에 넣는다.

## Constraints

- `array`의 길이는 1 이상 100 이하이다.
- `array`의 각 원소는 1 이상 100 이하이다.
- `commands`의 길이는 1 이상 50 이하이다.
- `commands`의 각 원소는 길이 3이다.

## My Approach

각 command를 순회하면서 `i`, `j`, `k`를 꺼낸다.

문제의 index는 1부터 시작하므로 C++ index로 바꿀 때는 `i - 1`, `k - 1`을 사용한다.

자른 배열은 새 `vector<int>`로 만든다.

```cpp
vector<int> sliced(array.begin() + (i - 1), array.begin() + j);
```

이후 `sliced`를 정렬하고 `sliced[k - 1]`을 결과에 추가한다.

## Thinking Process

예시:

```text
array = [1, 5, 2, 6, 3, 7, 4]
command = [2, 5, 3]
```

문제 기준으로 2번째부터 5번째까지 자르면:

```text
[5, 2, 6, 3]
```

C++ index로는 1번부터 4번까지 포함해야 한다.

`vector`의 범위는 끝을 포함하지 않기 때문에 다음처럼 자른다.

```cpp
array.begin() + (2 - 1)  // index 1 포함
array.begin() + 5        // index 5 미포함
```

결과적으로 index `1, 2, 3, 4`가 잘린다.

정렬하면:

```text
[2, 3, 5, 6]
```

3번째 숫자는 C++ index로 `2`이므로 `sliced[3 - 1] = 5`이다.

## Key Point

이 문제의 핵심은 index 변환이다.

문제의 `i`, `j`, `k`는 1-based이고, C++ `vector`는 0-based이다.

또 C++ iterator 범위는 `[start, end)`이므로 `j`번째까지 포함하려면 끝 iterator를 `array.begin() + j`로 둔다.

정리하면:

- 시작 위치: `i - 1`
- 끝 위치: `j`
- 정답 위치: `k - 1`

## Accepted Solution

```cpp
#include <string>
#include <vector>
#include <algorithm>
#include <iostream>

using namespace std;

vector<int> solution(vector<int> array, vector<vector<int>> commands) {
    vector<int> answer;
    for (auto cmd: commands) {
        int i = cmd[0];
        int j = cmd[1];
        int k = cmd[2];
        vector<int> sliced(array.begin() + (i - 1), array.begin() + j);
        sort(sliced.begin(), sliced.end());
        answer.push_back(sliced[k - 1]);
    }
    return answer;
}
```

## Complexity

명령어 개수를 `m`, 잘라낸 배열의 길이를 최대 `n`이라고 하면 각 명령어마다 정렬이 필요하다.

```text
명령어 하나당: O(n log n)
전체: O(m * n log n)
```

제한이 작기 때문에 이 방식으로 충분하다.

공간복잡도는 잘라낸 배열을 저장하므로 `O(n)`이다.

## Notes

- `vector<int> sliced(start, end)`는 원본을 수정하지 않고 새 vector를 만든다.
- `sort`를 쓰려면 `<algorithm>`이 필요하다.
- 이 문제는 제한이 작아서 매번 잘라서 정렬해도 충분하다.
