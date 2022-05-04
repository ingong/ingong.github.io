---
title: '[PG] 카카오 표 편집 (Node.js)'
date: 2022-05-02 16:21:13
category: 'PS'
draft: fals집
---

## 문제

[코딩테스트 연습 - 표 편집](https://programmers.co.kr/learn/courses/30/lessons/81303)

### 문제 풀이

**<풀이 방법>**

더블 링크드 리스트와 스택을 활용해 풀 수 있는 문제다. Node 클래스를 만들고, 해당 클래스가 값과 prev, next 프로퍼티를 참조하도록 선언한다.

1. 더블 링크드 리스트를 생성한다.
2. curNode는 현재 가리키고 있는 Node를 의미한다.
3. cmd에 대한 반복문을 처리한다.

3-1. 'U': count 만큼 이동하며 curNode를 prev로 바꿔준다.

3-2. 'D': count 만큼 이동하며 curNode를 next로 바꿔준다.

3-3. 'C': curNode를 stack에 넣고, prev와 next를 상호 연결시켜주고, curNode를 prev 또는 next로 변경시켜준다.

3-4. 'Z': 가장 마지막으로 제거한 요소를 stack으로부터 pop하고, pop한 노드의 prev와 next를 다시 자신에 맞게 연결한다.

1. answer을 모두 ‘O’값을 갖도록 선언한다.
2. stack을 순회하면서 stack에 존재한다면 제거된 노드를 의미하기 때문에 answer의 해당 index의 값을 ‘X’로 변경시켜준다.
3. answer를 리턴한다.

### Code

```jsx
const log = console.log

class Node {
  constructor(value) {
    this.value = value
    this.prev = null
    this.next = null
  }
}

function solution(n, k, cmd) {
  const stack = []
  const start = new Node(0)
  let curNode = start
  for (let i = 1; i < n; i++) {
    const node = new Node(i)
    curNode.next = node
    node.prev = curNode
    curNode = node
  }
  curNode = start

  // curNode를 행번호가 k인 노드로 변경
  let count = 0
  while (count < k) {
    curNode = curNode.next
    count++
  }

  for (const element of cmd) {
    let [command, step] = element.split(' ')
    step = Number(step)
    if (command === 'U') {
      let count = 0
      while (count < step) {
        curNode = curNode.prev
        count++
      }
    } else if (command === 'D') {
      let count = 0
      while (count < step) {
        curNode = curNode.next
        count++
      }
    } else if (command === 'C') {
      // stack
      stack.push(curNode)

      // 이전 노드와 다음 노드에 대한 설정
      const prevNode = curNode.prev
      const nextNode = curNode.next
      if (prevNode) prevNode.next = nextNode
      if (nextNode) nextNode.prev = prevNode

      // 현재 노드 설정
      if (nextNode) curNode = nextNode
      else curNode = prevNode
    } else if (command === 'Z') {
      const recoveredNode = stack.pop()
      const prevNode = recoveredNode.prev
      const nextNode = recoveredNode.next
      if (prevNode) prevNode.next = recoveredNode
      if (nextNode) nextNode.prev = recoveredNode
    }
  }

  const answer = new Array(n).fill('O')
  for (const node of stack) {
    const value = node.value
    answer[value] = 'X'
  }

  return answer.join('')
}
```

### Recap

처음 문제를 풀 때는 설마 연결리스트로 풀겠어? 라고 생각하면서 배열로 풀었는데 시간 초과때문에 통과할 수 없었다. 최근에 연결 리스트를 직접 구현해본적이 있어서 접근 방법을 알고 나서는 어렵지 않게 풀 수 있었다.
