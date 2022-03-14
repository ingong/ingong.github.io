---
title: 'JS 로 우선순위 큐 구현하기'
date: 2022-3-14 15:38:13
category: '자료구조'
draft: false
---

JS 의 우선순위 큐(Priority Queue)를 직접 구현하기 위해서는 힙(Heap) 자료구조가 필요하다. BOJ 1781 컵라면 문제를 푸는 과정에서 우선순위 큐가 필요했고 이를 위해서 힙부터 구현하면서 JS 로 우선순위 큐를 구현해보려 한다!

# Heap

힙은 트리 기반의 자료구조이다. 규칙에 따라 크게 두 가지 힙으로 나눌 수 있다.

- `Max Heap` : 부모 노드는 항상 자식 노드보다 크거나 같아야 한다.
- `Min Heap` : 부모 노드는 항상 자식 노드보다 값이 작아야 한다.

일반적으로 Heap 자료구조는 이진트리로 구현한다. **이진트리**는 각각의 부모노드가 최대 2개 이하의 자식노드를 가지는 트리 자료구조이다. 자식노드는 각각 왼쪽 자식 노드와 오른쪽 자식 노드로 구성된다. 이 때 힙은 완전 이진 트리를 사용한다.

이진 트리와 관련해서 항상 헷갈렸던 이진 트리를 나타내는 용어 3가지에 대해서 간략하게 정리해보자. 살펴볼 내용은 포화 이진 트리, 완전 이진 트리, 정 이진 트리이다.

- 포화 이진 트리

모든 레벨이 꽉 찬 이진 트리를 의미한다.

- 완전 이진 트리

위에서 아래로, 왼쪽에서 오른쪽으로 순서대로 차곡차곡 채워진 이진 트리를 의미한다. 마지막 레벨을 제외한 나머지 노드는 꽉 차있어야한다.

- 정 이진 트리

모든 노드가 0개 혹은 2개의 자식 노드만을 갖는 이진 트리를 의미한다.

힙이 완전 이진 트리를 사용한다는 내용에 근거하여 최소 힙의 규칙을 도출해보면 다음과 같다.

1. 부모 노드는 항상 자식 노드보다 값이 작야아한다.
2. 한 레벨이 모두 채워져야 다음 레벨로 트리가 늘어날 수 있다.

JS 에서는 이를 배열로 구현할 수 있다.

```jsx
왼쪽 자식 노드 Index = 부모 노드 Index * 2 + 1
오른쪽 자식 노드 Index = 부모 노드 Index * 2 + 2
부모 노드 Index = (자식 노드 Index - 1) / 2
```

## 힙은 왜 언제 사용할까?

힙은 주로 최소 또는 최대 값을 O(1)의 시간 복잡도로 얻어내기 위해서 사용된다. 배열이나 연결리스트에서는 선형 탐색으로 인해 최소/최대 값을 얻기 위해서 O(n) 이 소요된다. 따라서 이진 탐색을 활용하면 O(logn) 까지도 시간 복잡도를 줄일 수 있다.

우선순위 큐를 구현하는데 사용할 수 있고, 다익스트라 알고리즘을 구현하는 과정에서 최소 비용을 기반으로 그래프를 탐색하는데 사용할 수 있다.

## 최소 힙 구현하기

삽입과 삭제 이후에 heap 이 최소 힙이 유지되도록 만들어주는 과정이 중요하다. 소스코드를 읽어보고 삽입 삭제 과정에 대해서 살펴보자.

```jsx
class Heap {
  constructor() {
    this.heap = []
  }

  getLeftChildIndex = parentIndex => parentIndex * 2 + 1
  getRightChildIndex = parentIndex => parentIndex * 2 + 2
  getParentIndex = childIndex => Math.floor((childIndex - 1) / 2)

  peek = () => this.heap[0]
  insert = (key, value) => {
    const node = { key, value }
    this.heap.push(node)
    this.heapifyUp()
  }

  heapifyUp = () => {
    let index = this.heap.length - 1
    const lastInsertedNode = this.heap[index]

    while (index > 0) {
      const parentIndex = this.getParentIndex(index)

      if (this.heap[parentIndex].key > lastInsertedNode.key) {
        this.heap[index] = this.heap[parentIndex]
        index = parentIndex
      } else break
    }

    this.heap[index] = lastInsertedNode
  }
}
```

삽입을 하는 경우를 살펴보자. 정렬의 기준이 되는 `key` 와 `value` 를 갖는 객체인 `node` 를 만든다. 그리고 `heap` 의 마지막 요소에 해당 `node` 를 넣어준다. 그리고 `heap` 이 다시 최소 힙을 만족시키는 `heapifyUp` 메서드를 실행시킨다. `heapifyUp` 에 대해서 살펴보자.

### `heapifyUp`

- `let index = this.heap.length - 1`

집어넣은 node 가 갖는 index 를 let 으로 index 로 선언하고 저장한다.

- `const lastInsertedNode = this.heap[index]`

바꿔줘야하는 node 에 대한 값을 해당 변수에 저장시켜놓는다. 해당 node 는 target index 가 발견되면 해당 index 에 대입시켜준다.

- `while 문`

본인의 부모 노드의 index 를 저장한다. 이 자료구조의 정렬 기준은 key 이고 삽입된 노드의 key 값보다 부모 노드의 key 가 크다면 변경시켜준다. 만약 크지않다면 자료구조 내에 규칙이 유지되고 있기 때문에 while 문을 종료시킨다. 이 과정은 한 레벨 위의 부모 노드에 한정되지 않고 해당 규칙을 만족시킬 때까지 계속 올라가면서 탐색한다.

- `this.heap[index] = lastInsertedNode`

종료될 당시 index 에는 변경되어야할 parentIndex 가 저장되어있기 때문에 기존에 lastInsertedNode 를 해당 index 에 대입한다.

## 삭제

이제 삭제하는 과정에 대해서 살펴보자. 힙에서 삭제를 한다면 힙의 최상위 노드가 제거될 것이며 삽입할 때와 동일하게 힙이 규칙을 만족하도록 변경시켜줘야한다. 제거하는 메서드를 살펴보면 최상위 노드를 제거하고 heap 의 마지막 index 에 들어있는 요소를 꺼내서 맨 앞의 index 에 대입시켜준다. 그리고 규칙이 지켜지지 않은 heap 을 다시 규칙에 맞게 만들어주는 heapifyDown 메서드를 실행시켜준다.

```jsx
class Heap {
	//...

	remove = () => {
	    const count = this.heap.length
	    const rootNode = this.heap[0]

	    if (count <= 0) return undefined
	    if (count === 1) this.heap = []
	    else {
	      this.heap[0] = this.heap.pop() // 끝에 있는 노드를 부모로 만들고
	      this.heapifyDown() // 다시 min heap 의 형태를 갖추도록 한다.
	    }

	    return rootNode
	 }

	 heapifyDown = () => {
	    let index = 0
	    const count = this.heap.length
	    const rootNode = this.heap[index]

	    while (this.getLeftChildIndex(index) < count) {
	      const leftChildIndex = this.getLeftChildIndex(index)
	      const rightChildIndex = this.getRightChildIndex(index)

	      const smallerChildIndex =
	        rightChildIndex < count && this.heap[rightChildIndex].key < this.heap[leftChildIndex].key
	          ? rightChildIndex
	          : leftChildIndex

	      if (this.heap[smallerChildIndex].key <= rootNode.key) {
	        this.heap[index] = this.heap[smallerChildIndex]
	        index = smallerChildIndex
	      } else break
	    }

	    this.heap[index] = rootNode
	  }
```

### `heapifyDown`

메서드의 이름에서 알 수 있듯이 위에서 내려오면서 탐색하면서 바꿔주는 과정이다. 부모노드를 기준으로는 최대 2개의 자식 노드를 가질 수 있다. 2개의 자식 노드를 가진다고 가정했을 때 이 노드들 중 더 작은 key 값을 갖는 노드와 위치를 바꿔주어야한다.

먼저 while 문의 종료조건은 부모의 왼쪽 자식노드의 index 가 count 보다 같거나 커지면 종료하게 된다. 트리를 그려보면 해당 조건은 더 이상 해당 부모 노드의 자식 노드가 존재하지 않음을 나타낸다. 이후의 코드를 살펴보면 왼쪽, 오른쪽 자식 노드의 key 를 비교해 더 작은 key 를 갖는 노드의 index 를 저장해준다.

그리고 해당 자식 노드의 key 가 부모 노드의 key 보다 작으면 두 노드의 위치를 변경시킨다.

# 우선순위 큐

우선순위 큐는 다음과 같이 `Heap` 을 상속받아 사용할 수 있다. `enqueue` 는 우선순위와 값을 입력받아 queue 에 삽입하는 메서드이며, `dequeue` 는 상위 요소를 제거하고 반환하는 메서드이다.

```jsx
class PriorityQueue extends Heap {
  constructor() {
    super()
  }

  enqueue = (priority, value) => this.insert(priority, value)
  dequeue = () => this.remove()
  isEmpty = () => this.heap.length <= 0
}
```

## References

- [[Medium] javascript로 heap-priority-queue 구현하기](https://jun-choi-4928.medium.com/javascript%EB%A1%9C-heap-priority-queue-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0-8bc13bf095d9)
