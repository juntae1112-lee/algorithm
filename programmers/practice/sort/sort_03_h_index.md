# Sort 03 - H-Index

## Problem

- Platform: Programmers
- Category: Sort
- Title: H-Index
- Korean title: H-Index

## Problem Summary

논문별 인용 횟수 배열 `citations`가 주어진다.

H-Index는 다음 조건을 만족하는 `h`의 최댓값이다.

```text
h번 이상 인용된 논문이 h편 이상이다.
```

문제 설명에는 “나머지 논문이 h번 이하 인용되었다”는 표현도 있지만, 핵심은 합계가 아니라 논문 개수이다.

## Quick Summary

- H-Index는 인용 횟수의 합을 보는 문제가 아니다.
- `h번 이상 인용된 논문 수 >= h`를 만족하는 최대 `h`를 찾는 문제이다.
- `h`는 반드시 `citations` 안에 있는 값일 필요가 없다.
- 정렬 후 오른쪽에 남은 논문 수를 이용하면 효율적으로 계산할 수 있다.
- 오름차순 정렬에서 `i`번째 논문 이상 인용된 논문 수는 `n - i`이다.

## Constraints

- 논문 수는 1편 이상 1,000편 이하이다.
- 논문별 인용 횟수는 0회 이상 10,000회 이하이다.

## My Approach

처음에는 정렬 후 각 인용 수마다 “h번 이상 인용된 논문 수”를 세면 된다고 생각했다.

예시:

```text
citations = [3, 0, 6, 1, 5]
정렬 = [0, 1, 3, 5, 6]
```

각 값 이상 인용된 논문 수는 다음과 같다.

```text
0 이상 인용된 논문: 5편
1 이상 인용된 논문: 4편
3 이상 인용된 논문: 3편
5 이상 인용된 논문: 2편
6 이상 인용된 논문: 1편
```

이때 `3` 이상 인용된 논문이 3편이므로 답은 `3`이다.

하지만 여기서 중요한 점은 `h`가 꼭 배열 안의 인용 수일 필요는 없다는 것이다.

예:

```text
[4, 9, 37, 55, 77, 100]
```

이 배열에서 `5`는 원소가 아니지만, 5번 이상 인용된 논문은 5편이므로 H-Index는 `5`가 될 수 있다.

처음 작성한 코드는 배열 안에 있는 인용 수만 `h` 후보로 봤다.

```cpp
int solution(vector<int> citations) {
    int answer = 0;
    sort(citations.begin(), citations.end());
    for (int i = 0; i < citations.size(); i++) {
        int h = citations[i];
        int temp = citations.size() - i;
        cout << h << " " << temp << endl;
        if (temp >= h) {
            answer = h;
        } else {
            break;
        }
    }

    return answer;
}
```

이 접근의 문제는 `h`가 배열 안의 값일 때만 확인한다는 점이다.

예를 들어:

```text
[3, 9, 37, 55, 77, 100]
```

에서는 `5`가 배열에 없지만, `5`번 이상 인용된 논문이 5편이므로 답은 `5`가 될 수 있다.

## Thinking Process

정렬된 배열을 기준으로 오른쪽에 몇 개의 논문이 남는지를 본다.

```text
[0, 1, 3, 5, 6]
```

각 h 후보를 보면:

```text
h = 0 -> 0번 이상 인용된 논문 5편 -> 만족
h = 1 -> 1번 이상 인용된 논문 4편 -> 만족
h = 2 -> 2번 이상 인용된 논문 3편 -> 만족
h = 3 -> 3번 이상 인용된 논문 3편 -> 만족
h = 4 -> 4번 이상 인용된 논문 2편 -> 불만족
h = 5 -> 5번 이상 인용된 논문 2편 -> 불만족
```

답은 `3`이다.

그래서 배열에 있는 값만 보는 것이 아니라, 인용 수 사이의 빈 구간도 봐야 한다고 생각했다.

예를 들어 `[0, 1, 3, 5, 6]`은 이렇게 나눌 수 있다.

```text
i = 0; h = 0      -> 5편
i = 1; h = 1      -> 4편
i = 2; h = 2, 3   -> 3편
i = 3; h = 4, 5   -> 2편
i = 4; h = 6      -> 1편
```

이 생각을 코드로 옮긴 시도는 다음과 같다.

```cpp
int solution(vector<int> citations) {
    int answer = 0;
    sort(citations.begin(), citations.end());
    int lastH = 0;
    for (int i = 0; i < citations.size(); i++) {
        int h = citations.at(i);
        for (int j = lastH; j <= h; j++) {
            int cnt = citations.size() - i;
            if (h <= cnt) {
                answer = h;
            }
        }
        lastH = h + 1;
    }

    return answer;
}
```

방향은 맞았다.

배열에 없는 `h` 후보까지 보려고 했고, 정렬된 위치 `i`에서 오른쪽 논문 수가 `citations.size() - i`라는 것도 맞게 잡았다.

다만 코드에서는 `j`를 순회해 놓고 실제 조건에는 `h`를 쓰는 실수가 있었다.

```cpp
if (h <= cnt) {
    answer = h;
}
```

구간 후보를 직접 검사하려면 `j`를 써야 한다.

```cpp
if (j <= cnt) {
    answer = j;
}
```

이 방식으로도 정리할 수 있지만, 더 단순하게 보면 각 위치에서 가능한 최대 후보는 다음과 같다.

```text
candidate = min(citations[i], n - i)
```

이유:

- `citations[i]` 이상 인용된 논문 수는 `n - i`편이다.
- H-Index 후보는 인용 수보다 클 수 없다.
- H-Index 후보는 해당 논문 수보다 클 수도 없다.

따라서 각 위치에서 `min(citations[i], n - i)`를 구하고, 그중 최댓값을 답으로 잡는다.

### Why `min(citations[i], n - i)` Works

오름차순 정렬 후 `i`번째 값을 기준으로 보면:

```text
citations[i], citations[i + 1], ..., citations[n - 1]
```

이 논문들은 모두 `citations[i]`번 이상 인용된 논문이다.

이 개수는:

```text
n - i
```

이다.

즉 `i`번째 위치에서는 다음 두 값이 동시에 중요하다.

```text
citations[i] = 이 구간의 최소 인용 수
n - i        = 이 구간에 있는 논문 수
```

H-Index 후보 `h`는 이 둘 중 더 큰 값을 넘을 수 없다.

예를 들어:

```text
citations[i] = 9
n - i = 5
```

라면, 9번 이상 인용된 논문은 5편뿐이다.

이때 H-Index는 9가 될 수 없다. 9번 이상 인용된 논문이 9편 있어야 하기 때문이다.

하지만 `h = 5`는 가능하다.

```text
5번 이상 인용된 논문이 5편 이상
```

이므로 조건을 만족한다.

반대로:

```text
citations[i] = 3
n - i = 5
```

라면, 이 구간에 논문은 5편 있지만 기준 인용 수가 3이다.

이 위치만 기준으로는 `h = 5`라고 말할 수 없다. 현재 구간의 최소 인용 수가 3이기 때문이다.

그래서 이 위치에서 만들 수 있는 최대 후보는:

```text
min(citations[i], n - i)
```

가 된다.

모든 위치에서 이 후보를 계산하고 최댓값을 고르면 H-Index가 된다.

## Mistakes And Lessons

첫 번째 실수는 H-Index를 인용 횟수의 합으로 오해한 것이다.

문제에서 보는 것은 합이 아니라 개수이다.

```text
h번 이상 인용된 논문이 몇 편인가
```

두 번째 실수는 `h`를 배열 안에 있는 citation 값으로만 본 것이다.

하지만 `h`는 0부터 논문 수 `n`까지 가능한 지표 값이다. 배열 안에 없는 값도 H-Index가 될 수 있다.

세 번째 실수는 구간을 순회하면서 `j`를 돌렸지만, 실제 조건에는 `j`가 아니라 `h`를 사용한 것이다.

실수 코드의 핵심은 다음과 같다.

```cpp
for (int j = lastH; j <= h; j++) {
    int cnt = citations.size() - i;
    if (h <= cnt) {
        answer = h;
    }
}
```

이렇게 하면 `j`를 돌고 있지만 `j`가 전혀 사용되지 않는다.

구간 후보를 직접 검사하려면 다음처럼 봐야 한다.

```cpp
if (j <= cnt) {
    answer = j;
}
```

하지만 더 단순한 방식은 구간을 모두 돌지 않고 `min(citations[i], n - i)`를 후보로 보는 것이다.

중요한 점은 내 생각을 버리는 것이 아니라, 내 생각을 더 압축하면 `min(citations[i], n - i)`가 된다는 것이다.

## Key Point

H-Index의 핵심 조건은 다음이다.

```text
h 이상 인용된 논문 수 >= h
```

오름차순 정렬 후 `i`번째 위치에서:

```text
citations[i] 이상 인용된 논문 수 = n - i
```

따라서 해당 위치의 후보는:

```text
min(citations[i], n - i)
```

이고, 전체 후보 중 최댓값이 답이다.

코드로는 다음 세 줄이 핵심이다.

```cpp
int count = n - i;
int candidate = min(citations[i], count);
answer = max(answer, candidate);
```

## Accepted Solution

구간의 `h` 후보를 직접 확인하는 방식이다.

내가 생각한 “현재 위치에서 가능한 h들을 확인한다”는 흐름을 최대한 살린 코드다.

```cpp
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

int solution(vector<int> citations) {
    int answer = 0;
    sort(citations.begin(), citations.end());

    int lastH = 0;
    int n = citations.size();
    for (int i = 0; i < n; i++) {
        int h = citations[i];
        int count = n - i;
        for (int candidate = lastH; candidate <= h; candidate++) {
            if (candidate <= count) {
                answer = candidate;
            }
        }
        lastH = h + 1;
    }

    return answer;
}
```

## Alternative Solution

위 생각을 더 압축하면 구간을 하나씩 돌 필요 없이 `min(citations[i], n - i)`만 보면 된다.

```cpp
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

int solution(vector<int> citations) {
    int answer = 0;
    sort(citations.begin(), citations.end());

    int n = citations.size();
    for (int i = 0; i < n; i++) {
        int count = n - i;
        int candidate = min(citations[i], count);
        answer = max(answer, candidate);
    }

    return answer;
}
```

## Complexity

내 풀이처럼 구간 후보를 직접 확인하면 정렬에 `O(n log n)`, 후보 확인에 최대 인용 수만큼의 순회가 들어간다.

문제 제한에서 인용 횟수는 최대 10,000이므로 이 방식도 충분히 통과 가능하다.

개선 풀이처럼 `min(citations[i], n - i)`만 확인하면 정렬 후 한 번만 순회한다.

정렬이 필요하므로 시간복잡도는 `O(n log n)`이다.

정렬 후 한 번 순회하므로 추가 순회는 `O(n)`이다.

전체 시간복잡도는 `O(n log n)`이다.

추가 자료구조를 사용하지 않으므로 공간복잡도는 `O(1)`로 볼 수 있다.

## Notes

- H-Index는 인용 수 합계가 아니라 조건을 만족하는 논문 개수 문제이다.
- `h`는 배열 원소일 필요가 없다.
- 제한이 작아서 `h = 0 ~ n`을 직접 세는 방식도 통과는 가능하다.
- 하지만 정렬 후 `min(citations[i], n - i)`를 쓰면 더 깔끔하다.
