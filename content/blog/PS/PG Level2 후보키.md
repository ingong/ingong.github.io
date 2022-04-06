---
title: '[PG Level2] 후보키 (Node.js)'
date: 2022-04-06 17:56:13
category: 'PS'
draft: false
---

## 문제

[코딩테스트 연습 - 후보키](https://programmers.co.kr/learn/courses/30/lessons/42890)

### 문제 풀이

다음 조건을 만족하는 후보키의 개수를 산출하는 문제다.

- 관계 데이터베이스에서 릴레이션(Relation)의 튜플(Tuple)을 유일하게 식별할 수 있는 속성(Attribute) 또는 속성의 집합 중, 다음 두 성질을 만족하는 것을 후보 키(Candidate Key)라고 한다.
  - 유일성(uniqueness) : 릴레이션에 있는 모든 튜플에 대해 유일하게 식별되어야 한다.
  - 최소성(minimality) : 유일성을 가진 키를 구성하는 속성(Attribute) 중 하나라도 제외하는 경우 유일성이 깨지는 것을 의미한다. 즉, 릴레이션의 모든 튜플을 유일하게 식별하는 데 꼭 필요한 속성들로만 구성되어야 한다.

문제의 제한 조건 중 col의 길이가 8이하 이기때문에 조합으로 풀 수 있다. 문제의 풀이 흐름은 크게 3가지로 나누어 설명할 수 있다.

1. `getCombination` index를 key 결정해 조합 산출
2. `getUniqueKey` 해당 key가 유일한지 set 자료구조를 활용해
3. `getMinmalKey` 최소성을 보장하지 않는 키는 배열에서 제외

개인적으로 마지막 풀이가 가장 이해하기 어려웠던 것 같다. 해당 함수에 대한 내용을 살펴보자.

```jsx
function getMinimalKey(keyList) {
  const result = []

  while (keyList.length > 0) {
    result.push(keyList[0])
    keyList = keyList.reduce((acc, cur) => {
      // cur이 모두 key를 갖고 있다면 (포함관계가 존재)
      const isAllInclude = keyList[0].every(key => cur.includes(key))
      if (!isAllInclude) acc.push(cur)
      return acc //error
    }, [])

    // keyList = keyList.filter(key => {
    //     const isAllInclude = keyList[0].every(k => key.includes(k));
    //     if(!isAllInclude) return key;
    // })
  }

  return result
}
```

keyList의 첫 번째 요소를 result에 넣는다. 해당 요소는 더 이상 중복 요소를 비교할 수 없고, 이미 다른 중복 검사가 끝났기 때문에 유효한(최소성 보장) 키이다. `isAllInclude` 라는 변수를 살펴보자. 배열을 순회할 때 해당 요소(cur)이 keyList의 맨 앞 요소를 전부 포함한다면 `keyList[0] < cur (포함관계)` 를 만족하게 되고, 그렇다면 최소성을 보장할 수 없다. 따라서 이 경우에는 해당 요소를 배열에서 제외한다. 이 때 filter와 reduce를 모두 활용해도 좋다. 이번 풀이에서는 reduce를 활용해봤다. 해당 요소가 모두 포함관계가 아니라면 acc에 cur을 push해준다. 아니라면 기존에 존재하는 acc를 반환한다.

결과적으로 반환된 키 배열의 길이를 반환하도록 작성해주었다.

### Code

```jsx
function solution(relation) {
  const columLength = relation[0].length
  const keyList = Array.from({ length: columLength }, (_, i) => i)
  let candidates = []

  for (let i = 1; i <= columLength; i++) {
    const result = getCombination(keyList, i)
    candidates.push(...result)
  }

  candidates = getUniqueKey(relation, candidates)
  candidates = getMinimalKey(candidates)
  return candidates.length
}

function getMinimalKey(keyList) {
  const result = []

  while (keyList.length > 0) {
    result.push(keyList[0])
    keyList = keyList.reduce((acc, cur) => {
      // cur이 모두 key를 갖고 있다면 (포함관계가 존재)
      const isAllInclude = keyList[0].every(key => cur.includes(key))
      if (!isAllInclude) acc.push(cur)
      return acc //error
    }, [])
  }

  return result
}

function getUniqueKey(relation, keyList) {
  const result = []
  keyList.forEach(key => {
    const set = new Set()
    relation.forEach(rel => {
      set.add(
        rel.map((value, index) => (key.includes(index) ? value : '')).join('')
      )
    })
    if (set.size === relation.length) result.push(key)
  })
  return result
}

function getCombination(array, selectedNum) {
  const result = []
  if (selectedNum === 1) return array.map(v => [v])

  array.forEach((fixed, index, origin) => {
    const rest = origin.slice(index + 1)
    const combination = getCombination(rest, selectedNum - 1)
    combination.forEach(array => result.push([fixed, ...array]))
  })

  return result
}
```

[참고 풀이](https://velog.io/@euneun/프로그래머스-후보키-2019-kakao-blind-recruitment-javascript)

### Recap

이 문제를 계기로 조합, 순열, 중복조합, 중복순열에 대해서 확실하게 이해할 수 있었다. 또한 포함관계를 every로 구현하는 방법도 알게 되었고, reduce를 활용해서 포함관계가 성립하는 경우 해당 배열을 기존 배열에서 제거하는 방법 또한 알 수 있었다.
