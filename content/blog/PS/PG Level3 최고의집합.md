---
title: '[PG Level3] 최고의 집합 (Node.js)'
date: 2022-04-05 18:31:13
category: 'PS'
draft: false
---

## 문제

[코딩테스트 연습 - 최고의 집합](https://programmers.co.kr/learn/courses/30/lessons/12938)

자연수 n 개로 이루어진 중복 집합(multi set, 편의상 이후에는 "집합"으로 통칭) 중에 다음 두 조건을 만족하는 집합을 최고의 집합이라고 합니다.

1. 각 원소의 합이 S가 되는 수의 집합
2. 위 조건을 만족하면서 각 원소의 곱 이 최대가 되는 집합

집합의 원소의 개수 n과 모든 원소들의 합 s가 매개변수로 주어질 때, 최고의 집합을 return 하는 solution 함수를 완성해주세요.

### 문제 풀이

합이 s이고, 크기가 n인 경우 s를 n으로 균등하게 나눴을 때 각각의 숫자의 곱이 최대의 곱이 된다.

먼저 s를 n으로 나눈 값에 대해 버림 연산을 한 값을 배열의 초기값으로 설정한다. 해당 초기값을 갖는 크기가 n인 배열을 선언한다. 배열의 합, 즉 `초기값 * n` 이 s보다 작다면 그 차이를 계산한다. 결과값을 오름차순으로 반환해야하기 때문에 뒤에 index부터 차이만큼의 index의 요소 값을 1씩 더해준다.

예를 들어 n = 3 이고, s = 14 인 경우, 차이가 2이다. 따라서 배열 요소의 index가 1과 2인 요소를 1씩 더해준다.

마지막으로 결과를 반환할 때 배열의 값이 0을 포함한다면 자연수라는 조건을 만족하지 않기 때문에 `[-1]` 을 반환해준다.

### Code

```jsx
function solution(n, s) {
  const initialValue = Math.floor(s / n)
  const array = new Array(n).fill(initialValue)
  const diff = s - initialValue * n
  if (diff > 0) {
    for (let i = n - diff; i <= n - 1; i++) {
      array[i]++
    }
  } else return array
  return array.includes(0) ? [-1] : array
}
```

### Recap

특정 수의 곱의 최대 값은 그 수를 균등하게 나눈 수들의 곱이라는 사실을 알 수 있었다. 어렵지 않은 문제였지만 Level3 문제를 스스로 풀어낼 수 있어서 최근 꾸준히 알고리즘 문제를 공부한 보람을 느낄 수 있었다.
