[CSS 트랜지션 사용하기 - CSS: Cascading Style Sheets | MDN](https://developer.mozilla.org/ko/docs/Web/CSS/CSS_transitions/Using_CSS_transitions)s

CSS Transition은 특정 CSS 속성 값을 점진적으로 바꿀 수 있는 CSS 모듈으로, 타이밍/함수/시간 등 여러 속성을 통해 값이 바뀌는 방식을 조절할 수 있다.

속성 변경이 **즉시 영향을 미치게 하는 대신, 그 속성의 변화가 일정 기간에 걸쳐 일어나도록** 할 수 있다.

**암묵적 트랜지션**

- 두 상태 사이의 트랜지션을 포함하는 애니메이션
- 시작과 종료 상태 사이의 상태를 브라우저가 **암묵적으로 정의**하기 때문

CSS Transition은

- 어떤 속성을 움직이게 할지
- 언제 애니메이션이 시작할지
- 얼마나 지속할지
- 어떻게 트랜지션을 실행하는지 결정하게 함

## 애니메이션이 가능한 CSS 속성

모든 속성이 애니메이션 가능한 것은 아니다. 애니메이션이 가능한 속성의 리스트는 시간이 지남에 따라 변할 수 있다. **`auto` 값은 매우 복잡한 케이스다. 브라우저에 따라 예상하지 못한 결과를 초래**할 수 있어서, 피하는 것이 좋다.

## 트랜지션 정의에 사용한 CSS 속성

CSS 트랜지션은 단축 속성 `transition`을 사용하여 제어한다. 트랜지션을 설정하는 가장 좋은 방법이다.

- `transition-property`
    - 트랜지션을 적용해야하는 CSS 속성의 이름 혹은 이름들을 명시
    - 여기에 나열한 속성만 트랜지션하는동안 움직임
    - 다른 모든 속성에 대한 변화는 보통 떄와 같이 즉시 일어남
- `transition-duration`
    - 트랜지션이 일어나는 지속시간을 명시
    - 트랜지션동안 모든 속성에 적용하는 단일 지속시간을 명시하거나, 다른 주기로 각 속성이 트랜지션하게 하는 여러 지속시간을 명시할 수 있음
- `transition-timing-function`
    - 속성의 중간값을 계산하는 방법을 정의하는 함수를 명시
    - 대부분 타이밍 함수는 큐빅 베이지어를 정의하는 네 점에 의해 정의되므로, 상응하는 함수의 그래프로 제공해서 명시할 수 있음
    - https://easings.net/ 를 참고해서 이징을 선택할 수도 있음
- `transition-delay`
    - 속성이 변한 시점과 트랜지션이 실제로 시작하는 사이에 기다리는 시간을 정의

CSS 단축 문법으로 사용하면 아래와 같은 형태가 됨

```css
div {
	transition: <property> <duration> <timing-function> <delay>;
}
```

## 트랜지션 완료 감지하기

트랜지션을 완료하면 발생하는 단일 이벤트 : `transitionend` / `webkitTransitionEnd`(Webkit)

이벤트는 두 속성을 제공함

- `propertyName`
    - 트랜지션을 **완료한 CSS 속성의 이름**을 나타내는 문자열
- `elapsedTime`
    - **이벤트가 발생한 시점에 해당 트랜지션이 진행된 시간을 초로 나타내는 실수**
    - `transition-delay` 값에 **영향을 받지 않음**

```jsx
el.addEventListener("transitionend", updateTransition, true);
```

트랜지션을 중단하면 `transitionend` 이벤트는 발생하지 않음. 트랜지션을 완료하기 전에 애니메이션하고 있는 속성의 값이 바뀌기 때문.

## 속성값 목록이 다른 개수를 가진 경우

어떤 속성의 값 목록이 다른 것보다 짧다면, 일치되도록 그 값을 반복한다

```css
div {
  transition-property: opacity, left, top, height;
  transition-duration: 3s, 5s;
}
```

위 CSS는 아래와 같이 동작한다

```css
div {
  transition-property: opacity, left, top, height;
  transition-duration: 3s, 5s, 3s, 5s;
}
```

비슷하게 어떤 속성의 값 목록이 `transition-property` 목록보다 길다면, 끝을 잘라낸다

```css
div {
  transition-property: opacity, left;
  transition-duration: 3s, 5s, 2s, 1s;
}
```

위 CSS는 아래와 같이 동작한다.

```css
div {
  transition-property: opacity, left;
  transition-duration: 3s, 5s;
}
```