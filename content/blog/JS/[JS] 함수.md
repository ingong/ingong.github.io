---
title: '[JS] 함수(Function)'
date: 2022-03-17 00:15:13
category: 'JS'
draft: false
---

다양한 경험을 통해 학습했던 내용을 저만의 언어로 정리하는데 목적을 두고 있는 글입니다.

## 생성자의 메서드에 화살표 문법을 사용하여 얻는 이점

생성자 내부에서 화살표 함수를 메소드로 사용하는 주된 장점은 함수 생성시 `this` 의 값이 설정되고 그 이후에는 변경할 수 없다는 것이다. 이는 화살표 함수의 경우 this 가 evoke한 객체가 아닌 선언된 상위 스코프에 결정된다는 원리에 근거하고 있다. 즉, 화살표 함수는 컨텍스트 변경이 발생하지 않는다.

```jsx
const Person = function(firstName) {
  this.firstName = firstName
  this.sayName1 = function() {
    console.log(this.firstName)
  }
  this.sayName2 = () => {
    console.log(this.firstName)
  }
}

const john = new Person('John')
const dave = new Person('Dave')

john.sayName1() // John
john.sayName2() // John

// 일반 함수의 'this'값은 변경할 수 있지만, 화살표 함수는 변경할 수 없다.
john.sayName1.call(dave) // Dave (because "this" is now the dave object)
john.sayName2.call(dave) // John
```

## 고차 함수(high-order function)이란?

다른 함수를 매개 변수로 사용하여 데이터를 처리하거나 결과로 함수를 반환하는 함수이다. 고차 함수는 반복적으로 수행되는 어떤 연산을 추상화하기 위한 것이다. 예를 들어 `map` 은 고차 함수를 사용해 배열의 각 항목을 변환하고, 변환된 데이터로 새로운 배열을 반환한다. 이외에도 `forEach`, `filter`, `reduce` 가 있다.
