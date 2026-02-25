자세히 알지 못하지만 회사에서 쓰고 있으니 나도 그냥 쓰고 있는 기술, Turborepo에 대해 좀 더 자세히 이해하고 사용할 수 있길 바라며 글을 씁니다.

## Turborepo는 무엇인가

> Turborepo is a high-performance build system for JavaScript and TypeScript codebases. It is designed for scaling monorepos and also makes workflows in single-package workspaces faster, too.
[(ref)](https://turborepo.com/docs)

Turborepo는 현시점 자바스크립트/타입스크립트 생태계에서 가장 인기많고 강력한 모노레포 관리 툴이다. 여러가지 프로젝트를 하나의 repo에 때려박는 모노레포는 많은 장점을 가지고 있지만, 크기가 커질수록 실행/관리해야하는 태스크의 수가 많아지고 각종 태스크의 실행시간이 너무 오래걸리는 문제점이 있다.

그리고 그 문제를 Turborepo가 해결해준다. Turborepo는 리모트 캐시를 활용해서 불필요한 중복 태스크 실행을 방지하고, 병렬로 수행할 수 있는 태스크는 병렬로 수행하는 등 여러가지 최적화를 해준다.

## `create-turbo`로 시작하기
이미 존재하는 repo에 Turborepo를 도입하는 것도 어렵지 않지만, 일단 백지에서 시작하는 것을 기준으로 시작해보자.

요즘 툴 답게 CLI에서 명령어 하나로 간단하게 시작할 수 있다. npm이 깔려있는 환경에서 아래와 같이 터미널에 입력해주면된다.

```js
npx create-turbo@latest
```
![](https://velog.velcdn.com/images/mskwon/post/2989d782-8c8e-48d0-a28f-46ce2d389858/image.png)

물어보는 질문에 친절하게 몇번 답변해주면 끝난다.
- 어디에 repo를 만들것인지
- 어떤 패키지 매니저를 사용할 것인지

이후 내가 지정해준 폴더에 들어가보면, 사람이 하면 귀찮을 일들을 Turborepo CLI가 손수 나서서 아래와 같이 다 세팅해줬음을 확인할 수 있다.
![](https://velog.velcdn.com/images/mskwon/post/29bebfa3-07c7-4e76-90d8-b7ceb92b96a0/image.png)

## Turborepo 구성품 살펴보기
### 패키지 명시하기
일단 어디에 개별 패키지(프로젝트)들이 있는지 패키지매니저에게 알려야한다.

이 부분은 패키지 매니저별로 다르기에 사용하는 패키지 매니저에 맞는 방법을 찾으면된다. 우리의 경우에는 아까 PNPM으로 Turborepo가 세팅해줬기에, `pnpm-workspace.yaml`에 그 내용이 있다.

```yaml
packages:
  - "apps/*"
  - "packages/*"
```

이 파일의 내용은 단순하다. `apps` 폴더의 하위 폴더들과 `packages` 폴더의 하위 폴더들에 패키지가 있다는 것을 알리는 것이다.

`apps` 폴더와 `packages` 폴더로 나눈 것은 Turborepo가 디폴트로 세팅해주며 권장하는 방법으로, `app` 폴더에는 말그대로 애플리케이션들이 들어가고, `packages`에는 기타 라이브러리, 툴 등이 들어간다.

### 루트 `package.json`
repo 최상단에 있는 `package.json`은 워크스페이스의 기초가 되는 파일으로, 아래와 같이 간단하게 구성된다.

```json
{
  "name": "turbo-tutorial",
  "private": true,
  "scripts": {
    "build": "turbo run build",
    "dev": "turbo run dev",
    "lint": "turbo run lint",
    "format": "prettier --write \"**/*.{ts,tsx,md}\"",
    "check-types": "turbo run check-types"
  },
  "devDependencies": {
    "prettier": "^3.5.3",
    "turbo": "^2.5.4",
    "typescript": "5.8.2"
  },
  "packageManager": "pnpm@9.0.0",
  "engines": {
    "node": ">=18"
  }
}
```

루트에서 실행할 명령어들, 그리고 거기에 필요한 의존성 등이 명시된다.

### 루트 `turbo.json`
`tubo.json` 파일은 `turbo`의 동작방식을 설정하기 위해 존재하는 파일로, 아래와 같이 구성된다.

```json
{
  "$schema": "https://turborepo.com/schema.json",
  "ui": "tui",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "inputs": ["$TURBO_DEFAULT$", ".env*"],
      "outputs": [".next/**", "!.next/cache/**"]
    },
    "lint": {
      "dependsOn": ["^lint"]
    },
    "check-types": {
      "dependsOn": ["^check-types"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```
이 파일에서 `turbo`를 통해서 실행할 태스크들에 대한 설정을 할 수 있다. 상세한 내용은 추후 작성하겠지만, 간단하게 보자면 `build`, `lint`, `check-types`, `dev` 태스크가 있고, 각 태스크 실행/캐싱과 관련된 설정이 그 하위에 들어가있다.

### 패키지 매니저 락파일
기본적으로 패키지 매니저의 일관성 있는 동작을 보장해주기 위한 파일로, Turborepo는 이 파일을 활용해서 내부 패키지들의 의존성을 이해한다.

파일의 이름과 내용은 패키지 매니저별로 다르며, 나는 `pnpm`으로 세팅했기에 `pnpm-lock.yaml` 파일에서 그 내용을 확인해볼 수 있다. 하지만 사람이 이 파일을 직접 건드릴 일은 없다.

### 각 패키지 내의 `package.json`
`apps`, `pacakges` 폴더 하위의 폴더들이 모두 `package.json`을 가지고 있음을 볼 수 있다. 이 폴더들이 모두 각각의 "프로젝트"라고 생각하면 편하다. 각각의 프로젝트는 서로다른 `pacakge.json`을 가지고 있으며, 여기서 해당 프로젝트의 이름, 필요로 하는 의존성, 스크립트 등을 정의할 수 있다.

간단하게 `pacakges/ui/package.json`을 살펴보면 아래와 같다.

```json
{
  "name": "@repo/ui",
  "version": "0.0.0",
  "private": true,
  "exports": {
    "./*": "./src/*.tsx"
  },
  "scripts": {
    "lint": "eslint . --max-warnings 0",
    "generate:component": "turbo gen react-component",
    "check-types": "tsc --noEmit"
  },
  "devDependencies": {
    "@repo/eslint-config": "workspace:*",
    "@repo/typescript-config": "workspace:*",
    "@turbo/gen": "^2.5.0",
    "@types/node": "^22.15.3",
    "@types/react": "19.1.0",
    "@types/react-dom": "19.1.1",
    "eslint": "^9.28.0",
    "typescript": "5.8.2"
  },
  "dependencies": {
    "react": "^19.1.0",
    "react-dom": "^19.1.0"
  }
}
```

여기서 `dependencies`, `devDependencies`와 같은 친구들은 익숙할 것이고, Turborepo 관점에서 중요한 친구들은 아래와 같다.
- `name`
  - 이 패키지의 식별자로, **repository 내에서 유니크** 해야한다.
- `scripts`
  - 패키지의 컨텍스트 내에서 실행할 수 있는 스크립트를 정의하기 위해 사용된다.
  - Turborepo는 이 스크립트들의 이름을 활용해서 어떤 패키지의 어떤 스크립트를 실행할지 식별한다.
- `exports`
  - 이 패키지를 repository 내의 다른 패키지들이 사용하기 위한 엔트리포인를 정의하기 위해 사용된다.
  - 위와 같이 와일드 카드를 이용해 정의했다면, `src` 하위의 모든 `*.tsx` 파일들을 다른 패키지들이 import하여 사용할 수 있게된다.
   ```tsx
	// 다른 패키지에서 import
	import { Button } from "@repo/ui/button";
	``` 
  - 이 정보를 이용해서 IDE도 자동완성 기능 등을 제공해줄 수 있다.

## Reference
- https://turborepo.com/docs/crafting-your-repository/structuring-a-repository