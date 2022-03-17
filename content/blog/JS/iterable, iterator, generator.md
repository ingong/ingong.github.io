---
title: 'iterable, iterator, generator'
date: 2022-02-28 16:21:13
category: 'JS'
draft: false
---

### TL;DR

JS 의 iterable, iterator, Generator 에 대해서 알아보자.

- iterable
  [Symbol.iterator]() 메서드를 가진 객체이며, iterator 를 return 한다
- iterator
  next 메서드를 가지고, {value, done} 객체를 return 한다.
- Generator
  Iterable 이면서 Iterator 인 객체이다. Generator 함수가 반환하는 객체이다.
- Generator 함수
  `*` 키워드를 통해 사용할 선언, 하나 이상의 yield 문을 포함한다. Generator 를 반환한다. 함수의 흐름을 제어할 수 있다는 측면에서 유의미하다.

## Iterable

`**iterable protocol**` 에 부합하는 순회할 수 있는 객체이다. 규칙은 다음과 같다.

```jsx
1. Symbol.iterator 메서드를 가지고 있고,이는 iterator 객체를 반환해야합니다.
2. 순회할 수 있는 데이터를 가지고 있어야한다.
3. iterator 객체는 반드시 next 라고 하는 메서드를 가져야합니다.
4. next 에는 #1 에 저장되어있는 데이터에 접근할 수 있어야합니다.
5. iterator 객체인 iteratorObj 에 대해서 iteratorObj.next() 를 하면
	 {value: <storedData>, done: false} #1 데이터가 추출되며,
	 전부 다 순회한 경우 {done: false} 가 반환되어야합니다.
```

`iterable protocol` 를 요약하면

- 어떤 객체가 반복되기 위해서는 iteration 동작에 대해 정의해야한다.
- `iterator` 는 next 함수를 구현해야하고, `{value: someValue, done: false}` 를 반환해야한다.

위 규칙을 모두 따르고 있으면 `iterable` 이라고 부르며, 반환된 객체를 `iterator` 라고 부른다. 이터러블 객체의 대표적인 예시로는 배열과 문자열이 있다.

for ... of 는 이터러블 객체의 경우에만 적용이 가능하다. 왜냐하면 for ... of 는 `Symbol.iterator(특수 내장 심볼)` 이라는 메서드가 있는 객체만 순회할 수 있기 때문이다.

## for - in 문과 for - of 문

- `for ... in 문`

객체들의 키를 조회하기 위해서 사용한다. Key 를 리턴한다. **상속된 열거 가능한 속성**들을 포함한 객객체에서 문자열로 키가 지정된 모든 열거가능한 속성에 대해 반복한다. key 를 리턴한다.

- `for ... of 문`

[Symbol.iterator] 속성을 가지는 Collection 만 대상으로 한다. Object 는 순회할 수 없다. value 를 리턴한다. Array, Map, Set, String, TypedArray, Arguments 등을 예시로 가진다.

- Collection

프로그래밍 언어가 제공하는 값을 담을 수 있는 컨테이너이다. JS 에는 다음과 같은 컬렉션들이 존재한다.

1. **Indexed Collection**

인덱스값에 의해 정렬이 되는 자료구조이다. 배열과 유사 배열 생성자인 Array 객체, TypedArray 객체가 있다.

2. **Keyed Collection**

키값을 기준으로 정렬되는 자료구조이다. Objects, Map, Set, Weak Map, Weak Set 가 있다.

### for ... of 의 내부 동작 방식을 알아보자.

1. for ... of 가 시작되면 for ... of 는 `Symbol.iterator` 를 호출한다. 이 때 `Symbol.iterator` 는 반드시 **이터레이터**를 반환해야한다. **이터레이터**는 next 메서드가 있는 객체이다.
2. 이후 for ... of 는 **반환된 객체(이터레이터)** 만을 대상으로 동작한다.
3. for ... of 에 다음 값이 필요하면, for ... of 는 이터레이터의 next 메서드를 호출한다.
4. next 메서드의 반환 값은 `{done: Boolean, value: any}` 와 같은 형태여야한다. `done=true` 이면 반복이 종료되었음을 나타내고, `done=false` 일 때는 value 에 다음 값이 저장된다.

다음 예시를 어떻게 하면 순회할 수 있을까? 먼저 실행시켜보면 다음과 같은 결과를 얻을 수 있다.

```jsx
let range = {
  from: 1,
  to: 5,
}

for (let v of range) console.log(v)
```

<div align="center">
<img src="../../assets/Iterable.png">
</div>

for ... of 는 이터러블 객체를 순회할 수 있는 방법이고, 이터러블 객체는 `Symbol.iterator` 메서드를 갖고 있는 객체다. 따라서 Symbol.iterator 메서드를 구현하면 된다. 해당 메서드를 구현할 때는 iterator 를 반환(next 메서드를 가지는 객체) 해야한다는 것에 집중해서 구현해보자.

```jsx
let range = {
  from: 1,
  to: 5,
}

range[Symbol.iterator] = function() {
  // iterator -> {value : asd, done: boolean}
  return {
    current: this.from,
    last: this.to,

    next() {
      if (this.current <= this.last)
        return { done: false, value: this.current++ }
      else return { done: true }
    },
  }
}

for (let num of range) console.log(num)
```

이렇게 작성하는 것도 좋지만 range 자체를 iterator 로 만드는 것이 더 좋아보인다. 한 번 수정해보자.

```jsx
let range = {
	from: 1,
	to: 5,

	[Symbol.iterator]() {
		this.current = this.from;
		return this;
	}

	next(){
		if (this.current <= this.next) return { done: false, this.current++ }
		else { done: true }
	}

}
```

range 는 iterable 이자 iterator 가 되었다. 이렇게 하나씩 `Symbol.iterator` 메서드를 구현하고, `next()` 메서드를 구현할 일이 많을 것 같지는 않다. 이를 위한 해결책인 `Generator` 가 있기 때문이다.

- 예제

  ```jsx
  function makeRangeIterator(start = 0, end = Infinity, step = 1) {
    let nextIndex = start
    let n = 0

    let rangeIterator = {
      next: function() {
        let result
        if (nextIndex < end) result = { value: nextIndex, done: false }
        else if (nextIndex === end) result = { value: n, done: true }
        else result = { done: true }
        nextIndex += step
        n++
        return result
      },
    }

    return rangeIterator
  }

  const it = makeRangeIterator(1, 4)

  let result = it.next()

  while (!result.done) {
    console.log(result.value)
    result = it.next()
  }
  ```

### Iterable Consumers

다음 문법들은 Iterable 을 사용한다.

1. `for-of loop`

```jsx
for (const value of someIterable) {
  // ...
}
```

1. `spread Operator`

```jsx
// Iterable의 모든  value들이 arr에 추가된다.
let arr = [firstElem, ...someIterable, lastElem]

// Iterable의 모든 value들이 arguments에 추가된다.
someFunction(firstArg, ...someIterable, lastArg)
```

이는 iterator 가 반환하는 객체 중 done 이 false 인 value 를 배열의 형태로 반환한다. 사실은 이미 알고 있는 내용이다.

1. `Positional Destructing`

```jsx
// Iterable의 첫 3개 값들이 저장된다.
let [a, b, c] = someIterable
```

## Generator

제네레이터는 iterable 프로토콜과 iterator 프로토콜을 따르는 객체이다. 즉, [Symbol.iterator]() 메서드를 가지며, next 메서드를 갖는 객체이다.

제네레이터 함수는 `*` 키워드를 통해 생성되며, 하나 이상의 yield 문을 포함한다. 일반 함수와 다르게 실행 과정에서 중지와 재시작이 가능하다. 제네레이터 객체가 가진 next 메서드를 호출하면 다시 실행될 수 있고, 제네레이터 함수의 yield 키워드를 만나면 중지된다.

간단하게 예시를 통해 Generator 함수를 선언하고, Generator 객체를 사용해보자 . 제네레이터 함수는 `function*` 키워드를 사용해서 선언할 수 있다. 다음은 피보나치 수열의 값을 반환하는 Generator 를 구현한 함수이다.

```jsx
function* fibonacci() {
  let prev = 0,
    curr = 1
  yield prev
  yield curr
  while (true) {
    let temp = prev
    ;[prev, curr] = [curr, temp + curr]
    yield curr
  }
}

for (const n of fibonacci()) {
  if (n > 30) break
  console.log(n)
}
```

- `yield`

제네레이터 함수의 실행을 일시적으로 정지시키는 키워드이며, `yield` 뒤에 오는 표현식이 반환된다.

- `next`

done 과 value 프로퍼티를 가진 객체를 반환한다. 또한 next 메서드를 호출할 때 매개변수를 제공하여 그 값을 generator 에게 전달할 수 있다.

```jsx
function* gen() {
  while (true) {
    var value = yield null
    console.log(value)
  }
}

var g = gen()
g.next(1)
// "{ value: null, done: false }"
// 처음에 전달하는 인자는 무시된다.
g.next(2)
// "{ value: null, done: false }"
// 2
```
