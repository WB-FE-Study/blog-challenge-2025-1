## QueryClient

모든 것은 `QueryClient`에서 시작된다. 앱이 시작될 때 클래스 인스턴스로 만들어지고, `QueryClientProvider`에 의해서 전역에 뿌려지는 클래스다.

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

// ⬇️ this creates the client
const queryClient = new QueryClient()

function App() {
  return (
    // ⬇️ this distributes the client
    <QueryClientProvider client={queryClient}>
      <RestOfYourApp />
    </QueryClientProvider>
  )
}
```

내부적으로 `QueryClientProvider`는 리액트 컨텍스트를 사용해서 앱 전역에 `QueryClient` 인스턴스를 뿌려준다. 클라이언트 자체는 안정적인 값으로, 실수로 여러번 만들어지도록 하지만 않으면 바뀌지 않는다.

![](https://velog.velcdn.com/images/mskwon/post/bc912a52-a4ec-42eb-9c64-ec6f8c121f54/image.png)


`QueryClient` 자체는 사실 많은 것을 하지는 않는다. `QueryCache`와 `MutationCache`를 가지고 있는 컨테이너이고, 이 친구들이 `QueryClient` 인스턴스가 만들어질 때 생성된다.

## QueryCache

![](https://velog.velcdn.com/images/mskwon/post/f27f1b6f-92c1-40fd-9933-e5c2fc390348/image.png)


`QueryCache`는 메모리에 존재하는 객체로, 키값으로 직렬화된 쿼리키가, 값으로 `Query` 클래스의 인스턴스가 들어간다.

중요하게 알아둬야 할 점은, 리액트 쿼리가 디폴트로는 데이터를 메모리에만 저장한다는 것이다. 즉 브라우저 페이지를 새로고침 하게되면, 캐시는 사라진다. 만약 이 캐시를 어딘가에 영속하고싶다면 persister를 사용해야한다.

## Query

![](https://velog.velcdn.com/images/mskwon/post/1eab7f3c-ae4f-4d97-88d3-e3746c463e03/image.png)


`Query`는 대부분의 로직이 존재하는 곳으로, 쿼리에 대한 정보들을 포함할 뿐 아니라 쿼리 함수를 실행하기도 하고 재시도/취소/중복제거 로직이 들어간다.

내부에는 state machine을 가지고 있어서, 이상한 상태가 될 일은 없다. 이미 fetching하고 있을 때 queryFn이 호출될 때 중복제거 로직이 들어갈 것이고, 쿼리가 취소되면 이전 상태로 돌아간다.

가장 중요한 것은, Query 입장에서 누가 쿼리 데이터에 관심을 가지고 있는지 알고 있는 것이다. `Observer`를 통해서 이 정보를 알 수 있으며, 변화가 발생했을때 옵저버에게 알리면 된다.

## QueryObserver

![](https://velog.velcdn.com/images/mskwon/post/973805c9-b414-4fc9-8027-ea90acfbc06f/image.png)


옵저버는 `Query`와 그것을 필요로 하는 컴포넌트들 간의 연결고리다. `useQuery` 등을 호출했을 때 옵저버가 생성되며, 반드시 한번에 하나의 쿼리와만 연결된다. 이런 이유로 `queryKey`를 넘겨야하는 것이다.

옵저버의 역할은 좀 더 많다. 대부분의 최적화가 발생하는 곳이 바로 옵저버다. 옵저버는 컴포넌트가 쿼리의 어떤 프로퍼티에 관심있는지 알고 있어서 불필요한 리렌더를 발생시키지 않는다.

여기에 더해서, 각 `Observer`는 select 옵션을 가지며, 이를 통해서 데이터의 어떤 필드에 관심을 가지고 있는지 알 수 있다. `staleTime`, interval fetcing 등에 사용되는 타이머들도 옵저버 레벨에 존재한다.

## Active and Inactive queries

`Observer`가 하나도 연결되지 않은 `Query`를 inactive 쿼리라고 부른다. 계속 캐시에 존재하긴 하지만, 아무 컴포넌트도 해당 쿼리를 사용하고 있지 않기 때문이다.

## Complete Picture

![](https://velog.velcdn.com/images/mskwon/post/59a5b250-89d7-4c20-9608-9e26e03d1e93/image.png)


큰 그림으로 보면, 대부분의 로직이 프레임워크에 의존하지 않는 쿼리코어에 있다는 것을 알 수 있다. `QueryClient`, `QueryCache`, `Query`, `QueryObserver` 모두 코어에 있다.

이런 이유로 프레임워크에 맞는 어댑터를 만들기가 어렵지 않다. 옵저버를 만들고, 구독하고, 옵저버가 알림을 받았을 때 리렌더링을 발생시키게 해주면된다. 실제로 react/solid의 `useQuery` 어댑터는 100줄 정도의 코드로만 작성되어있다.

## 컴포넌트 관점

![](https://velog.velcdn.com/images/mskwon/post/b46c7072-38a6-4f42-83ae-eebf7eb813ea/image.png)


- 컴포넌트가 마운트 되고, `useQuery`가 호출되어 `Observer`가 생성된다.
- `Observer`는 `Query`를 구독하며, 쿼리는 `QueryCache` 내에 있다.
    - 구독을 하기 위해서 `Query`가 생성될 수 있으며(존재하지 않았다면), 데이터가 낡은 것으로 판명된다면 백그라운드 리페칭이 발생할 수 있다.
- 페칭이 시작되면 `Query`의 상태가 바뀌며, `Observer`도 여기에 대한 알림을 받게된다
    - 옵저버는 필요에 따라서 컴포넌트에게 상태 업데이트를 알려서 새로운 상태로 렌더를 발생시킨다
- `Query`가 실행되고 난 뒤, 다시한번 옵저버에게 알린다
