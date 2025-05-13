# useTransition

# useTransition

### useTransition의 개념

- useTransition 리액트 공식 문서의 서술
    
    `useTransition은 UI의 일부를 백그라운드에서 렌더링 할 수 있도록 해주는 React Hook입니다.` 
    
    `const [isPending, startTransition] = useTransition()` 
    
- useTransition이란
    - ui의 non-blocking 상태로 state 업데이트를 관리할 수 있는 react hook이다. 비동기 작업이나 복잡한 계산을 실행할 때 유효하다.
    - `isPending` : 대기 중인 transition이 있는지 여부를 알려준다.
    - `startTransition` : 상태 업데이트를 transition으로 표시할 수 있다.
    
    간단한 예시는 다음과 같다.
    
    ```jsx
    function TabContainer() {
      const [isPending, startTransition] = useTransition();
      const [tab, setTab] = useState('about');
    
      function selectTab(nextTab) {
        startTransition(() => {
          setTab(nextTab);
        });
      }
      // ...
    }
    ```
    
- 실질적인 예시를 통해 각각의 역할을 알아보기
    
    ```jsx
    import { useState, useTransition } from 'react';
    
    function SearchComponent() {
      const [query, setQuery] = useState('');
      const [results, setResults] = useState([]);
      const [isPending, startTransition] = useTransition();
    
      const handleChange = (e) => {
        // 높은 우선순위
        setQuery(e.target.value);
        
        // 낮은 우선순위
        startTransition(() => {
          const searchResults = performSearch(e.target.value);
          setResults(searchResults);
        });
      };
    
      return (
        <>
          <input value={query} onChange={handleChange} />
          {isPending ? (
            <p>검색 중...</p>
          ) : (
            <ul>
              {results.map(item => <li key={item.id}>{item.name}</li>)}
            </ul>
          )}
        </>
      );
    }
    ```
    
    위의 예시는 `<input>`에 검색어를 입력하고, `performSearch` 를 통해 검색하는 작업을 의미한다.
    
    여기서 `performSearch` 는 위에서 언급한 “복잡한 계산”의 예시로 볼 수 있다.
    
    순차적으로 사용자가 ‘a’, ‘ab’, ‘abc’를 입력했을 때의 단계별 화면 변경사항에 대해 살펴보자.
    
    1. ‘a’ 입력 시
        
        `handleChange`가 호출된다. 이때 높은 우선순위를 가진 `setQuery('a')` 가 실행된다. input값이 ‘a’로 업데이트 되면서 사용자가 보는 화면에도 ‘a’가 노출된다.
        
        이때 화면이 업데이트 되면서, `startTransition` 작업도 시작된다. `isPending` 이 `true` 로 바뀌면서 로딩이 활성화 된다. 이때는 `performSearch(e.target.value)` 작업을 실행하고자 한다.
        
    2. ‘ab’를 빠르게 입력했다. 즉, 기존의 ‘a’에 ‘b’를 하나 더 입력하여 ‘ab’가 되었다.
        
        `handleChange` 가 다시 호출된다. `setQuery('ab')` 가 실행되고, input값은 ‘ab’가 되어 곧바로 화면에 노출된다. 
        
        그러나 낮은 우선순위였던 ‘a’에 대한 trasition은 취소되고, 새로운 `startTransition` 이 시작된다. 이때 `isPending` 은 `true` 로 유지되고 있으므로 화면에도 로딩이 계속해서 노출된다.
        
    3. 사용자가 타이핑을 멈췄다. ( ‘ab’에서 더이상 변화가 일어나지 않는다.)
        
        브라우저가 여유시간을 감지하고, `performSearch` 를 실제로 실행한다. 이때 실행이 완료되면 `isPending` 이 `false`로 바뀌며 결과 목록이 렌더링 된다.
        
        - [추가공부필요] 여유시간 감지 → 리액트 스케줄러 / fiber 개념 확인
            - https://d2.naver.com/helloworld/2690975
            - https://jser.dev/react/2022/03/16/how-react-scheduler-works

- startTransition(action) 구조 살펴보기
    - `startTransition` 은 `action` 을 파라미터로 받는다. 여기서 `action` 이란, set함수를 한 개 이상 불러 상태를 업데이트하는 함수를 의미한다. `startTransition` 에서 매개변수 없이 즉시 action이 호출되고, action의 함수 호출 내의 동기적 상태 업데이트를 모두 transition으로 표시한다.
    - 반환값은 없다.
    - 내부에서 비동기 호출 사용이 가능하다. 단, `await` 이후의 `set` 함수 호출은 별도의 `startTransition` 으로 감싸야 한다.
        - [**`React는 await 이후의 상태 업데이트를 Transition으로 처리하지 않습니다.`**](https://ko.react.dev/reference/react/useTransition#react-doesnt-treat-my-state-update-after-await-as-a-transition)
        - javascript의 한계로 인해, asyncContext의 범위를 잃기 때문이다.
            - javascript에서 await 이후의 실행은 새로운 실행 컨텍스트에서 일어나기 때문에 기존에 react가 transition과 관련하여 기록한 플래그를 읽을 수 없다.
        
        ```jsx
        startTransition(async () => {
          await someAsyncFunction();
          // 두 번 감싸야만 원하는 방식대로 작동하게 된다.
          startTransition(() => {
            performSearch(e.target.value)
          });
        });
        ```
        

### useTransition사용 시 유의점

- useTransition은 Hook이므로 컴포넌트나 커스텀 hook 내부에서만 호출 가능하다. 컴포넌트 외부에서 호출하려면 독립형 `startTransition` 을 호출해야 한다.
    - `startTransition` 의 경우 일반 함수이다. 따라서 전환 상태(`isPending` )를 반환하지 않는다. 단지 콜백 내부의 상태 업데이트를 비긴급으로 표기할 뿐이다.
- 해당하는 state의 set함수에 액세스할 수 있는 경우에만 업데이트를 transition으로 래핑 가능하다. prop이나 커스텀 Hook값에 대한 응답으로 transition을 시작하려면 `useDeferredValue` 를 사용해야 한다.
- `startTransition`에 전달한 함수는 즉시 실행된다. React는 이 함수를 즉시 실행하여 실행하는 동안 발생하는 모든 state 업데이트를 Transition으로 표기한다. 그러나 `setTimeout` 와 같이 현재 실행 스택이 완료 된 후 state 업데이트를 시도하는 함수는 Transition으로 표시되지 않는다.
    - setTimeout 예시
        
        ```jsx
        'use client';
        
        import { useState, startTransition } from 'react';
        
        export default function TransitionTestPage() {
          const [results, setResults] = useState<string | null>(null);
        
          const handleClick = () => {
            console.log("1");
        
            startTransition(() => {
              console.log("2");
        
              setTimeout(() => {
                console.log("4");
                setResults("Some data");
              }, 0);
        
              console.log("3");
            });
        
            console.log("5");
          };
        
          return (
            <div style={{ padding: 20 }}>
              <h1>React Transition Test</h1>
              <button onClick={handleClick}>Run Test</button>
              <div style={{ marginTop: 10 }}>Results: {results}</div>
            </div>
          );
        }
        //결과값
        // 1
        // 2
        // 3
        // 5
        // 4
        ```
        
- [추가공부필요]  *`startTransition` 함수는 안정된 식별성(stable identity)을 가지므로 Effect 의존성에서 생략되는 경우가 많습니다. 하지만 포함해도 Effect가 실행되지는 않습니다. linter가 의존성을 생략해도 오류를 발생시키지 않는다면 생략해도 안전합니다. [자세한 내용은 Effect 의존성 제거에 대한 문서를 참고하세요.](https://ko.react.dev/learn/removing-effect-dependencies#move-dynamic-objects-and-functions-inside-your-effect)*


---

https://mycodings.fly.dev/blog/2024-07-14-react-19-tutorial-1-usetransition-and-new-form-api

https://velog.io/@kyeun95/React-useTransition%EC%9D%B4%EB%9E%80

https://www.youtube.com/watch?v=N5R6NL3UE7I

https://medium.com/@lovleshpokra/react-19-how-to-use-usetransition-useoptimistic-and-useactionstatehooks-d77352c03128

https://medium.com/@jakintemi/usetransition-react-19-25a7bb376818

https://d2.naver.com/helloworld/2690975

https://velog.io/@tnghgks/React-Concurrent-mode

https://jser.dev/2023-05-19-how-does-usetransition-work/

https://stackoverflow.com/questions/74993789/how-this-usetransition-hook-works-in-reactjs

https://velog.io/@happygyu/%EB%B2%88%EC%97%AD-React-%EC%8A%A4%EC%BC%80%EC%A4%84%EB%9F%AC%EB%8A%94-%EB%82%B4%EB%B6%80%EC%97%90%EC%84%9C-%EC%96%B4%EB%96%BB%EA%B2%8C-%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C

https://stackoverflow.com/questions/35193867/react-non-blocking-rendering-of-big-chunks-of-data

https://www.linkedin.com/pulse/blocking-vs-non-blocking-decoding-heart-efficient-reactjs-kalara-uyzaf/

https://ko.react.dev/reference/react/useTransition#react-doesnt-treat-my-state-update-after-await-as-a-transition

https://ko.react.dev/reference/react/startTransition

https://github.com/reactjs/rfcs/pull/229

https://junilhwang.github.io/TIL/Javascript/Domain/Execution-Context/#_1-%E1%84%80%E1%85%A2%E1%84%82%E1%85%A7%E1%86%B7

https://jy-beak.tistory.com/181#01