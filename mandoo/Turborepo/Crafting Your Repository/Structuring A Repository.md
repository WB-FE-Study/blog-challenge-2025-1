`turbo`는 자바스크립트 생태계에 존재하는 패키지 매니저의 기능인 **Workspaces**를 기반으로 만들어졌다. 이는 하나의 저장소에 여러 패키지를 그루핑할 수 있게 해주는 기능이다.

## Anatomy of a workspace

자바스크립트에서, 워크스페이스는 하나의 패키지 일수도 있고 여러 패키지의 모음일수도 있다. 이 가이드에서는 여러개의 패키지로 구성된 워크스페이스(모노리포)에 집중한다.

### 최소 요구사항

- 패키지 매니저에 의해 패키지가 명시되어야 함
- 패키지 매니저 락파일
- 루트 `package.json`
- 루트 `turbo.json`
- 각 패키지 내의 `package.json`

### 모노리포에서 패키지 명시하기

우선, 패키지 매니저가 패키지들의 위치를 설명할 수 있어야한다. 각 패키지들을 `apps`(애플리케이션들) 또는 `packages`(패키지들)로 나누는 것으로 시작하는 것이 권장된다.

```yaml
packages:
  - "apps/*"
  - "packages/*"
```

위 설정이면, `apps` / `packages` 디렉토리 내의  `package.json`을 가지고 있는 모든 디렉토리가 패키지로 인식된다.

### 루트 `package.json`

워크스페이스의 베이스가 되는 파일이다. 아래는 간단한 예시

```yaml
{
  "private": true,
  "scripts": {
    "build": "turbo run build",
    "dev": "turbo run dev",
    "lint": "turbo run lint"
  },
  "devDependencies": {
    "turbo": "latest"
  },
  "packageManager": "pnpm@9.0.0"
}
```

### 루트 `turbo.json`

`turbo`의 동작을 설정하기 위한 파일이다. 자세한 설명은 생략한다.

### 패키지 매니저 락파일

락파일은 패키지 매니저와 `turbo`의 재현가능한 동작을 위한 키다. 추가로, **Turborepo는 락파일을 활용해서 내부 패키지들의 의존성을 이해한다.**

## Anatomy of a package

패키지를 독립적으로 설계하는 것이 권장된다. 높은 레벨에서, 각 패키지는 작은 프로젝트에 가깝다. 패키지가 자신의 `package.json`, 도구 구성, 소스코드를 가지고 있기 때문이다. 이렇게 생각하는 것에 분명한 한계가 있지만, 이렇게 생각을 시작하는 것이 편하다.

또한, 패키지는 **워크스페이스 내의 다른 패키지들이 사용하기 위해 접근할 수 있도록 해주는 `exports` 엔트리포인트**를 가진다.

### 패키지를 위한 `package.json`

- `name`
    - 패키지의 식별자다. 워크스페이스 내에서 유니크해야한다.
- `scripts`
    - 패키지의 컨텍스트 내에서 실행할 수 있는 스크립트를 정의하기 위해 사용된다.
    - Turborepo는 이 스크립트들의 이름을 활용해서 어떤 패키지의 어떤 스크립트를 실행할지 식별한다.
- `exports`
    - `exports` 필드는 이 패키지를 다른 패키지들이 사용하기 위한 엔트리포인트를 정의하기 위해 사용된다.
        
        ```json
        {
          "exports": {
            ".": "./src/constants.ts",
            "./add": "./src/add.ts",
            "./subtract": "./src/subtract.ts"
          }
        }
        ```
        
    - 위처럼 설정해두면 다른 패키지에서 아래와 같은 방법으로 접근가능해진다.
        
        ```tsx
        import { GRAVITATIONAL_CONSTANT, SPEED_OF_LIGHT } from '@repo/math';
        import { add } from '@repo/math/add';
        import { subtract } from '@repo/math/subtract';
        ```
        
    - 이 방식으로 활용했을 때 3가지 큰 이점이 있다
        - 배럴 파일 사용 피하기
            - 편해보이지만, 배럴 파일은 컴파일러나 번들러 입장에서 핸들링하기 쉽지 않으며 성능 문제로 이어질 가능성이 크다.
        - 더 강력한 기능
            - `main` 필드를 사용하는 것에 비해 더 강력한 기능들을 제공한다(예를 들어서 [조건부 export](https://nodejs.org/api/packages.html#conditional-exports))
        - IDE 자동완성
            - 코드 에디터가 해당 패키지의 exports에 있는 것들에 대해 자동완성을 제공할 수 있다
- `imports`
    - 패키지 내의 다른 모듈에 대한 subpath를 만들 수 있게 해준다.
    - 더 간단한 import path를 쓰기 위한 숏컷으로 생각해도 좋다. 리팩토링 등의 작업이 있을때 파일을 옮기는 것이 상대적으로 쉬워진다.
    - 타입스크립트의 `compilerOptions.paths` 옵션을 대체할 수 있음
- 소스코드
    - 패키지 내의 소스코드
    - 일반적으로 `src` 디렉토리에 소스코드를 저장하고, `dist` 디렉토리에 컴파일하여 사용한다. (필수는 아님)

## Common Pitfalls

- 타입스크립트를 사용중이면, 워크스페이스의 루트에 `tsconfig.json`을 사용할 필요가 없다. 각 패키지들은 각자의 config를 독립적으로 가지고 있어야 하고, 워크스페이스 내에서 공유하는 `tsconfig.json`을 가지는 것이 일반적이다.
- 패키지 바운더리를 넘어가서 파일에 접근하는 것은 지양해야한다. 필요한 패키지는 import해와서 쓰는 것이 맞다.