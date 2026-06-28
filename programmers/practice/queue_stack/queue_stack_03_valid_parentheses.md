# Queue/Stack 03 - Valid Parentheses

## Problem

- Platform: Programmers
- Category: Queue / Stack
- Title: Valid Parentheses
- Korean title: 올바른 괄호

## Problem Summary

문자열은 `'('` 또는 `')'`로만 이루어져 있다.

괄호가 올바르려면 열린 괄호 `'('`가 나오면 반드시 나중에 닫힌 괄호 `')'`로 닫혀야 한다. 닫힌 괄호가 먼저 나오거나, 끝까지 봤는데 열린 괄호가 남아 있으면 올바른 괄호가 아니다.

## Constraints

- 문자열 `s`의 길이는 100,000 이하의 자연수이다.
- 문자열 `s`는 `'('` 또는 `')'`로만 이루어져 있다.

## My Solving Approach

이 문제는 기초적인 stack 문제로 볼 수 있다.

내 아이디어는 stack의 top에 있는 괄호가 열린 괄호인지 확인하고, 현재 문자가 그 열린 괄호와 쌍이 되는 닫힌 괄호라면 stack에서 pop하는 방식이다.

## Stack Thinking

문자열을 앞에서부터 하나씩 확인한다.

- `'('`를 만나면 아직 닫히지 않은 열린 괄호이므로 stack에 넣는다.
- `')'`를 만나면 stack의 top을 확인한다.
- stack의 top에 `'('`가 있으면 서로 짝이 맞으므로 pop한다.
- stack이 비어 있는데 `')'`가 나오면 닫을 열린 괄호가 없으므로 올바르지 않다.
- stack의 top과 현재 닫힌 괄호가 쌍으로 맞지 않는 경우도 올바르지 않다.

모든 문자를 확인한 뒤에는 stack이 비어 있어야 한다.

stack이 비어 있으면 모든 괄호가 제대로 짝지어진 것이고, stack에 값이 남아 있으면 아직 닫히지 않은 열린 괄호가 있다는 뜻이다.

## Key Point

닫힌 괄호 `')'`가 나왔을 때 바로 판단할 수 있다.

이때 stack이 비어 있으면 닫을 대상이 없기 때문에 바로 `false`가 된다. 반대로 끝까지 검사했는데 stack이 비어 있지 않으면 열린 괄호가 남은 것이므로 이것도 `false`이다.

## Examples

- `"()()"` -> `true`
- `"(())()"` -> `true`
- `")()("` -> `false`
- `"(()("` -> `false`

## My Notes

- 열린 괄호는 stack에 쌓는다.
- 닫힌 괄호를 만나면 stack의 top에 있는 열린 괄호와 짝이 맞는지 확인한다.
- 짝이 맞으면 pop한다.
- 짝이 안 맞거나, 닫힌 괄호를 만났는데 stack이 비어 있으면 바로 실패이다.
- 마지막에는 stack이 비어 있는지가 최종 정답 기준이다.

## Accepted Solution

```cpp
#include <string>
#include <iostream>
#include <stack>

using namespace std;

bool solution(string s)
{
    bool answer = true;

    stack<char> temp;
    for (auto c: s) {
        // 열림 괄호일 경우 stack에 push
        if ('(' == c) {
            temp.push(c);
        } else { // 닫힘 괄호일 경우 stack의 top이 열림인지 확인
            if (true == temp.empty()) {
                answer = false;
                break;
            } else {
                if ('(' == temp.top()) {
                    temp.pop();
                } else {
                    answer = false;
                    break;
                }
            }
        }
    }

    // 끝까지 확인했는데 열린 괄호가 남아 있으면 올바른 괄호가 아니다.
    if (false == temp.empty()) {
        answer = false;
    }

    return answer;
}
```
