https://ko.react.dev/blog/2024/04/25/react-19-upgrade-guide

## 설치

JSX Transform - 이제 필수입니다

- 2020년에 번들 사이즈 개선 + JSX를 import 하지 않고도 사용하기 위해서 JSX Transform이 추가되었습니다.
- React 19에서는, ref를 prop으로 쓰기, JSX 속도 개선과 같은 추가 개선사항이 JSX Transform을 필요로 합니다. 따라서, JSX Transform 사용이 이제 필수가 됩니다.

## Codemods

Codemod와 협업을 통해서 CLI 딸깍으로 마이그레이션을 할 수 있습니다.

```bash
npx codemod@latest react/19/migration-recipe
```

위 명령어를 수행하면 5가지 codemod가 모두 실행됩니다. 하지만 이는 타입스크립트 변경사항을 포함하지 않습니다.

## Breaking Changes

### 렌더 과정에서 에러가 re-throw 되지 않음

이전 리액트 버전에서는, 렌더링 과정에서 던져진 에러는 잡혀서 다시 던져졌습니다. 개발 환경에서는 `console.error`도 찍어서 중복 에러 로그가 발생했습니다.

React 19에서는, 다시 던지는 과정을 없앰으로써 에러가 핸들링 되는 방식을 개선했습니다.

- Uncaught Erros
    - 에러 바운더리가 잡지 못한 에러는 `window.reportError`를 통해 리포트 됩니다.
- Caught Errors
    - 에러 바운더리가 잡은 에러는 `console.erro`를 통해 리포트 됩니다.

이 변화는 대부분 앱에 영향을 미치지 않겠지만, 만약 당신의 에러 리포팅이 re-throw된 에러에 의존하고 있다면 에러 핸들링 로직을 업데이트해야합니다. 이를 지원하기 위해 `createRoot`와 `hydrateRoot`에 새로운 메서드들을 추가했습니다.

```tsx
const root = createRoot(container, {
  onUncaughtError: (error, errorInfo) => {
    // ... log error report
  },
  onCaughtError: (error, errorInfo) => {
    // ... log error report
  }
});
```

### Deprecate된 React API 제거

- 함수형 컴포넌트 - `propTypes`, `defaultProps` 제거
- `contextTypes` & `getChildContext`를 사용하는 레거시 Context
- 문자열 `ref`
- 모듈 패턴 팩토리
- `React.createFactory`
- `react-test-renderer/shallow`

### Deprecate된 React DOM API 제거

- `react-dom/test-utils`
    - `act`가 `react` 패키지로 옮겨짐, import 바꾸면 됨
    - 나머지 `test-utils` 함수들은 제거됨
- `ReactDOM.render`
    - `ReactDOM.createRoot`로 대체됨
- `ReactDOM.hydrate`
    - `ReactDOM.hydrateRoot`로 대체됨
- `unmountComponentAtNode`
    - `root.unmount`로 대체됨
- `ReactDOM.findDOMNode`
    - DOM ref로 대체됨

### 새로운 Deprecations

- `element.ref`
    - 이제 `ref`가 `props`로 제공되기 때문에, `element.ref`는 `element.props.ref`로 대체되어야 합니다.
- `react-test-renderer`
    - 사용자 환경과 다른 렌더러 환경을 생성하고, React 내부코드에 의존하기 때문에 deprecate 됨
    - 대신 `@testing-library/react` 또는 `@testing-library/react-native`를 사용하는 것을 추천

## 주요 변화

### StrictMode 변경사항

React 19에는 Strict Mode에 대한 몇가지 수정사항과 개선사항이 포함됩니다.

개발환경에서 Strict Mode로 인해 렌더링이 두번 발생하면, `useMemo`와 `useCallback`은 첫번째 렌더에서 계산된 값을 그대로 사용합니다.

모든 Strict Mode 동작은 **개발과정에서 컴포넌트에 대한 버그를 빠르게 찾아낼 수 있게 하기 위함**입니다.

### Suspense 개선사항

React 19에서, 컴포넌트가 suspend 되면 React가 즉시 가장 가까운 Suspense 경계의 fallback을 다른 형제 트리가 렌더링 될 때까지 기다리지 않고 커밋합니다. fallback이 커밋된 이후, React가 형제 트리의 pre-warm을 스케쥴링합니다.

![image.png](attachment:da8cc9ba-0ebe-4009-a079-7706e5f1aad0:image.png)

이 개선으로 인해 Suspense의 fallback이 더 빨리 보이게 되는 반면, 여전히 lazy하게 suspend된 나머지 트리를 pre-warm하게 됩니다.

### UMD 빌드 제거

UMD는 과거에 React를 빌드 스텝없이 로딩할 수 있는 편한 방법이였습닌다. 현재는 HTML 문서 내에서 모듈을 스크립트로 로드할 수 있는 다른 방법이 있습니다. React 19부터는, 더 이상 UMD 빌드를 생성하지 않습니다.

React 19를 script 태그로 로딩하려면, ESM 기반의 CDN을 사용하는 것을 추천합니다.

### React Internals에 의존하는 라이브러리의 업그레이드가 어려울 수 있음

사용하지 말라고 써져있는 `SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED` 등의 internals를 사용한 경우, React 19에서 이러한 internals가 업데이트 되었기에 업그레이드에 어려움이 있을 수 있습니다.

이러한 변경사항은 Breaking Change로 취급되지 않으며, 가이드라인을 잘 따라줬다면 업그레이드에 문제가 없을겁니다.

이러한 Internal 사용을 더 막기 위해, `SECRET_INTERNALS` suffix를 `_DO_NOT_USE_OR_WARN_USERS_THEY_CANNOT_UPGRADE`로 대체합니다.

## Typescript 변경사항

### deprecate된 타입스크립트 타입 제거

React 19에서 제거된 타입들을 제거했습니다. 

### `ref`의 클린업이 필수가 됨

`ref` cleanup function이 추가됨에 따라, `ref` 콜백에서 클린업 함수가 아닌 것을 반환하게 되면 타입스크립트가 오류를 뱉게 됩니다. 

```tsx
- <div ref={current => (instance = current)} />
+ <div ref={current => {instance = current}} />
```

### `useRef`의 인자가 필수가 됨

`useRef`에 대한 아주 흔했던 불만으로, 이제 `useRef`의 타입이 변경되어서 반드시 하나의 인자를 받게 디ㅗ었습니다. 이제 더 context처럼 동작하게 됩니다.

```tsx
// @ts-expect-error: Expected 1 argument but saw none
useRef();
// Passes
useRef(undefined);
// @ts-expect-error: Expected 1 argument but saw none
createContext();
// Passes
createContext(undefined);
```

이는 곧 모든 ref가 mutable 해짐을 의미합니다. 따라서 더이상 `null`로 초기화된 `ref`의 값을 재할당할 수 있게됨을 의미합니다.

```tsx
const ref = useRef<number>(null);

// Cannot assign to 'current' because it is a read-only property
ref.current = 1;
```

`MutableRef`는 이제 deprecate되고, `useRef`는 `RefObject`만을 반환합니다.

```tsx
interface RefObject<T> {
  current: T
}

declare function useRef<T>: RefObject<T>
```

### `ReactElement` 타입에 대한 변경사항

리액트 요소의 `props`는 이제 기본적으로 `unknown` 타입이 됩니다(이전에는 `any`)

### 타입스크립트에서의 JSX namespace

전역 `JSX` namespace가 제거되고, `React.JSX` 형태로 들어가게 됩니다. 이를 통해서 `JSX`를 사용하는 다른 라이브러리와의 충돌을 피할 수 있습니다. 

이제 `JSX` namespace에 대한 module augmentation을 `declare module` 구문으로 감싸줘야합니다.

```tsx
// global.d.ts
+ declare module "react" {
    namespace JSX {
      interface IntrinsicElements {
        "my-element": {
          myElementProps: string;
        };
      }
    }
+ }
```

정확한 module specifier는 `tsconfig.json` → `compilerOptions` 에서 지정한 값에 따라 다릅니다.

- `jsx: react-jsx` → `react/jsx-runtime`
- `jsx: react-jsxdev` → `react/jsx-dev-runtime`
- `jsx: react` & `jsx: preserve` → `react`

## 더 나은 `useReducer` 타입

새로운 best practice는 `useReducer`에게 아무런 타입도 넘기지 않는 것입니다.

```tsx
- useReducer<React.Reducer<State, Action>>(reducer)
+ useReducer(reducer)
```

하지만 `Action`을 튜플 형태로 넘기는 엣지 케이스의 경우에는 워킹하지 않을 수 있습니다.

```tsx
- useReducer<React.Reducer<State, Action>>(reducer)
+ useReducer<State, [Action]>(reducer)
```

리듀서를 인라인으로 정의하는 경우, 함수의 파라미터에 대한 타입을 지정하는 것을 추천합니다.

```tsx
- useReducer<React.Reducer<State, Action>>((state, action) => state)
+ useReducer((state: State, action: Action) => state)
```

reducer를 `useReducer` 바깥으로 빼려면 아래와 같이 타입을 정의하면 됩니다.

```tsx
const reducer = (state: State, action: Action) => state;
```