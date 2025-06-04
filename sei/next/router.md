# 📍 Router 구현 원리 톺아보기 - prefetch

이전 글에서는 Next.js Link 태그의 구현 원리를 톺아보면서 아래와 같이 정리했습니다.

- `<Link>`는 사용자가 링크를 클릭하기 전, 해당 경로의 데이터를 미리 가져와 빠른 전환을 가능하게 합니다.
- 뷰포트 진입 또는 마우스 hover 시 자동으로 prefetch가 실행됩니다.
- 동일한 링크에 대해 여러 번 prefetch하지 않도록 내부적으로 Set을 사용해 캐싱을 수행합니다.

이 과정에서 `<Link>` 컴포넌트는 결국 Next.js의 router의 기능을 활용해 `<a>`를 래핑한 컴포넌트임을 알 수 있었습니다.
자연스럽게 "그렇다면 이 router는 어떤 원리로 동작할까?"라는 궁금증이 생겼고, 이번 글에서는 그 구현 원리에 대해 정리해보려 합니다.

> 코드는 Next.js 15.3 버전을 기준으로 합니다.
> 주요 파일
> - router.ts
> - router-loader.ts
> - page-loader.ts

## Prefetch 구현 원리

결론부터 말씀드리면 prefetch는 DOM의 link 태그를 사용하여 구현되고 있습니다.

page-loader.ts의 prefetch의 반환값을 살펴보면, 또 다시 routerLoader.prefetch의 반환값을 반환하고 있습니다.

```ts
prefetch(route: string): Promise<void> {
  return this.routeLoader.prefetch(route)
}
```

prefetch의 구현체를 보면, prefetch가 가능한 경우인지 검사하고 스크립트를 prefetch하는 것을 볼 수 있습니다.

```ts
/**
 * 1. prefetch 할 리소스 경로를 가져온다.
 * 2. prefetch를 지원하는 환경인지 체크한다.
 * 3. link 태그를 생성해 DOM에 부착한다.
 */
prefetch(route: string): Promise<void> {
  /** NOTE: 네트워크 상태 체크 */
  // https://github.com/GoogleChromeLabs/quicklink/blob/453a661fa1fa940e2d2e044452398e38c67a98fb/src/index.mjs#L115-L118
  // License: Apache 2.0
  let cn
  if ((cn = (navigator as any).connection)) {
    // Don't prefetch if using 2G or if Save-Data is enabled.
    if (cn.saveData || /2g/.test(cn.effectiveType)) return Promise.resolve()
  }
  return (
    getFilesForRoute(assetPrefix, route)
      // 프리페치 가능한 경우에만 스크립트 프리페치
      .then((output) =>
        Promise.all(
          canPrefetch
            ? output.scripts.map((script) =>
                prefetchViaDom(script.toString(), 'script')
              )
            : []
        )
      )
      .then(() => {
        requestIdleCallback(() =>
          this.loadRoute(route, true).catch(() => {})
        )
      })
      .catch(
        // swallow prefetch errors
        () => {}
      )
  )
},

// canPrefetch
/**
 * 프리페치 가능한 브라우저인지 확인하는 함수.
 * IE11 브라우저는 프리페치를 지원하지만 감지되지 않음.(relList.support를 지원하지 않아서)
 */
function hasPrefetch(link?: HTMLLinkElement): boolean {
  try {
    link = document.createElement('link')
    return (
      // detect IE11 since it supports prefetch but isn't detected
      // with relList.support
      (!!window.MSInputMethodContext && !!(document as any).documentMode) ||
      link.relList.supports('prefetch')
    )
  } catch {
    return false
  }
}
```

prefetchViaDom 함수를 타고 들어가면, 드디어 우리가 찾았던 로직이 나타납니다.
```ts
function prefetchViaDom(
  href: string,
  as: string,
  link?: HTMLLinkElement
): Promise<any> {
  return new Promise<void>((resolve, reject) => {
    /**
     * NOTE, HIGHLIGHT
     *
     * 이미 프리페치된 리소스가 있는지 확인하는 함수.
     * 이미 프리페치된 리소스가 있으면 프리페치를 수행하지 않는다.
     */
    const selector = `
      link[rel="prefetch"][href^="${href}"],
      link[rel="preload"][href^="${href}"],
      script[src^="${href}"]`
    if (document.querySelector(selector)) {
      return resolve()
    }

    /**
     * NOTE, HIGHLIGHT
     *
     * 프리페치를 위한 링크 요소를 생성한다.
     */
    link = document.createElement('link')

    // 순서는 의도된 것임.
    if (as) link!.as = as
    link!.rel = `prefetch`
    link!.crossOrigin = process.env.__NEXT_CROSS_ORIGIN!
    link!.onload = resolve as any
    link!.onerror = () =>
      reject(markAssetError(new Error(`Failed to prefetch: ${href}`)))

    // `href` should always be last:
    link!.href = href

    document.head.appendChild(link)
  })
}
```

## Prefetch 최적화

prefetch는 무분별하게 실행되지 않도록 여러 조건과 최적화 기법들을 사용하고 있었습니다.

### 실행 조건

먼저 실행 조건입니다. prefetch는 production 환경이 아니거나 Bot일 때 실행되지 않도록 처리되어 있었습니다.

```tsx
// router.ts
// on-demand로 동작하기 때문에 개발 환경에서는 프리페치 작동 안함
if (process.env.NODE_ENV !== 'production') {
  return
}

// 링크를 렌더링하는 봇(bot)에 대해서는 프리페치를 수행하지 않습니다.
if (typeof window !== 'undefined' && isBot(window.navigator.userAgent)) {
  return
}

// is-bot.ts
export function isBot(userAgent: string): boolean {
  return isDomBotUA(userAgent) || isHtmlLimitedBotUA(userAgent)
}

function isDomBotUA(userAgent: string) {
  return HEADLESS_BROWSER_BOT_UA_RE.test(userAgent)
}

const HEADLESS_BROWSER_BOT_UA_RE =
  /Googlebot|Google-PageRenderer|AdsBot-Google|googleweblight|Storebot-Google/i
```

흥미로웠던 점은 bot이 접속한 경우 prefetch가 동작하지 않게 처리된 부분이었습니다. bot의 경우 하드 네비게이션으로 링크를 탐색하기 때문에 prefetch를 제외한 것이 타당하다고 느껴졌습니다.

### 최적화 기법

Next.js 는 정적/동적 라우팅 탐색 시 이를 최적화하기 위해 Bloom Filter라는 자료구조를 활용하고 있습니다.

Bloom Filter란 확률적인 자료구조로, 어떤 요소가 set에 존재할 '가능성'을 알 수 있습니다.
Bloom Filter를 사용하면 '집합에 속한 원소를 속하지 않았다고 말하는 일'은 0%입니다. 다만 '속하지 않은 원소를 가끔 속해 있다'고는 얘기할 수 있습니다. 이를 False Negative하다고 이야기합니다.

예를 들어 ['a', 'b', 'c']의 원소가 있을 때, 'x' 가 존재하는지 여부는 틀릴 수 있지만, 'a'가 존재하는지는 확실하게 알 수 있습니다.

원리는 간단합니다. 해싱 함수를 이용해 원소를 해싱한 다음, Bloom Filter에 해시를 마스킹합니다. 이는 요소마다 진행이되고, 요소가 많을 수록 Bloom Filter에 마스킹된 비트가 많아집니다. [링크 참고](https://llimllib.github.io/bloomfilter-tutorial/)

Bloom Filter는 데이터 탐색 과정에서 전처리 단계에 활용되어 데이터베이스의 요청 부하를 줄이기 위해 활용됩니다.

Next.js에서는 정적 라우트와 동적 라우트를 각각의 bfl에 저장하여, 실제 존재하는 라우트인지 검사할 때 활용하고 있습니다.

```ts
/**
 * NOTE, HIGHLIGHT
 *
 * - 클라이언트 사이드에서 처리 가능한 라우트는 클라이언트에서, 서버 사이드는 하드 네비게이션으로 처리.
 * - Bloom Filter를 사용하여 메모리 사용량 최적화.(정체 라우트 목록을 저장하지 않고 비트 배열만 사용)
 */
async _bfl(
  as: string,
  resolvedAs?: string,
  locale?: string | false,
  skipNavigate?: boolean
) {
  if (process.env.__NEXT_CLIENT_ROUTER_FILTER_ENABLED) {
    if (!this._bfl_s && !this._bfl_d) {
      // 정적 라우트, 동적 라우트 필터가 모두 없는 경우
      const { BloomFilter } =
        require('../../lib/bloom-filter') as typeof import('../../lib/bloom-filter')

      type Filter = ReturnType<
        import('../../lib/bloom-filter').BloomFilter['export']
      >
      let staticFilterData: Filter | undefined
      let dynamicFilterData: Filter | undefined

      try {
        ;({
          __routerFilterStatic: staticFilterData,
          __routerFilterDynamic: dynamicFilterData,
        } = (await getClientBuildManifest()) as any as {
          __routerFilterStatic?: Filter
          __routerFilterDynamic?: Filter
        })
      } catch (err) {
        // failed to load build manifest hard navigate
        // to be safe
        console.error(err)
        if (skipNavigate) {
          return true
        }
        handleHardNavigation({
          url: addBasePath(
            addLocale(as, locale || this.locale, this.defaultLocale)
          ),
          router: this,
        })
        return new Promise(() => {})
      }

      const routerFilterSValue: Filter | false = process.env
        .__NEXT_CLIENT_ROUTER_S_FILTER as any

      if (!staticFilterData && routerFilterSValue) {
        staticFilterData = routerFilterSValue ? routerFilterSValue : undefined
      }

      const routerFilterDValue: Filter | false = process.env
        .__NEXT_CLIENT_ROUTER_D_FILTER as any

      if (!dynamicFilterData && routerFilterDValue) {
        dynamicFilterData = routerFilterDValue
          ? routerFilterDValue
          : undefined
      }

      if (staticFilterData?.numHashes) {
        this._bfl_s = new BloomFilter(
          staticFilterData.numItems,
          staticFilterData.errorRate
        )
        this._bfl_s.import(staticFilterData)
      }

      if (dynamicFilterData?.numHashes) {
        this._bfl_d = new BloomFilter(
          dynamicFilterData.numItems,
          dynamicFilterData.errorRate
        )
        this._bfl_d.import(dynamicFilterData)
      }
    }

    let matchesBflStatic = false
    let matchesBflDynamic = false
    const pathsToCheck: Array<{ as?: string; allowMatchCurrent?: boolean }> =
      [{ as }, { as: resolvedAs }]

    for (const { as: curAs, allowMatchCurrent } of pathsToCheck) {
      if (curAs) {
        const asNoSlash = removeTrailingSlash(
          new URL(curAs, 'http://n').pathname
        )
        const asNoSlashLocale = addBasePath(
          addLocale(asNoSlash, locale || this.locale)
        )

        if (
          allowMatchCurrent ||
          asNoSlash !==
            removeTrailingSlash(new URL(this.asPath, 'http://n').pathname)
        ) {
          matchesBflStatic =
            matchesBflStatic ||
            !!this._bfl_s?.contains(asNoSlash) ||
            !!this._bfl_s?.contains(asNoSlashLocale)

          for (const normalizedAS of [asNoSlash, asNoSlashLocale]) {
            // if any sub-path of as matches a dynamic filter path
            // it should be hard navigated
            const curAsParts = normalizedAS.split('/')
            for (
              let i = 0;
              !matchesBflDynamic && i < curAsParts.length + 1;
              i++
            ) {
              const currentPart = curAsParts.slice(0, i).join('/')
              if (currentPart && this._bfl_d?.contains(currentPart)) {
                matchesBflDynamic = true
                break
              }
            }
          }

          // if the client router filter is matched then we trigger
          // a hard navigation
          if (matchesBflStatic || matchesBflDynamic) {
            if (skipNavigate) {
              return true
            }
            handleHardNavigation({
              url: addBasePath(
                addLocale(as, locale || this.locale, this.defaultLocale)
              ),
              router: this,
            })
            return new Promise(() => {})
          }
        }
      }
    }
  }
  return false
}
```


https://steemit.com/kr-dev/@heejin/bloom-filter