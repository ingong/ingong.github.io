---
title: '[PG Level3] 카카오 보석쇼핑 (Node.js)'
date: 2022-04-30 14:31:13
category: 'PS'
draft: false
---

## 문제

[코딩테스트 연습 - 보석 쇼핑](https://programmers.co.kr/learn/courses/30/lessons/67258?language=javascript)

### 문제 풀이

정확도와 효율성 모두를 통과시켜야하는 문제다. 투포인터와 쇼핑 기록 확인을 위한 객체를 활용하여 문제를 풀었다. 모든 종류의 보석을 적어도 하나 이상씩 포함하게 되도록 쇼핑을 해야하고 그 경우의 시작 index와 종료 index를 answer 배열에 넣어준다.

모든 보석 리스트를 한 번만 순회하면서 문제를 풀어야한다. 먼저 보석의 종류를 확인하기 위해 Set을 활용했다. 그리고 이를 활용해 보석의 종류별 개수를 0으로 초기화한 `gemDict` 를 선언해준다.

start와 end 그리고 kinds 변수를 선언해주었다. start는 조건을 만족하여 포인터를 이동시켜야하는 경우 활용하기 위한 변수이다. end는 조건을 만족시키지 못하여 우측으로 포인터를 더 이동시켜야하는 경우 활용해야하는 포인터다. kinds는 보석의 종류를 나타내는 변수다.

처음에는 kinds라는 변수를 선언하지 않고 gems에 대한 value를 순회해서 0보다 value를 filter해서 이를 보석의 카테고리와 비교하는 방식을 활용했고, 이는 효율성을 통과시키지 못하는 원인이 되었다. kinds라는 변수를 활용한 경우 보석을 제거했을 때는 그 개수가 0이 되는 경우에만 kinds를 1 감소시키고, 보석을 추가한 경우에는 그 개수가 1이 되는 경우만 kinds를 추가해주면 된다.

start와 end 조정이 끝난 후에 kinds가 보석의 종류와 같다면 `[start + 1, end]` 를 answer 배열에 push한다. index가 1부터 시작하기 때문에 start에 1을 더 해주고, end는 그 다음 추가할 보석에 대한 index를 담고 있기 때문에 굳이 1을 더 해줄 필요가 없다.

그리고 문제의 조건에서 여러 개의 답안이 존재하는 경우 1. 길이가 짧은 2. index 가 더 낮은 이라는 우선 순위를 갖는다. 이를 기준으로 answer 배열을 정렬해주고 첫 번째 요소가 반환되도록 return 문을 작성해준다.

### Code

```jsx
const log = console.log

function solution(gems) {
  const gemSet = new Set(gems)
  const gemSetSize = gemSet.size
  const gemLength = gems.length
  const gemDict = {}
  for (const gem of gemSet) gemDict[gem] = 0

  const answer = []
  let start = 0,
    end = 0,
    kinds = 0
  while (true) {
    if (kinds === gemSetSize) {
      const gem = gems[start]
      gemDict[gem]--
      if (gemDict[gem] === 0) kinds--
      start++
    } else if (end === gemLength) break
    else {
      const gem = gems[end]
      gemDict[gem]++
      if (gemDict[gem] === 1) kinds++
      end++
    }

    if (kinds === gemSetSize) answer.push([start + 1, end])
  }

  const [START, END] = [0, 1]
  answer.sort((a, b) => {
    const aLength = a[END] - a[START]
    const bLength = b[END] - b[START]
    if (aLength !== bLength) return aLength - bLength
    else if (aLength === bLength) return a[START] - b[START]
  })

  return answer[0]
}
```

### Recap

1시간10분이 소요됐다. 투 포인터의 start와 end가 의미하는 바를 헷갈려서 처음에 시간이 걸렸고, 효율성을 염두에 두고 객체를 사용했지만 객체의 장점인 search에 O(1)이 걸리는 장점을 잘 활용하지 못했다. 이 부분에서도 시간을 잡아먹었다.

최근 라이브 코딩을 준비하면서 자료구조와 알고리즘에 대한 강의를 수강하면서 공부를 했었는데 알고리즘 문제를 풀 때도 도움을 받는다는 느낌을 받았다. 자연스럽게 시간 복잡도를 계산하고 있는 내 자신을 보면서 만족했다. 카카오 Level3 문제여도 문제에서 요구하는 바를 명확하게 이해한다면 충분히 풀 수 있다는 것을 느낄 수 있어 좋았다.
