---
title: 'Recoil Migration'
date: 2021-12-05 16:21:13
category: 'React'
draft: false
---

### TL;DR

- Context API 와 Reducer 를 활용한 상태관리에서 느꼈던 불편함을 정리했다.
- 마이그레이션은 일종의 리팩토링이다. 같은 동작을 보장하기 위해 우리는 테스트 코드를 작성을 시도했지만 실패했다.
- 기능 명세를 기반으로 Recoil 로의 마이그레이션을 성공하고, 상태 관리 라이브러리를 왜 쓰는지를 체감했다.

부스트 캠프 팀프로젝트인 HyupUp 에서 우리는 상태 관리에 대해 고민했다. 첫 번째 고민은 상태 관리 대상과 구조였으며, 이를 해결한 후 상태 관리를 어떻게 할지 고민을 시작했다. 우리는 이전 JavaScript 의 뜨거운 맛을 보고, React 의 달콤함을 경험했던 것처럼 상태관리에서도 경험해보고 싶었다. 우리의 상태 관리 방법에 대한 도전기를 간단하게 정리해보고, 느낀 점을 정리해보려한다.

<br>

## Context API 와 Reducer 로 상태 관리하기

이전 글에서 우리가 전역적으로 관리했던 상태에 대해서 다뤘지만, 아직 읽지 않은 분들이 계실 수 있어서 간략하게 요약하면 다음과 같다. User 가 존재하며, 하나의 조직에 속할 수 있으며, 여러개의 프로젝트에 참여할 수 있다. 하나의 프로젝트에는 여러 개의 에픽이 존재하며, 하나의 에픽에는 여러 개의 스토리가, 하나의 스토리에는 여러 개의 테스크가 존재한다. 다음 그림을 보면 충분히 이해가 될 것이라고 생각한다.

<br>

<div align="center">
<img src="../../assets/HyupUp.png">
</div>

<br>

우리는 치열한 의사소통 끝에 User, 에픽, 스토리를 전역적으로 관리하기로 했다. 우리는 개발 3주차까지 상태 관리 라이브러리를 사용하지 않았기 때문에 context 객체를 만들고, 컴포넌트가 이 context 를 가지도록 해당 컴포넌트 상위에 `provider` 로 부턱 context 를 정의한 변수를 감쌌다. User 에 대한 상태가 비교적 간단해 UserContext 코드를 가져왔다.

```tsx
export const UserContext = createContext<ContextType>({
  userState: null,
  dispatch: null,
})

export const UserProvider = ({ children }: { children: React.ReactNode }) => {
  const [userState, dispatch] = useReducer(reducer, user)
  return (
    <UserContext.Provider value={{ userState, dispatch }}>
      {children}
    </UserContext.Provider>
  )
}
```

상태에 변화를 주는 방법은 Reducer 로 정의했다. reducer 에 Action 을 정의하고, 해당 Action 에 따라서 코드가 실행되도록 했다. User 에 대한 reducer 코드는 50줄이였기 때문에 얼마나 길었는지 보여주기 위해 UserAction 에 대한 Type 정의 코드를 가져왔다.

```tsx
type UserAction =
  | { type: 'GET_USER'; payload: UserState }
  | { type: 'LOGOUT' }
  | { type: 'UPDATE_USER'; payload: UserState }
  | { type: 'ADD_PRIVATE_TASK'; payload: PrivateTask }
  | { type: 'DELETE_PRIVATE_TASK'; payload: number }
  | { type: 'UPDATE_PRIVATE_TASK'; payload: PrivateTask }
  | { type: 'UPDATE_PROJECT_TASK'; payload: PrivateTask }
```

우리는 상태를 정의하고, 변경을 정의하는 과정에서 보일러 플레이트 코드가 정말 많다는 느낌을 받았다. 새로운 상태변화 로직이 필요할 때마다 액션을 새로 정의하거나, 기존 코드를 수정해야했다. 이 시점에 Recoil 을 사용해보지는 않았지만, 하나씩 정의할 때보다는 코드를 줄여줄 것으로 내심 기대했다.

코드 차원에서의 복잡도도 문제였지만, 상태 자체의 복잡도도 존재했다. 다음은 우리가 정의한 `UserState` 이다.

```tsx
UserState = {
  id?: number;
  name?: string;
  job?: string;
  email?: string;
  imageURL?: string;
  admin?: boolean;
  organization?: number;
  currentProjectName?: string;
  currentProjectId?: number;
  projects?: Array<ProjectType>;
  privateTasks?: Array<PrivateTask>;
  projectTasks?: Array<ProjectTask>;
};
```

`UserState`에는 개인 할 일(privateTasks)배열과 프로젝트에서의 할 일(projectTasks)배열이 있다. 이 둘을 불변성을 지키면서 변경시키는 것은 꽤나 복잡했다. 게다가 이 둘을 사용하는 `ListView` 컴포넌트의 경우 두 프로퍼티 외 다른 프로퍼티들을 참조하지도 않았다. 즉, 컴포넌트에서 필요한 상태 외 다른 상태를 가져와야만 했고, 객체에 프로퍼티에 접근하고 이를 변경시켜야만 했다. 그렇다고 위 둘을 분리하기에는 새로운 context를 만드는 것이 부담이기도 했고, 다른 프로퍼티와 연관이 아예 없는 것도 아니었다.

심지어 `EpicState`, `StoryState` 의 상태 관리의 복잡도는 더 컸다. 두 가지 상태 모두 사용자의 인터렉션에 의해 빈번하게 상태가 변화했기 때문이다. 따라서 우리는 4주차의 시작 시점에 마이그레이션을 적용할 시점이라고 판단하고, 마이그레이션을 시작했다. 마이그레이션을 통해서 우리는 기존에 정상 동작을 보장하고, 코드만 수정해야했고 이를 어떻게 보장할지 고민했다.

## 정상 동작을 보장하기 위한 노력

필자는 꼭 테스트로 이 과정을 보장하고 싶었다. 테스트 코드에 대한 경험이 없었고, 이 경험이 없었기 때문에 더욱 시도해보고 싶었다. 우리가 기획 단계에서 작성한 백로그에서는 프론트 엔드 테스트 코드 작성에 대한 내용이 없었다. 당연하게 주말을 할애해서 테스트 환경을 구축해야했고, 워낙 열정이 넘칠 때라 자신감있게 시도했다. 칸반 보드 내에서 스토리를 담당했어서, 해당 기능의 정상 동작을 보장하기 위해 먼저 테스트 환경을 구축했다.

생각보다 환경을 구축하는 과정부터 어려웠다. React 훅을 테스팅하려면 컴포넌트의 환경과 동일하게 구축해야했고, 우리는 Context API 를 사용했기 때문에 이에 대한 환경 구축이 필요했다. 다음은 CustomRender 를 작성한 코드이다.

```tsx
import React, { ComponentType, ReactElement } from 'react'
import { render } from '@testing-library/react'
import theme from 'client/src/styles/theme'
import { ThemeProvider } from 'styled-components'
import { StoryProvider } from 'client/src/contexts/storyContext'
import { EpicProvider } from 'client/src/contexts/epicContext'

const CustomProvider = ({ children }: { children: React.FC }) => {
  return (
    <ThemeProvider theme={theme}>
      <EpicProvider>
        <StoryProvider>{children}</StoryProvider>
      </EpicProvider>
    </ThemeProvider>
  )
}

const CustomRender = (ui: ReactElement, options?: any) => {
  return render(ui, { wrapper: CustomProvider as ComponentType, ...options })
}

export default CustomRender
```

이번 글의 목적이 테스팅은 아니기 때문에 이 코드에 대한 설명은 생략하겠다. 뿐만 아니라 svgTransform 코드도 작성해야했고 이외 여러가지 외부 모듈을 설치하면서 이슈들을 하나씩 해결해나갔다. 정말 테스트를 제대로 작성하지 못하다면 슬플 것 같았다.

결론부터 말하자면 스토리 아이템 컴포넌트와 스토리 추가 컴포넌트에 대해서는 테스트 작성을 성공했다. 기능 명세를 토대로 테스트를 작성했고, 통과됐다. 사실 이미 구현한 기능에 대해서 테스트를 작성하면서 굳이 왜 해야하는지 의구심이 들었지만 일단 시도했고, 성공은 했다.

이제는 리코일에 대해서 테스팅만 하면 됐다. 원래 목표는 주말 내에 완성이였지만, 이 테스트를 도입하고 나니 시간은 일요일 저녁이였다. Recoil 테스트에 대해서 찾아보고 자야지 생각하고 구글링을 시도했다. 구글링에서 결과가 많이 나오지 않아서 당황했지만, 내일의 내 자신이 해결할 것이라고 생각하고 일단 구글링을 멈췄다.

월요일, 화요일, 수요일 3일동안 맡은 기능 개발을 마무리한 후에는, 리코일 테스트를 시도했다. 결과는 실패였다. 나는 Unit Test 를 시도했지만, 이미 Recoil 로 코드를 작성한 이후부터 어려운 것 같았다. 그리고 내가 테스트를 작성한 목적에 대해서 다시 생각해봤다. 나는 정상동작을 보장하기 위해 테스트를 작성했지만, Recoil 로 마이그레이션을 하기 위해 또 별도의 테스트 코드를 작성해야했다.

개발 리소스를 고려했을 때, 과연 이게 팀의 서비스 완성도를 위해 올바른 선택인지 고민하는 순간이였다. 팀원들에게 의견을 전달하고, 테스트 코드 작성을 유보했다. 6주 기간 동안에 도입은 힘들다고 생각했고, 우리는 백로그에 작성한 사용자 동작을 기반으로 정상 동작 보장을 하기로 했다. 한 줄이면 끝날 수 있었던 정상 동작 보장을 위한 노력이 결과물이 없어 아쉬웠다. 하지만 목표를 세우고 몰입했던 경험이 나쁘지 않았다는 생각도 들었고, 나중에는 꼭 해결해보고 싶다는 다짐을 했다.

## Recoil 로 마이그레이션하기

다른 팀원이였던 해수님이 Recoil 로 마이그레이션을 성공하고, 관련한 내용을 공유해주셨다. 말씀해주신 내용과 Recoil 공식 문서를 토대로 코드를 작성했고, 생각보다 빠르게 적용할 수 있었다. 워낙 훅과 사용방법이 유사해서 이틀 정도를 투자해 마이그레이션을 할 수 있었다.

```tsx
// recoil/Story/atom.ts
import { atom } from 'recoil'
import { StoryListType } from '@/types/story'

const storyListAtom = atom<StoryListType>({
  key: 'storyListAtom',
  default: [],
})

export default storyListAtom
```

`atom`이라는 키워드로 상태를 담고 컴포넌트 최상단을 `RecoilRoot` 로 감싸기만 하면 끝이였다. 복잡한 정의도 필요 없어서 기존의 **Context API를 사용하던 것에 비해 훨씬 간단**해졌다. 상태의 변경 또한 매우 간단했다.

```tsx
import { useRecoilState, useRecoilValue } from 'recoil';
import storyListAtom from '@/recoil/story/atom';
import userAtom from '@/recoil/user';


const KanbanColumn = ({
  category,
}: KanbanType) => {
 const [storyList, setStoryList] = useRecoilState(storyListAtom);
  // KanbanColum Code
```

스토리 배열에서 key 에 해당하는 아이템만 가져오기 위해서는 Selector 를 사용해야했으며, 다음과 같이 정의하면서 사용할 수 있었다.

```tsx
// recoil/Story/selector.ts

import { DefaultValue, selectorFamily } from 'recoil'
import storyListAtom from './atom'
// import

export const storyState = selectorFamily<StoryType | undefined, number>({
  key: 'storyWithIdSelector',
  get: id => ({ get }) => {
    const storyList = get(storyListAtom)
    return storyList?.find(story => story.id === id)
  },
  set: (id: number) => ({ set, get }, newValue) => {
    set(
      storyListAtom,
      newValue instanceof DefaultValue
        ? get(storyListAtom)
        : prev =>
            produce(prev, draft => {
              const newStoryIdx = draft?.findIndex(story => story.id === id)
              if (newStoryIdx === -1 || !newValue) return
              draft[newStoryIdx] = newValue
            })
    )
  },
})
```

위처럼 id 를 인자로 받아와서 가져오도록 작성한다면 끝이였다. 이를 통해 하나의 스토리 아이템을 가져와야하는 컴포넌트에서 굳이 배열 전체 상태를 가져올 필요가 없어졌다.

```tsx
// client/src/components/KanbanColumn/KanbanItemInput
const updateStoryName = useSetRecoilState(storyState(key))
```

다음과 같이 코드를 작성해 원하는 스토리를 가져올 수 있었다.

## 마무리

우려와 달리 우리는 리코일을 생각보다 어렵지 않게 도입할 수 있었다. Recoil 을 사용하면서 체감했던 장점을 정리해보면 다음과 같다. 먼저 러닝커브가 낮았다. Redux 를 도입을 고려했지만 최종적으로 Recoil 을 도입한 가장 중요한 계기였다. 두 번째로 코드량이 눈에 띄게 감소했다. `atom` 함수 하나로 간단하게 전역상태를 만들 수 있었고, 이는 기존의 `context` 에 비해 훨씬 코드가 간결했다. 마지막으로 원하는 상태만을 컴포넌트에서 가져와 사용할 수 있었다. `selector` 키워드로 상태를 분리해 관리할 수 있게 되었고, 필요한 부분만 조회하고 수정하는게 가능해졌다.

짧은 기간 동안 Recoil 을 사용하면서 아쉬웠던 점도 역시 있었다. 먼저 레퍼런스가 부족했다. 대부분의 블로그들은 리코일의 공식 문서를 학습한 내용들이였고, 스택오버플로우에 답변도 Redux 에 비하면 현저하게 적었다. 두 번째는 테스트 코드 작성의 어려움이다. 레퍼런스가 없다보니 당연히 테스트 코드에 대한 레퍼런스도 부족했다. 마지막은 캐싱 지원 부분이다. 개발한 HyupUp 서비스는 실시간으로 상태가 변화하고 동기화되어야했고, 오히려 이 캐싱 기능은 방해가 되었다. 현재 unstable 단계인 `useRecoilRefresher` 을 사용하여 해결할 수 있지만, unStable 단계여서 get 내부에서 비동기 로직을 작성하지 않았다.
