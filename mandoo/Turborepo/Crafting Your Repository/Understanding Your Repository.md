## `turbo ls`

패키지를 리스트 할 때 사용한다. 어떤 패키지들이 있고 어디에 위치하는지 알 수 있다.

```bash
> turbo ls
 
  @repo/eslint-config packages/eslint-config
  @repo/typescript-config packages/typescript-config
  @repo/ui packages/ui
  docs apps/docs
  web apps/web
```

필터 사용도 가능

```bash
> turbo ls --filter ...ui
3 packages (pnpm9)
 
  @repo/ui packages/ui
  docs apps/docs
  web apps/web
```

## `turbo run`

어떤 태스크들을 실행할 수 있는지 보려면, 그냥 `turbo run`만 실행한다.

```bash
> turbo run
No tasks provided, here are some potential ones
 
  lint
    @repo/ui, docs, web
  build
    docs, web
  dev
    docs, web
  start
    docs, web
  generate:component
    @repo/ui
```

## `turbo query`

리포지토리 구조를 좀 더 파보고싶다면, 2.2 버전부터 리포지토리 구조에 대한 GraphQL 인터페이스를 제공한다. 이를 이용해서 특정 태스크가 있는 패키지들을 모두 찾아내는 등의 작업을 할 수 있다.

```bash
> turbo query "query { packages(filter: { has: { field: TASK_NAME, value: \"build\"}}) { items { name } } }"
{
  "data": {
    "packages": {
      "items": [
        {
          "name": "//"
        },
        {
          "name": "docs"
        },
        {
          "name": "web"
        }
      ]
    }
  }
}
```

이를 활용해서 패키지의 의존성 그래프에 있는 문제에 대한 원인을 탐색하는데 도움을 받을 수 있다. 예를 들어 10번이상 direct 임포트 되는 패키지를 찾는다거나..

```bash
> turbo query "query { packages(filter: { greaterThan: { field: DIRECT_DEPENDENT_COUNT, value: 10 } }) { items { name } } }"
{
  "data": {
    "packages": {
      "items": [
        {
          "name": "utils"
        }
      ]
    }
  }
}
```

`--affected` 플래그를 사용하는데, 의도치않게 더 많은 태스크가 실행되는 것처럼 느껴져서 원인을 찾고싶다거나..

```bash
> turbo query "query { affectedPackages(base: \"HEAD^\", head: \"HEAD\") { items { reason {  __typename } } } }"
{
  "data": {
    "affectedPackages": {
      "items": [
        {
          "name": "utils",
          "reason": {
            "__typename": "FileChanged"
          }
        },
        {
          "name": "web",
          "reason": {
            "__typename": "DependencyChanged"
          }
        },
        {
          "name": "docs",
          "reason": {
            "__typename": "DependencyChanged"
          }
        },
        {
          "name": "cli",
          "reason": {
            "__typename": "DependencyChanged"
          }
        },
      ]
    }
  }
}
```
