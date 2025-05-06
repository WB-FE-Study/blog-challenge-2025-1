(내용 보충할 예정입니다!)

# 📍 Next.js에서 Navigation - `<Link>` 와 `<a>`

Next.js에서 페이지 간 이동(Navigation)은 웹 앱의 핵심 요소입니다.
Next 프로젝트에서는 다양한 방식으로 경로를 이동할 수 있으며, 상황에 따라 알맞은 Navigation 방식을 선택하는 것이 중요합니다.

이 글에서는 Next.js의 App Router를 기준으로 사용 가능한 Navigation 방법들을 간단히 정리하고,
그중에서도 가장 자주 사용되는 `<Link>`와 `<a>` 태그를 중심으로 왜 `<Link>`를 사용하는 것이 좋은지,
그리고 `<Link>`가 단순한 링크 이상의 역할을 한다는 것을 구현 원리까지 파고들어 살펴보겠습니다.

## Next.js에서 Navigation 방법

Next.js는 클라이언트, 서버, DOM 수준에서 다양한 Navigation API를 제공합니다. 아래 표는 대표적인 방식과 그 특성을 정리한 것입니다.

| 방법 | 특징 | 동작 위치 | import 위치 |
| --- | --- | --- | --- |
| `<Link href="..."/>` | SPA 방식 기본 내비게이션 | 클라이언트 | `import Link from 'next/link';` |
| `router.push()` | 자바스크립트로 동적 이동 | 클라이언트 | `import { useRouter } from "next/navigation";` |
| `router.replace()` | 스택 쌓지 않고 이동 | 클라이언트 | `import { useRouter } from "next/navigation";` |
| `router.back()` | 뒤로 가기 | 클라이언트 | `import { useRouter } from "next/navigation";` |
| `redirect()` | 서버 사이드 강제 이동 | 서버 컴포넌트 | `import { redirect } from 'next/navigation';` |
| `<a href="...">` | 전체 리로드 | HTML 기본 | x |

이 많은 API 중 어떤 상황에서 어떤 API를 사용해야 할까요?

### 동적 URL 업데이트가 필요한 경우: useRouter

검색, 필터, 정렬처럼 사용자의 입력을 URL에 반영해야 하는 경우에는 useRouter의 router.push()를 활용하는 것이 유용합니다.

```tsx
'use client';
import { useRouter, useSearchParams } from 'next/navigation';
import { useState } from 'react';

export default function SearchBox() {
  const router = useRouter();
  const searchParams = useSearchParams();
  const [query, setQuery] = useState(searchParams.get('q') ?? '');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    const params = new URLSearchParams(searchParams);
    query ? params.set('q', query) : params.delete('q');
    router.push(`?${params.toString()}`);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <button type="submit">검색</button>
    </form>
  );
}
```

이처럼 JavaScript 기반 Navigation은 UI 상태를 URL에 반영해야 할 때 유용하며, 유연한 사용자 경험을 제공합니다.

### 클릭해서 다른 페이지로 이동하는 경우

반면, 사용자가 버튼이나 카드 등을 클릭해 다른 페이지로 이동해야 하는 일반적인 상황이라면, URL을 직접 조작하는 것보다 명시적인 링크를 사용하는 것이 좋습니다.
이렇게 해야 검색 엔진이 href를 인식하고 페이지 간의 연결 구조를 이해할 수 있기 때문입니다.

Next.js에서는 `<a>` 태그 대신 `<Link>` 컴포넌트를 사용함으로써 접근성(Accessibility)을 지키면서도 클라이언트 라우팅의 이점을 함께 누릴 수 있습니다.

```tsx
// 명시적인 링크 이동
<Link href="/about">About</Link>

// 게시물 상세 페이지로 이동
<Link href={`/post/${id}`}>게시물 제목</Link>
```

#### ✅ 왜 꼭 `<Link>`를 써야 할까?

Next.js의 `<Link>`는 최종적으로 브라우저에서 `<a>` 태그로 렌더링되지만,
자바스크립트를 활용해 기본 동작을 제어하고 다양한 기능을 추가할 수 있게 설계되어 있습니다.

브라우저 기본 UX 유지 (새 탭 열기, 링크 복사 등 지원)

페이지 리소스를 미리 불러오는 prefetch 기능 지원

즉, `<Link>`는 단순한 링크 이상으로, 사용자 경험과 성능을 모두 고려한 컴포넌트입니다.
그렇다면 이 기능들은 내부적으로 어떻게 구현되어 있을까요?

<br />

## `<Link>` 구현 원리

`<Link>`는 렌더링 시 `<a>`로 출력되지만, 클릭 이벤트와 렌더링 타이밍에서 특별한 로직을 수행합니다.

### Soft Navigation과 replace: 전체 리로드 없이 경로 전환하기

Next.js는 `<a>` 태그 클릭 시 브라우저의 기본 동작(Hard Navigation)을 차단하고, 내부 라우터를 이용해 URL을 변경합니다.
이 과정을 통해 페이지를 새로고침하지 않고도 자연스럽게 화면을 전환할 수 있습니다.

```tsx
function linkClicked(
  e: React.MouseEvent,
  router: NextRouter | AppRouterInstance,
  href: string,
  as: string,
  replace?: boolean,
  shallow?: boolean,
  scroll?: boolean,
  locale?: string | false
): void {
  // 브라우저 기본 리로드를 막음
  e.preventDefault()

  const routerScroll = scroll ?? true
  // Next.js의 router를 활용해 페이지 전환
  if ('beforePopState' in router) {
    router[replace ? 'replace' : 'push'](href, as, {
      shallow,
      locale,
      scroll: routerScroll,
    })
  } else {
    router[replace ? 'replace' : 'push'](as || href, {
      scroll: routerScroll,
    })
  }
}

const handleClick = (e: React.MouseEvent<HTMLAnchorElement>) => {
  if (!router || e.defaultPrevented) return
  linkClicked(e, router, href, as, replace, shallow, scroll, locale)
}
```

핵심은 e.preventDefault()로 브라우저의 기본 리로드를 막고, router.push 또는 router.replace를 호출하는 부분입니다. 이를 통해 Next.js는 기존 SPA처럼 빠른 페이지 전환을 구현합니다.

### Prefetch: 클릭 전에 리소스 미리 불러오기

Next.js는 사용자 경험을 더 빠르게 만들기 위해 링크에 마우스를 올리거나 화면에 보이는 시점에 미리 데이터를 불러오는 prefetch 기능을 제공합니다.

```tsx
function prefetch(
  router: NextRouter,
  href: string,
  as: string,
  options: PrefetchOptions
): void {
  // ...생략
  router.prefetch(href, as, options).catch((err) => {
    if (process.env.NODE_ENV !== 'production') {
      throw err
    }
  })}
```

이 함수는 다음 조건에서 실행됩니다.

1. 프로덕션 환경일 것
2. prefetch 옵션이 활성화되어 있을 것
3. 링크 요소가 뷰포트에 진입하거나 마우스가 hover 되었을 때

```tsx
useEffect(() => {
  if (process.env.NODE_ENV !== 'production') return
  if (!router) return
  if (!isVisible || !prefetchEnabled) return
  prefetch(router, href, as, { locale })
}, [as, href, isVisible, locale, prefetchEnabled, router?.locale, router])
```

마우스가 링크 위에 올라갔을 때도 prefetch가 실행됩니다. 이때는 중복 여부를 무시하고 강제로 실행할 수 있습니다.

```tsx
const handleMouseEnter = () => {
  if (!router) return
  prefetch(router, href, as, {
    locale,
    priority: true,
    bypassPrefetchedCheck: true,
  })
}
```

prefetch()가 자주 실행되면 성능에 부담이 될 수 있습니다. 이를 위해 최적화 코드도 있습니다.
prefetch() 함수 정의 부분에 사실 이런 코드가 생략되어 있었습니다.

```tsx
const prefetched = new Set<string>()

function prefetch(
  router: NextRouter,
  href: string,
  as: string,
  options: PrefetchOptions
): void {
  if (typeof window === 'undefined') return
  if (!isLocalURL(href)) return

  if (!options.bypassPrefetchedCheck) {
    const locale = typeof options.locale !== 'undefined' ? options.locale : 'locale' in router ? router.locale : undefined
    const prefetchedKey = href + '%' + as + '%' + locale
    if (prefetched.has(prefetchedKey)) return
    prefetched.add(prefetchedKey)
  }

   router.prefetch(href, as, options).catch((err) => {
    if (process.env.NODE_ENV !== 'production') {
      throw err
    }
  })
}
```

Set 자료구조를 활용하여 같은 URL 조합(href + as + locale) 에 대해서는 한 번만 prefetch하도록 처리하는 것을 확인할 수 있습니다.

#### Prefetch 요약

- `<Link>`는 사용자가 링크를 클릭하기 전, 해당 경로의 데이터를 미리 가져와 빠른 전환을 가능하게 합니다.
- 뷰포트 진입 또는 마우스 hover 시 자동으로 prefetch가 실행됩니다.
- 동일한 링크에 대해 여러 번 prefetch하지 않도록 내부적으로 Set을 사용해 캐싱을 수행합니다.

## 정리하기

여기까지 Next.js에서 사용할 수 있는 다양한 Navigation 방법을 살펴보고, 그중에서도 `<Link>`의 역할과 구현 방식을 중심으로 정리해보았습니다.

UI 상에서 명확한 이동이 필요한 경우에는 `<Link>`가 기본값이 되고, 필터나 검색처럼 상태를 URL에 반영해야 할 때는 router.push() 같은 방식이 적절합니다.

특히 `<Link>`를 사용할 때는 prefetch를 적절하게 사용해 사용자 경험을 끌어올릴 수 있다는 점을 확인했습니다.

오늘 스터디를 통해 Navigation은 단순히 API 선택의 문제가 아니라,
사용자 흐름과 맥락을 어떻게 설계할 것인가에 대한 고민이라는 점을 다시 한 번 느꼈습니다.

이 글이 여러분에게도 그런 고민을 시작해보는 계기가 되었기를 바랍니다. 🙂

---

### 참고자료
- [Next.js `<Link>`](https://nextjs.org/docs/14/app/api-reference/components/link#replace)
- [Next.js GitHub - `next.js/packages/next/src/client
/link.tsx`](https://github.com/vercel/next.js/blob/canary/packages/next/src/client/link.tsx)