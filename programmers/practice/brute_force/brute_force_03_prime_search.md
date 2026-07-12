# Brute Force 03 - Prime Search

## Problem

- Platform: Programmers
- Category: Brute Force
- Title: Prime Search
- Korean title: 소수 찾기

## Problem Summary

숫자가 적힌 종이 조각들이 문자열 `numbers`로 주어진다.

종이 조각을 일부 또는 전부 사용해서 만들 수 있는 숫자 중 소수가 몇 개인지 구해야 한다.

같은 숫자는 한 번만 세야 한다.

## Quick Summary

- 각 자리 숫자를 선택하는 모든 순열을 만들어야 한다.
- 길이 1부터 `numbers.size()`까지의 숫자를 모두 만들 수 있다.
- 같은 숫자가 여러 번 만들어질 수 있으므로 `unordered_set<int>`로 중복 제거한다.
- 숫자를 모두 만든 뒤, 각 숫자가 소수인지 검사한다.
- 재귀에서 현재 문자열을 직접 누적하면 같은 함수의 다음 반복에 영향을 줄 수 있으므로 `num_temp`를 만들어 넘긴다.

## Constraints

- `numbers`의 길이는 1 이상 7 이하이다.
- `numbers`는 0부터 9까지의 숫자로만 이루어져 있다.
- 앞에 0이 붙은 숫자는 같은 숫자로 취급한다.
  - 예: `011`은 `11`과 같은 숫자이다.

## My Approach

종이 조각을 하나씩 선택하면서 만들 수 있는 숫자를 재귀로 생성했다.

이미 사용한 인덱스는 `vector<bool> used`로 표시했다.

```cpp
vector<bool> used(size, false);
```

만든 숫자는 중복될 수 있으므로 `unordered_set<int>`에 넣었다.

예를 들어 `"011"`에서는 다음처럼 같은 숫자가 여러 번 만들어질 수 있다.

```text
0 선택 -> 1 선택 -> 1 선택 -> 011 -> 11
1 선택 -> 1 선택               -> 11
```

따라서 `vector<int>`에 바로 넣으면 중복 처리가 필요하다.

`unordered_set<int>`을 쓰면 같은 숫자가 여러 번 들어와도 한 번만 저장된다.

## Thinking Process

문제의 핵심은 두 단계다.

```text
1. 만들 수 있는 모든 숫자를 생성한다.
2. 생성된 숫자 중 소수의 개수를 센다.
```

숫자 생성은 재귀로 처리했다.

현재까지 만든 문자열 `num`이 있고, 아직 사용하지 않은 숫자를 하나 붙여 다음 재귀로 들어간다.

```text
numbers = "17"

1 선택 -> 1
1 선택 -> 7 선택 -> 17

7 선택 -> 7
7 선택 -> 1 선택 -> 71
```

이때 `1`, `17`, `7`, `71`을 모두 후보로 저장한다.

소수 검사는 `1` 이하를 제외하고, 나누어떨어지는 수가 있는지 확인했다.

## Mistakes And Lessons

처음에는 재귀 함수 안의 `for`문에서 `num`을 직접 수정했다.

```cpp
num += numbers.at(i);
used[i] = true;
s.insert(stoi(num));
create_num(numbers, used, num, s);
```

함수 인자로 넘어갈 때 `num`은 복사된다.

하지만 문제는 재귀 호출이 아니라, 현재 함수의 `for`문 안에서 `num`이 계속 누적된다는 점이다.

예를 들어 현재 함수의 `num`이 `"1"`이라고 하면:

```text
i = 1
num += '2' -> "12"
재귀 호출
현재 함수로 돌아옴

i = 2
num += '3' -> "123"
```

원래는 `"13"`을 만들고 싶었는데, 이전 반복에서 붙인 `"2"`가 남아서 `"123"`이 된다.

그래서 루프 안에서는 현재 값을 직접 바꾸지 않고, 다음 재귀에 넘길 값을 따로 만들어야 한다.

```cpp
string num_temp = num;
vector<bool> used_temp = used;
num_temp += numbers.at(i);
used_temp[i] = true;
create_num(numbers, used_temp, num_temp, s);
```

또 하나의 개선점은 소수 판별이다.

처음에는 `2`부터 `num - 1`까지 전부 확인했다.

```cpp
for (int i = 2; i < num; i++) {
    if ((num % i) == 0) {
        ret = false;
    }
}
```

이 방식도 제한에서는 통과 가능하지만, 약수를 찾으면 바로 `false`를 반환하고 `sqrt(num)`까지만 확인하면 더 효율적이다.

## Key Point

재귀로 순열을 만들 때는 다음 두 가지를 분리해서 생각한다.

```text
현재 상태
다음 재귀에 넘길 상태
```

현재 함수의 루프에서 상태를 직접 누적하면 다음 반복에 영향을 줄 수 있다.

따라서 다음 상태를 복사해서 만든 뒤 재귀로 넘기는 방식이 안전하다.

중복 숫자는 `unordered_set<int>`으로 제거한다.

```cpp
unordered_set<int> s;
s.insert(stoi(num));
```

## Accepted Solution

내가 작성한 방식이다.

재귀로 만들 수 있는 모든 숫자를 생성하고, `unordered_set`으로 중복 제거한 뒤 소수 개수를 센다.

```cpp
#include <string>
#include <vector>
#include <iostream>
#include <unordered_set>

using namespace std;

bool check(int num) {
    if (num <= 1) {
        return false;
    }
    bool ret = true;
    for (int i = 2; i < num; i++) {
        if ((num % i) == 0) {
            ret = false;
        }
    }
    return ret;
}

void create_num(string numbers, vector<bool> used, string num, unordered_set<int> &s) {
    s.insert(stoi(num));
    int size = numbers.size();
    for (int i = 0; i < size; i++) {
        if (true == used[i]) {
            continue;
        }
        string num_temp = num;
        vector<bool> used_temp = used;
        num_temp += numbers.at(i);
        used_temp[i] = true;
        create_num(numbers, used_temp, num_temp, s);
    }
}

int solution(string numbers) {
    int answer = 0;
    int size = numbers.size();
    unordered_set<int> s;

    for (int i = 0; i < size; i++) {
        string num;
        num += numbers.at(i);
        vector<bool> used(size, false);
        used[i] = true;
        create_num(numbers, used, num, s);
    }

    for (auto i : s) {
        cout << i << endl;
        if (true == check(i)) {
            answer++;
        }
    }

    return answer;
}
```

## Alternative Solution

소수 판별을 개선하면 불필요한 반복을 줄일 수 있다.

약수는 제곱근까지만 확인하면 된다.

```cpp
#include <string>
#include <vector>
#include <unordered_set>

using namespace std;

bool is_prime(int num) {
    if (num <= 1) {
        return false;
    }

    for (int i = 2; i * i <= num; i++) {
        if (num % i == 0) {
            return false;
        }
    }

    return true;
}

void make_numbers(const string &numbers, vector<bool> used, string current, unordered_set<int> &result) {
    result.insert(stoi(current));

    for (int i = 0; i < numbers.size(); i++) {
        if (used[i]) {
            continue;
        }

        string next = current + numbers[i];
        vector<bool> next_used = used;
        next_used[i] = true;

        make_numbers(numbers, next_used, next, result);
    }
}

int solution(string numbers) {
    unordered_set<int> candidates;

    for (int i = 0; i < numbers.size(); i++) {
        vector<bool> used(numbers.size(), false);
        used[i] = true;

        string current;
        current += numbers[i];

        make_numbers(numbers, used, current, candidates);
    }

    int answer = 0;
    for (int num : candidates) {
        if (is_prime(num)) {
            answer++;
        }
    }

    return answer;
}
```

## Complexity

`numbers`의 길이를 `n`이라고 하면 만들 수 있는 숫자는 순열 기반으로 생성된다.

길이 1부터 `n`까지의 순열을 만들므로 대략 다음 규모이다.

```text
P(n, 1) + P(n, 2) + ... + P(n, n)
```

`n`은 최대 7이므로 재귀 완전탐색이 가능하다.

각 후보 숫자에 대해 소수 판별을 한다.

기존 소수 판별은 `O(num)`에 가깝고, 개선한 소수 판별은 `O(sqrt(num))`이다.

공간복잡도는 생성된 후보 숫자를 저장하는 `unordered_set` 크기에 비례한다.

## Notes

- `stoi("011")`은 `11`이 되므로 앞자리 0 처리가 자연스럽게 된다.
- 같은 숫자가 여러 방식으로 만들어질 수 있으므로 중복 제거가 필요하다.
- `unordered_set`은 중복 제거에 적합하다.
- 재귀에서 상태를 직접 수정할지, 복사해서 넘길지 구분해야 한다.
- 이번 실수의 핵심은 `num`이 함수 호출로 복사되는지는 맞지만, 같은 함수의 다음 반복에서 이미 바뀐 `num`이 유지된다는 점이었다.
