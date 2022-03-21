---
title: 'Async-Await'
date: 2022-03-22 03:21:13
category: 'JS'
draft: false
---

Async-await 는 ECMAScript 2017에서 표준으로 정의된 키워드이다. 비동기 프로그래밍을 동기 방식처럼 직관적으로 표현할 수 있다는 장점이 있다. 비동기가 들어간 비지니스 로직은 중첩될수록 그 복잡도가 기하 급수적으로 늘어나고. 이를 간결하게 한다는 것은 유지보수와 생산성의 향상으로 귀결된다. 결과적으로 `promsie` , `generator` 가 ES2015 에 포함되었고, `async-await` 가 ES2017 로 포함되었다.

물론 최신 문법은 우리가 좀 더 편하게 개발할 수 있다는 장점도 있지만, 개발자는 다양한 환경에서 코드가 동작할 수 있도록 신경을 써야한다. 대부분이 경험했듯이 바벨이 우리의 수고를 덜어주는 역할을 해준다. 이는 브라우저 환경에 맞게 코드를 트랜스파일링하고, 필요한 경우에는 폴리필을 제공하여 정상 동작을 지원한다. 즉, 우리가 이번에 다룰 `async-await` 도 예외가 아니다. 그렇다면 이제 `async-await` 에 대해서 좀 더 알아보자. 이 구현체에는 `generator` 와 `promise` 에 대한 내용이 깊게 관여한다.

```jsx
// ES7
async function foo() {
  await bar()
}

function bar() {
  return new Promise(() => setTimeout(() => console.log('a'), 1000))
}

// ES5 compiled
let foo = (() => {
  var _ref = _asyncToGenerator(function*() {
    yield bar()
  })

  return function foo() {
    return _ref.apply(this, arguments)
  }
})()
```

`async` 키워드를 `generator` 로 바꾸고, `await` 키워드는 `yield` 로 바꾸었다. `generator` 는 `yield` 를 만날 때까지 실행되고, 해당 컨텍스트는 저장된 상태로 남아있게 된다. 따라서 비동기 로직이 종료되었을 때 적절하게 `next()` 를 호출해주면 `async-await` 함수가 완성된다.

그러나 `await` 키워드가 붙은 `bar()` 함수의 작업이 종료되는 시점은 `bar()` 함수밖에 모른다. 그렇다고 `next()` 를 `bar()` 함수 내에서 실행하게 되면 의존성이 생기게 된다. 그렇다면 어떻게 의존성을 분리할 수 있을까? 이 문제를 해결하기 위해 사용된 것이 promise 다. Babel은 promise와 재귀함수를 이용하여 `next()`를 대신 호출해주는 함수를 만드는데, 그게 바로 `_asyncToGenerator`이다. 다음 코드를 먼저 살펴보자.

```jsx
function _asyncToGenerator(fn) {
  return function() {
    var gen = fn.apply(this, arguments)
    return new Promise(function(resolve, reject) {
      function step(key, arg) {
        try {
          var info = gen[key](arg)
          var value = info.value
        } catch (error) {
          reject(error)
          return
        }
        if (info.done) resolve(value)
        else {
          return Promise.resolve(value).then(
            function(value) {
              step('next', value)
            },
            function(err) {
              step('throw', err)
            }
          )
        }
      }
      return step('next')
    })
  }
}
```

`fn.apply`를 실행하여 인자로 넘어온 generator를 실행하여 iterator 객체를 클로저로 저장해둔다. 나머지는 클로저에 저장한 iterator를 실행시키고, 반환된 promise객체를 재귀함수(여기서는 `step`)를 통해 반복해서 실행시켜 주는 것이다.

정리하자면 generator는 비동기적 패턴을 yield를 통해 동기적인 “모습"으로 바꾸어주고, promise는 generator로 만든 iterator를 반복해서 실행해주는 역할을 한다. await keyword에 사용하는 함수가 항상 promise를 반환해야하는 이유가 여기에 있다.
