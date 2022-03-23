---
title: 'React.js: JSX'
date: 2022-03-20 21:21:13
category: 'React'
draft: false
---

이 글은 [Build your own React](https://pomb.us/build-your-own-react/) 를 학습하고 정리한 글 입니다.

## JSX

```jsx
const element = <h1>Hello, world!</h1>
```

JSX는 JS확장 문법이다. JSX를 활용하면 JS파일에 마크업을 포함시킬 수 있다.
브라우저는 JS가 아닌 JSX를 해석할 수 없고, 빌드 전 Babel 과 같은 트랜스파일러를 활용하여 JS로 트랜스파일해줘야한다.

Babel을 사용하면 @babel/plugin-transform-react-jsx 플러그인을 활용하여 JSX를 두 가지 방식으로 트랜스파일 할 수 있는데, 첫 번째로 React.createElement(...) 를 호출하도록 변환하는 전통적인 방식(classic runtime)과 react/jsx-runtime 패키지를 import 하여 해당 패키지에서 헬퍼 함수를 가져와 호출하도록 변환하는 새로운 방식(automatic runtime)이 있다.

두 가지 트랜스파일 방식을 제공하는 이유는 react가 17버전 부터 새로운 JSX transform 을 도입했기 때문인데, 그 이유는 여기에서 소개된 문제들을 해결하기 위함으로 이는 여기서 다루고자 하는 내용의 범위를 벗어나기 때문에 다른 포스팅에서 다루도록 하곘다.

앞으로 다루게 될 내용에서 JSX는 Babel의 classic runtime에서와 같이 React.createElement(...)로 변환된다고 가정하고 포스팅을 작성하려고한다.

JSX 트랜스파일 과정에 대한 이해를 돕기 위해 예를 들어 다음과 같은 JSX가 있다고 생각해보자.

```jsx
const element = <h1 className="greeting">Hello, world!</h1>
```

```jsx
const element = /_#**PURE**_/ React.createElement('h1', { className: 'greeting' }, 'Hello, world!')
```

React.createElement 의 arguments 로 전달된 것들을 자세히보면, element 태그의 type, props, children 이 순서대로 전달된 것을 확인할 수 있다. 이 코드를 실행하면 React.createElement 함수는 다음과 같은 오브젝트를 반환한다.

```jsx
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!',
  },
}
```

함수 호출 결과로 반환된 오브젝트는 type, props 등의 키를 포함한다. type 은 DOM 노드의 타입을 명시하고 props 에는 JSX에 전달된 모든 key-value 가 포함되며 children 이란 특수한 키를 포함한다. children은 여러 elements를 포함한 배열이거나 string 이다. 나중에 Fiber 아키텍쳐를 살펴보면 알 수 있는데 children의 type이 무엇인지에 따라서 자식 요소에 대한 추가 탐색 여부를 결정한다.

## Step Zero: Review

지금부터 React가 어떻게 JSX를 활용하는지 알아보도록 하자.
먼저 아래 간단한 예시를 통해 React코드를 vanilla JS로 변환하면서 React, JSX, DOM element가 어떻게 동작하는지 살펴보자.

```jsx
const element = <h1 title="foo">Hello</h1>
const container = document.getElementById('root')
ReactDOM.render(element, container)
```

위 코드에 포함된 JSX는 Babel에 의해 트랜스파일되어 React.createElement(...) 로 표현되고, 최종적으로 다음과 같은 오브젝트로 변환된다.

```jsx
const element = {
  type: 'h1',
  props: {
    title: 'foo',
    children: 'Hello',
  },
}
```

다음으로는 `ReactDOM.render` 함수를 살펴보자. 이 함수는 argument로 주어진 element를 container에 마운트 시키는 기능을 한다. 이 함수를 간단한 vanilla JS코드로 바꾸면 다음과 같다.

```jsx
const element = { ... } // 생략
const container = document.getElementById('root')

const node = document.createElement(element.type)
node['title'] = element.props.title
const text = document.createTextNode('')
text['nodeValue'] = element.props.children

node.appendChild(text)
container.appendChild(node)
```

먼저 element의 type에 맞는 DOM 노드(node)를 생성하고, element의 props를 노드의 attribute로 적용한다. 그리고 element의 children에 해당되는 노드(text)를 생성하는데, 이 경우 "Hello" 라는 string이기 때문에 textNode를 생성하고 해당 노드의 nodeValue attribute로 전달한다. 최종적으로 text를 node에 append하고 다시 node를 container에 append하면 DOM tree가 완성된다.

Note: 용어를 보다 명확히 하기 위해 "element"는 React element를, "node"는 DOM element를 지칭한다고 가정한다.

이렇게해서 JSX가 DOM element로 만들어지기까지의 대략적인 과정을 살펴보았다. 지금까지는 JSX가 어떻게 해석되는지 이해하는데 초점을 맞추었다면, 지금부터는 React의 동작 원리에 초점을 맞춰서, React에서 제공하는 API들이 내부적으로 어떻게 돌아가는지 알아보도록 하자.

## Step I: The createElement Function

React의 createElement 함수는 type, props, children 을 arguments로 받아서 React element( type, props 키를 가진 오브젝트)을 return하는 함수라는 것을 알았으니, 이 함수를 직접 만들어보자. 먼저 다음과 같은 JSX를 가정하자.

```jsx
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
```

위 JSX는 다음과 같이 트랜스파일된다.

```jsx
const element = React.createElement(
  'div',
  { id: 'foo' },
  React.createElement('a', null, 'bar'),
  React.createElement('b')
)
```

div 태그의 자식으로 a, b태그가 있는데, children이 여러개인 경우 React.createElement 함수의 argument로 element의 type, props 그리고 children(배열)이 차례로 전달된다. 이 함수를 직접 작성해보면 다음과 같다.

```js
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map(child =>
        typeof child === 'object' ? child : createTextElement(child)
      ),
    },
  }
}
```

이때 children의 각 요소는 React elements 혹은 primitive value가 될 수 있는데, primitive value인 경우 특수한 type("TEXT_ELEMENT")을 갖는 wrapper element를 반환하도록 createTextElement 함수를 만들어주자.

```js
function createTextElement(text) {
  return {
    type: 'TEXT_ELEMENT',
    props: {
      nodeValue: text,
      children: [],
    },
  }
}
```

Note: 실제로 React는 primitive value를 wrapping 하거나 빈 children 배열을 만들지 않지만, 여기서는 코드를 단순하게 만들기 위해 이처럼 하였다.

이제 Babel이 JSX를 트랜스파일할 때 React.createElement 가 아닌 우리가 작성한 createElement 를 호출하도록 변경해주면 된다. 먼저 우리가 작성한 라이브러리를 Didact 라고 이름을 지어주고, 다음과 같이 JSX가 위치한 파일에 주석을 달아주자.

```jsx
;/\*_ @jsx Didact.createElement _/
const Didact = {
  createElement,
}
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
```

Babel을 사용하는 경우 JSX가 위치한 파일에 /\*_ @jsx ... _/ 주석을 달아주면 Babel이 해당 JSX를 트랜스파일할 때 React가 아닌 주석에 정의된 라이브러리를 사용한다.

## Step II: The render Function

이제 ReactDOM.render 함수를 직접 만들어보자. 지금은 DOM update, delete는 무시하고 일단 DOM 트리에 추가하는 기능만 생각하자. 먼저 render 함수의 윤곽을 잡아보자.

```jsx
function render(element, container) {
  // DOM 노드를 element type에 맞게 생성한다
  const dom = document.createElement(element.type)
  // children DOM 노드를 생성한다
  element.props.children.forEach(child => render(child, dom))
  // element를 컨테이너에 append 한다
  container.appendChild(dom)
}

const Didact = {
  createElement,
  render,
}
```

먼저 element의 type에 맞는 DOM 노드를 생성하고, 이 노드를 container에 추가해야한다. 그리고 element의 children에 대해서 이를 반복해야 한다. 이때 element의 type이 "TEXT_ELEMENT" 인 경우를 핸들링 해준다.

```jsx
const dom =
  element.type == 'TEXT_ELEMENT'
    ? document.createTextNode('')
    : document.createElement(element.type)
```

마지막으로 element의 props를 DOM 노드의 attribute로 전달하기만 하면 된다.

```jsx
function render(element, container) {
  const dom =
    element.type == 'TEXT_ELEMENT'
      ? document.createTextNode('')
      : document.createElement(element.type)
  // children을 제외한 props를 DOM attribute로 전달한다
  const isProperty = key => key !== 'children'
  Object.keys(element.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = element.props[name]
    })

  element.props.children.forEach(child => render(child, dom))
  container.appendChild(dom)
}
```

이렇게 하면 이제 JSX를 React가 아닌 우리가 작성한 라이브러리로 렌더링할 수 있게 된다.

```jsx
const container = document.getElementById('root')
Didact.render(element, container)
```

## Recap

이번 포스팅에서는 JSX가 DOM Element로 해석되는 방식과 React가 이를 DOM tree로 만드는 과정에 대해 살펴보았다. 살펴본 내용에 대해서 정리해보자.

- JSX는 Babel과 같은 트랜스파일링 도구를 브라우저가 해석할 수 있는 JS로 트랜스파일되고, React API 호출을 통해 변환된다.
- `React.createElement()` 함수는 type, props, children을 인자로 받아 type과 props를 키로 갖는 React element를 반환한다.
- ReactDOM의 render 함수는 element와 cntainer를 인자로받아 해당 container에 해당 돔 노드를 추가한다.

## REFERENCE

- [Build your own React](https://pomb.us/build-your-own-react/)
- [React 공식문서: JSX](https://ko.reactjs.org/docs/introducing-jsx.html)
- [Velog: JSX](https://velog.io/@sdadssadsadasd/JSX-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC)
