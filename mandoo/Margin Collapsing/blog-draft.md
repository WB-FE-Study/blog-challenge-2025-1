## Margin Collapsing

Margin Collapsing은 두 개 이상의 블록 요소의 상하 마진이 겹칠 때, 각 마진 값을 따로 적용하지 않고 한쪽의 값만 적용하는 브라우저의 렌더링 규칙이다.

Margin Collapsing은 크게 3가지 상황에서 발생할 수 있다.

### 인접 형제 블록 간의 Margin Collapsing
인접한 형제 블록이 각각의 margin을 가지고 있고 서로 겹치는 경우, 더 큰 값으로 상쇄되어 렌더링이 된다. 만약 겹쳐진 값이 동일한 경우, 중복을 상쇄하고 렌더링이 된다.

<iframe width="100%" height="300" src="//jsfiddle.net/mageeeeek/un0eqbp3/4/embedded/html,css,result/" frameborder="0" loading="lazy" allowtransparency="true" allowfullscreen="true"></iframe>

위 예시에서 첫번째 요소는 `margin-bottom` 값으로 10px을, 두 번째 요소는 `margin-top` 값으로 40px를 가지고 있는데, Margin Collapsing으로 인해 더 큰 값인 두번째 요소의 40px로 두 요소 간의 여백공간이 결정된다.

![](https://velog.velcdn.com/images/mskwon/post/d41b06d1-18e2-43ef-808f-c72a03ea8a11/image.png)

### 부모 자식 블록 간의 Margin Collapsing
부모와 자식 블록 간의 "경계"가 없다면 Margin Collapsing이 발생한다. 여기서 경계란, CSS border/padding 또는 인라인 콘텐츠가 있는지 여부를 말한다. 
Margin Collapsing이 발생했다면, margin 값은 부모/자식 사이공간이 아닌 부모의 margin 값으로 적용된다.

<iframe width="100%" height="300" src="//jsfiddle.net/mageeeeek/fLzpa5kv/5/embedded/html,css,result/" frameborder="0" loading="lazy" allowtransparency="true" allowfullscreen="true"></iframe>

위 예시에서 부모 요소는 `margin-top` 값으로 20px, 자식 요소는 `margin-top` 값으로 80px를 가지고 있다. "경계"가 없는 상황에서는 Margin Collapsing이 적용되어 부모 쪽에 `margin-top` 값으로 80px이 적용된다.

![](https://velog.velcdn.com/images/mskwon/post/5e308aed-6028-45b6-bd77-5d547d219bef/image.png)

하지만 border/padding 등을 통해서 부모와 자식 사이의 "경계"가 있는 경우에는, 부모/자식 쪽에 따로따로 margin 값이 적용된다.

![](https://velog.velcdn.com/images/mskwon/post/9ab5e718-f0f7-426c-b27a-a234c69e750e/image.png)

![](https://velog.velcdn.com/images/mskwon/post/cf28e640-0820-42e3-bab0-bd87b5629dcd/image.png)

### 빈블럭 Margin Collapsing
"빈 블럭"에 `margin-top` / `margin-bottom` 값이 적용되어 있는 경우, 더 큰 값으로 Margin Collapsing 되어 한 쪽의 margin만 적용된다.

여기서 "빈 블럭"이란, `border`, `padding`, `height`, `min-height` 속성이 명시되어 있지 않고, 인라인콘텐츠 또한 없는 블럭을 말한다.

<iframe width="100%" height="300" src="//jsfiddle.net/mageeeeek/spcdhtnv/19/embedded/html,css,result/" frameborder="0" loading="lazy" allowtransparency="true" allowfullscreen="true"></iframe>

위 예시에서 가운데에 껴 있는 요소는 `margin-top`/`margin-bottom` 값으로 모두 40px를 가지고 있지만, "빈 블럭"으로 정의된 위쪽 케이스의 경우 Margin Collapsing이 되어 실제로는 40px 만큼의 공간만 차지한다.

![](https://velog.velcdn.com/images/mskwon/post/1ba72d6d-1b34-4aac-84e7-8549f28722b0/image.png)

반면 아래쪽 케이스의 경우 `height` 값이 명시되어 있어, Margin Collapsing이 발생하지 않아 위아래로 모두 40px 만큼의 공간을 차지한다.

![](https://velog.velcdn.com/images/mskwon/post/957f89f8-ee7f-44a0-878a-cdd129e53349/image.png)

### Margin Collapsing이 발생하지 않는 상황
- `position` 속성 값이 `float` 또는 `absolute`인 요소
- `display` 속성 값이 `flex` 또는 `grid`인 요소

위 경우에는 Margin Collapsing이 발생하지 않는다. 

### 느낀점
- 부끄럽지만 몇 년간 FE 개발을 해왔음에도, Margin Collapsing이라는게 있는지도 몰랐다. 보통 flexbox를 이용해서 코드를 작성해왔기 때문에.. 그렇지 않을까 싶다.
- 이걸 계속 몰랐으면 미래에 Margin Collapsing에 의해서 예상하지 못한 렌더링 결과가 발생할 때 많이 당황했을 것 같다.
- 부모/자식/형제 등 복합적인 상황에서는 더 복잡하게 Margin Collapsing이 동작한다고 한다. 최대한 예측가능한 코드를 작성하기 위해, Margin Collapsing을 피해가는게 최선일수도 있겠다는 생각이 든다.

### Reference
- https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_model/Mastering_margin_collapsing
- https://velog.io/@raram2/CSS-%EB%A7%88%EC%A7%84-%EC%83%81%EC%87%84Margin-collapsing-%EC%9B%90%EB%A6%AC-%EC%99%84%EB%B2%BD-%EC%9D%B4%ED%95%B4
- https://www.joshwcomeau.com/css/rules-of-margin-collapse/