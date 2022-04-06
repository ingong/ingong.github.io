---
title: '[PG Level2] 카카오 양궁 (Node.js)'
date: 2022-03-31 17:11:13
category: 'PS'
draft: false
---

## 문제

[코딩테스트 연습 - 양궁대회](https://programmers.co.kr/learn/courses/30/lessons/92342)

### 문제 풀이

처음에는 어떻게 풀어야할지 막막했는데 DFS를 활용해 문제를 푸는 방식이였다. n, index, array를 인자로 갖는 함수를 작성하고 재귀 호출하도록 작성했다. 처음에는 n이 0이 될 때 DFS를 종료하도록 조건을 작성했는데 모든 테스트 케이스는 통과했지만 마지막 네 번째 테스트 케이스는 통과하지 못했다. 따라서 n이 아닌 index를 기준으로 종료조건을 명시했다.

종료조건을 만족한 경우 만약 n이 남아있다면 마지막 Index에 남은 n들을 더해주고, 차이 합이 최대 값보다 크다면 최대 값을 최신화하고 결과 배열에 해당 배열을 넣어준다. 최대값이 -Infinity라면 [-1]을 return 한다. 그렇지 않다면 문제의 조건에서 최대값을 갖는 배열 중 낮은 점수의 횟수가 높은 것을 기준으로 정렬 후 첫 번째 배열 요소를 return한다.

### Recap

정렬에 대한 콜백 함수 작성에서 시간을 많이 소요했다. 차이가 0 보다 크다면 true를 반환하도록 sort의 콜백함수를 작성했는데 이상하게 정렬이 되지 않았다. 처음에는 익숙하지 않아서 실수를 했나보다 생각하면서 곰곰히 찾아보고 결국 MDN 문서를 찾아봤다. MDN문서에는 sort의 콜백 함수에 대해서 다음과 같이 나와있다.

> `compareFunction`이 제공되면 배열 요소는 compare 함수의 반환 값에 따라 정렬됩니다. a와 b가 비교되는 두 요소라면,

- `compareFunction(a, b)`이 0보다 작은 경우 a를 b보다 낮은 색인으로 정렬합니다. 즉, a가 먼저옵니다.
- `compareFunction(a, b)`이 0을 반환하면 a와 b를 서로에 대해 변경하지 않고 모든 다른 요소에 대해 정렬합니다. 참고 : ECMAscript 표준은 이러한 동작을 보장하지 않으므로 모든 브라우저(예 : Mozilla 버전은 적어도 2003 년 이후 버전 임)가 이를 존중하지는 않습니다.
- `compareFunction(a, b)`이 0보다 큰 경우, b를 a보다 낮은 인덱스로 소트합니다.
  >

그리고 예시는 다음과 같다.

```tsx
function compare(a, b) {
  if (a is less than b by some ordering criterion) {
    return -1;
  }
  if (a is greater than b by the ordering criterion) {
    return 1;
  }
  // a must be equal to b
  return 0;
}
```

나는 1과 -1를 return하는 것이 아닌 true와 false를 return했기 때문에 정상적으로 동작하지 않은 것이였다. 당연하게 썼는데 조금 긴장하다보니 알던 내용도 실수해서 작성을 한 것 같다. 그래도 이전까지는 카카오 알고리즘 문제는 정말 어려울 것 같다는 막연한 두려움을 갖고 시도해보지 않았는데 최근에는 스스로 힘으로 푸는 경험을 하면서 자신감을 얻게 된 것 같아 기분이 좋다 :)

### Code

```tsx
const log = console.log

function solution(n, info) {
  const results = []
  const array = new Array(11).fill(0)
  let maxSum = -Infinity
  dfs(n, 0, array)

  if (maxSum === -Infinity) return [-1]
  else {
    const resultWithOutOne = [...results].filter(v => v !== -1)
    const candidates = resultWithOutOne.filter(v => getDiffSum(v) === maxSum)
    candidates.sort((a, b) => sortByBehindMax(a, b))
    return candidates[0]
  }

  function sortByBehindMax(a, b) {
    for (let i = 10; i >= 0; i--) {
      if (a[i] - b[i] > 0) return -1
      else if (a[i] - b[i] < 0) return 1
      else continue
    }
  }

  function getDiffSum(array) {
    return array
      .map((v, i) => {
        if (v - info[i] > 0) return 10 - i
        else if (v - info[i] < 0) return -(10 - i)
        else return 0
      })
      .reduce((a, b) => a + b, 0)
  }

  function dfs(n, index, array) {
    if (index === 12) {
      const tempArr = array
      if (n !== 0) tempArr[10] += n
      const sum = getDiffSum(tempArr)
      if (sum > 0) {
        if (sum >= maxSum) {
          results.push(tempArr)
          maxSum = sum
        }
      } else {
        if (!results.includes(-1)) results.push([-1])
      }
      return
    }

    if (n > info[index]) {
      const temp = [...array]
      const count = info[index] + 1
      temp[index] = count
      dfs(n - count, index + 1, temp)
    }
    dfs(n, index + 1, array)
  }
}
```
