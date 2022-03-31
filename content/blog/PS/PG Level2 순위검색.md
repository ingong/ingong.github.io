---
title: '[PG Level2] 카카오 순위검색 (Node.js)'
date: 2022-03-29 16:21:13
category: 'PS'
draft: false
---

## 문제

[코딩테스트 연습 - 순위 검색](https://programmers.co.kr/learn/courses/30/lessons/72412)

## 문제 풀이

info를 순회하면서 해당 조건을 모두 만족시키는 경우의 수를 모두 만들어서 문자열로 합쳐준다. 이 때 Combination을 통해서 만들어준다.

```tsx
java backend junior pizza 150
```

해당 조건을 갖는 사원에 대한 정보는 총 16가지에 포함되도록 만들어 질 수 있다. 예를 들면 `[-, backend, junior, pizza]`, `[java, -, junior, pizza]` 등등 2^4 의 경우의 수를 갖는다. 따라서 이를 Combination으로 만들어준다. 개인적으로 이 풀이에서 오래 걸렸고, 아직까지 재귀로 문제를 많이 풀어야겠다는 생각을 하게 해주었다.

이후에 문자열 키가 만들어지면 해당 키가 Object에 존재하는지 확인하고 존재한다면 해당 배열에 점수를 넣어주고, 없다면 해당 점수만을 갖는 배열을 값으로 만들어준다.

모든 info 배열에 대한 순회가 종료되었다면 모든 키들의 점수를 오름차순으로 정렬해준다. 문제의 조건에서 점수가 1이상 100,000이하의 자연수를 갖는다고 하였다. 그리고 이를 Filter를 통해 해당 점수 이상을 갖는 사원들의 길이를 구하려한다면 query의 길이도 1이상 100,000이하의 자연수이기 때문에 100억의 시간복잡도를 갖게 되기 때문이다. 따라서 이는 lowerBound를 통해 해당 점수를 처음 갖는 사람의 index를 찾는 이분 탐색을 실행해준다.

## Recap

카카오 문제는 구현하는 과정에서 고민을 더 많이 해야한다. 그리고 그 과정에서 평소에 확실하게 알지 못한 개념이나 잘 몰랐던 개념에 대해서 정립할 수 있는 기회를 주는 것 같아서 재밌는 것 같다.

부끄럽지만 이분 탐색의 LowerBound라는 개념에 대해서 알지 못했다. 따라서 이를 어떤 방식으로 구현하는지, 원리는 어떻게 적용되는지를 학습했고 관련된 백준 문제들을 풀어봤다. 지금처럼 꾸준히 부족한 부분을 하나씩 매워가면서 공부하면 좋을 것 같다.

## Code

```tsx
const log = console.log

function solution(info, query) {
  const map = {}
  for (let i = 0; i < info.length; i++) {
    const array = info[i].split(' ')
    const score = +array.pop()
    combination(0, '', array, score)
  }

  for (let i in map) map[i].sort((a, b) => a - b)

  const answer = []
  for (let i = 0; i < query.length; i++) {
    const arr = query[i].replace(/ and /g, ' ').split(' ')
    const score = Number(arr.pop())
    const key = arr.join('')
    const scoreArray = map[key]
    if (!scoreArray) answer.push(0)
    else {
      const result = lowerBound(scoreArray, score)
      answer.push(result)
    }
  }

  function combination(cnt, key, arr, score) {
    if (cnt === 4) {
      if (!map[key]) map[key] = [score]
      else map[key].push(score)
      return
    }
    combination(cnt + 1, key + arr[cnt], arr, score)
    combination(cnt + 1, key + '-', arr, score)
  }

  function lowerBound(arr, score) {
    let [start, end] = [0, arr.length]
    while (start < end) {
      const mid = Math.floor((start + end) / 2)
      if (arr[mid] < score) start = mid + 1
      else end = mid
    }
    return arr.length - start
  }

  return answer
}
```
