---
title: '[PG Level2] 괄호 회전하기 (Node.js)'
date: 2022-04-08 20:15:13
category: 'PS'
draft: false
---

## 문제

[코딩테스트 연습 - 괄호 회전하기](https://programmers.co.kr/learn/courses/30/lessons/76502)

### 문제 풀이

문자열을 왼쪽으로 회전시키면서 올바른 괄호인지 판별하는 문제다. 회전하지 않은 문자열에 대해서도 올바른 괄호인지 판별해야하기 때문에 먼저 올바른 괄호인지 여부를 판단한다. 그리고`shift()`로 배열의 맨 앞 요소를 꺼내고, 이를 맨 뒤에 넣어준다. 그리고 count 를 1씩 증가시켜준다. 배열의 길이와 같아지면 while loop를 종료한다.

isValidBracket이라는 함수를 선언하고 올바른 괄호인지 여부를 판별했다. bracket 객체를 선언했다. key는 닫는 괄호, value는 여는 괄호가 된다. string을 순회하면서 여는 괄호라면 stack에 넣어준다. 닫는 괄호인 경우에는 먼저 스택에 요소가 있어야하고, 해당 bracket object에 해당 하는 값이 매핑되는지 확인한다. 매핑되지 않는다면 올바른 괄호가 아니기 때문에 false를 반환한다. 순회가 종료되었을 때 stack에 요소가 남아있다면 이는 여는 괄호가 남아있다는 뜻이고 즉 올바른 괄호 문자열이 아니다. 따라서 stack 의 요소가 있다면 false를 없다면 올바른 괄호 문자열을 의미하는 true를 반환한다.

### Code

```jsx
const log = console.log

function solution(s) {
  let count = 0,
    answer = 0
  let array = [...s]

  while (count < array.length) {
    if (isValidBracket(array)) answer++
    const firstElement = array.shift()
    array = [...array, firstElement]
    count++
  }

  return answer
}

function isValidBracket(string) {
  const bracket = { ']': '[', ')': '(', '}': '{' }
  const stack = []

  for (let i = 0; i < string.length; i++) {
    if (!Object.keys(bracket).includes(string[i])) stack.push(string[i])
    else {
      if (stack && bracket[string[i]] === stack[stack.length - 1]) stack.pop()
      else return false
    }
  }

  return stack.length > 0 ? false : true
}
```

### Recap

올바른 괄호 문자열인지 판별하는 함수에서 스택에 요소가 남아있는 경우를 처음에는 고려하지 못했다. 올바른 괄호와 관련된 문제도 정말 많이 접했던 것 같은데, 이제는 실수하지 말아야겠다.
