# CSS Transition

## 개요
CSS Transition은 CSS 속성을 변경할 때 애니메이션 속도를 조절하는 방법을 제공한다. CSS Transition을 활용하면 CSS 속성이 변경될 때 **즉시 변화가 일어나게 하는 대신, 일정 기간에 걸쳐서 일어나도록** 할 수 있다.

![](https://velog.velcdn.com/images/mskwon/post/e79294dd-b0d1-461c-882a-90f576b926c2/image.gif)

위 GIF는 CSS Transition을 안썼을때와 썼을때의 차이점을 보여주는 간단한 예시다.
- 왼쪽의 박스는 CSS Transition을 쓰지 않은 예시로, 속성 값 변경이 발생할 때 즉시 변화가 일어난다.
- 오른쪽의 박스는 CSS Transition을 사용한 예시로, 속성 값 변경이 발생할때 지정한 Transition 속성에 맞게 변화가 서서히 일어난다.

CSS Transition을 활용하여 구현한 애니메이션은 시작과 종료 상태 사이의 상태를 브라우저가 암묵적으로 정하게 되는데, 이를 **암묵적 트랜지션**이라고 부른다.

## 사용방법
CSS 트랜지션을 설정하는 가장 간단한 방법은 단축 CSS 속성 `transition`을 사용하는 것이다.

```css
div {
	transition: <property> <duration> <timing-function> <delay>;
}
```

또는 아래와 같은 관련 속성들을 직접 지정해주는 방법이 있다.

- `transition-property`
    - 트랜지션을 **적용해야하는 CSS 속성의 이름 혹은 이름들**을 명시
    - 여기에 나열한 속성만 트랜지션으로 동작함
    - 다른 모든 속성에 대한 변화는 보통 때와 같이 즉시 일어남
- `transition-duration`
    - 트랜지션이 일어나는 **지속시간**을 명시
    - 트랜지션동안 모든 속성에 적용하는 단일 지속시간을 명시하거나, 다른 주기로 각 속성이 트랜지션하게 하는 여러 지속시간을 명시할 수 있음
- `transition-timing-function`
    - 속성의 **중간값을 계산하는 방법**을 정의하는 함수를 명시
    - 대부분 타이밍 함수는 큐빅 베이지어를 정의하는 네 점에 의해 정의되므로, 상응하는 함수의 그래프로 제공해서 명시할 수 있음
- `transition-delay`
    - 속성이 변한 시점과 트랜지션이 실제로 시작하는 사이에 **기다리는 시간**을 정의

```css
div {
  transition-property: background-color;
  transition-duration: 1s;
  transition-timing-function: linear;
  transition-delay: 200ms;
}
```

위 두가지 방법중 하나를 선택하여 원하는 요소에 적용해주면, 조건에 맞는 CSS 속성의 변경이 발생할때 트랜지션 애니메이션이 실행된다. 개발을 하다보면 React, Vue 등 상태값에 따라 특정 요소의 CSS 속성을 바꾸게 되는 경우가 많은데, JS 단의 코드를 추가하지 않고도 아주 간단하게 부드러운 애니메이션을 추가할 수 있는 방법인 셈이다.

아래 코드는 위 GIF 이미지의 실제 리액트 코드다. JS 단에서는 조건에 따라 CSS 속성값만 변경해주면, CSS Transition을 통해서 브라우저가 알아서 "암묵적 트랜지션"을 수행해준다.

```tsx
import { useState } from "react";

export default function App() {
  const [toggle, setToggle] = useState(false);

  return (
    <div
      style={{
        display: "flex",
        alignItems: "flex-start",
        gap: 20,
      }}
    >
      <div
        style={{
          width: 100,
          height: 100,
          backgroundColor: toggle ? "red" : "blue",
        }}
      />

      <div
        style={{
          width: 100,
          height: 100,
          backgroundColor: toggle ? "red" : "blue",

          transitionProperty: "background-color",
          transitionDuration: "1s",
        }}
      />

      <button onClick={() => setToggle((prev) => !prev)}>색상변경</button>
    </div>
  );
}
```

## Transition 완료 감지
트랜지션을 완료하면 발생하는 `transitionend` / `webkitTransitionEnd`(Webkit) 이벤트가 발생하게된다. 이 이벤트는 두 속성을 제공한다.

- `propertyName`
    - 트랜지션을 **완료한 CSS 속성의 이름**을 나타내는 문자열
- `elapsedTime`
    - **이벤트가 발생한 시점에 해당 트랜지션이 진행된 시간을 초로 나타내는 실수**
    - `transition-delay` 값에 **영향을 받지 않음**

트랜지션을 중단하면 `transitionend` 이벤트는 발생하지 않음. 트랜지션을 완료하기 전에 애니메이션하고 있는 속성의 값이 바뀌기 때문.

이걸 어떻게 활용할 수 있을지 생각해봤는데.. 딱히 사용할 일이 많을 것 같지는 않다.
- 여러 요소에 순차적으로 Transition 적용
  - A -> B -> C 순으로 Transition을 적용하고싶을 때 사용할 수 있지 않을까?
  - 근데 이 경우 그냥 keyframe을 쓰는게 나을 것 같음
- Transition 완료 후에 UI 변경
  - 특정 트랜지션 애니메이션이 완료된 후에 UI를 변경시키고 싶을때?

## 주의사항
CSS Transition을 적용하려는 속성에 `auto` 값을 지정하는 경우, 브라우저에 따라서 동작할수도 안할수도, 애니메이션의 방식이 모두 다를수도 있다.

간단한 예시로, Collapse/Uncollapse 기능에 간단한 애니메이션을 추가하기 위해 아래와 같은 코드를 작성해봤다.

```tsx
import { useState } from "react";

export default function App() {
  const [showMore, setShowMore] = useState(false);

  return (
    <div>
      <div style={{ marginTop: 20 }}>
        어쩌구저쩌구...
        <button onClick={() => setShowMore((prev) => !prev)}>
          {showMore ? "가리기" : "더보기"}
        </button>
        <div
          style={{
            overflow: "hidden",
            height: showMore ? "auto" : 0,
            transitionProperty: "height",
            transitionDuration: "1s",
          }}
        >
          Adipisicing ea dolor eiusmod. Aute minim cupidatat voluptate ea anim
          deserunt do nisi velit excepteur cillum aute do tempor laboris. Ea
          elit enim do. Dolor aliquip officia anim ipsum aliquip irure irure
          irure ad aliqua proident sit dolore labore.
        </div>
      </div>
    </div>
  );
}
```
`height` 속성값이 `0` <-> `auto` 왔다갔다 할 때 애니메이션이 추가될 것이라고 생각하겠지만, 현실은 아래와같다.

![](https://velog.velcdn.com/images/mskwon/post/f770f8d9-c8b8-4750-a565-08aa2873f14e/image.gif)

이 경우 간단한 workaround로 트랜지션 속성을 `max-height`로 바꾸고, 적당한 `max-height` 값 지정으로 바꾸는 방법이 있다.

```tsx
<div
  style={{
    overflow: "hidden",
    maxHeight: showMore ? 100 : 0,
    transitionProperty: "max-height",
    transitionDuration: "1s",
  }}
 >
```

![](https://velog.velcdn.com/images/mskwon/post/d9674ccd-00bd-4619-ba52-20359f874b1e/image.gif)


또는 아직 실험적 기능인 [`calc-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/calc-size) CSS 함수를 이용하는 방법이 있긴하고 결과가 제일 깔끔하지만, 아직 사파리/파이어폭스 등의 메이저 브라우저에서 지원하지 않아 상용환경에서 사용하기에는 무리가 있다.

```tsx
<div
  style={{
    overflow: "hidden",
	height: showMore ? "calc-size(auto, size)" : 0,
    transitionProperty: "height",
    transitionDuration: "1s",
  }}
 >
```

![](https://velog.velcdn.com/images/mskwon/post/c6d83669-89df-4c8d-8896-28ffb23e9e6b/image.gif)
