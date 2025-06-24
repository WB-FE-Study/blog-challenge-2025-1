환경변수 인풋은 애플리케이션의 중요한 부분중 하나로, Turborepo 구성시에 유심히 살펴봐야한다.

## 태스크 해시에 환경변수 추가하기

애플리케이션 동작에 변화를 줄 수 있는 환경 변수들에 대해서는, `turbo.json` 파일에서 `env`, `globalEnv` 키값을 지정해줘야한다.

```json
{
  "globalEnv": ["IMPORTANT_GLOBAL_VARIABLE"],
  "tasks": {
    "build": {
      "env": ["MY_API_URL", "MY_API_KEY"]
    }
  }
}
```

- `globalEnv`
    - 이 리스트에 있는 환경변수들은 모든 태스크의 해시에 영향을 준다
- `env`
    - 태스크에 영향을 주는 환경변수들

터보리포는 와일드카드 문자를 지원한다. 환경변수 지정시 prefix 등을 사용할 수 있음을 의미한다.

## 프레임워크 추론

터보리포는 자동으로 `env` 키에 prefix 와일드카드들을 통상적인 프레임워크에 대해 추가해준다. 아래 프레임워크들을 사용하고 있다면, prefix를 지정해줄 필요가 없다.

| **Framework** | **`env` wildcards** |
| --- | --- |
| Astro | `PUBLIC_*` |
| Blitz | `NEXT_PUBLIC_*` |
| Create React App | `REACT_APP_*` |
| Gatsby | `GATSBY_*` |
| Next.js | `NEXT_PUBLIC_*` |
| Nitro | `NITRO_*`, `SERVER_*`, `AWS_APP_ID`, `INPUT_AZURE_STATIC_WEB_APPS_API_TOKEN`, `CLEAVR`, `CF_PAGES`, `FIREBASE_APP_HOSTING`, `NETLIFY`, `STORMKIT`, `NOW_BUILDER`, `ZEABUR`, `RENDER` |
| Nuxt.js | `NUXT_*`, `NITRO_*`, `SERVER_*`, `AWS_APP_ID`, `INPUT_AZURE_STATIC_WEB_APPS_API_TOKEN`, `CLEAVR`, `CF_PAGES`, `FIREBASE_APP_HOSTING`, `NETLIFY`, `STORMKIT`, `NOW_BUILDER`, `ZEABUR`, `RENDER` |
| RedwoodJS | `REDWOOD_ENV_*` |
| Sanity Studio | `SANITY_STUDIO_*` |
| Solid | `VITE_*` |
| SvelteKit | `VITE_*`, `PUBLIC_*` |
| Vite | `VITE_*` |
| Vue | `VUE_APP_*` |

프레임워크 추론에서 opt-out 하고싶다면..

- `--framework-inference=false`로 태스크 실행
- `env` 키에 negative 와일드카드를 추가한다
    - `"env": ["!NEXT_PUBLIC_*"]`

## 환경 모드

태스크의 런타임에 어떤 환경변수를 사용가능하게 할지 정할 수 있게 해주는 모드다.

- Strict Mode (디폴트)
    - `env`, `globalEnv` 키에 있는 환경변수만 필터링
- Loose Mode
    - 프로세스의 모든 환경변수를 사용가능하게 함

### Strict Mode

`env`, `globalEnv` 키에 있는 환경변수만 필터링하여 사용가능하게한다. 만약 특정 태스크가 필요로 하는 환경변수가 있으나 명시되지 않았다면, 해당 태스크의 실행이 실패할 가능성이 높다.

**Passthrough Variables**

몇몇 심화 케이스에서는, 몇몇 환경변수들을 해시에 포함시키지 않되 태스크가 사용가능하게 하고싶을 수 있다. 이 경우에는, 해당 환경변수들을 `globalPassThroughEnv`와 `passThroughEnv`에 추가해주면된다.

**CI Vendor Compatibility**

`turbo.json`에서 명시해주지 않았다면 CI Vendor들의 환경변수를 모두 필터링해버릴 것이다. 이들을 사용해줘야한다면 반드시 설정파일에 명시해줘야한다.

### Loose Mode

환경변수에 대한 아무 필터링을 하지 않는다. `--env-mode` 플래그를 사용해서 실행할 수 있다.

하지만 이 모드에 의존할 경우, 환경변수를 컨피그에 포함시키지 않는 케이스가 발생할 가능성이 높고, 캐시 히트가 발생하면 안되는데 캐시 히트가 발생하는 경우가 나올 수 있다.

따라서 Strict Mode를 사용하는 것이 권장된다.

### 플랫폼 환경변수

Vercel에 애플리케이션을 배포하는 경우, 이미 프로젝트에 환경변수가 세팅되어있을 수 있다. 터보리포는 해당 변수들을 `turbo.json` 설정과 비교하여 확인하고, 있어야하는데 없는 변수가 있다면 경고해준다.

`TURBO_PLATFORM_ENV_DISABLED=false` 설정을 통해서 이 동작을 끌 수 있다.

## `.env` 파일 핸들링

터보리포는 `.env` 파일들을 태스크 런타임에 로드해오지 않고, 프레임워크나 `dotenv`와 같은 툴이 알아서하게 둔다.

하지만, `.env` 파일에 변화가 있는 경우 해시에 사용되도록 설정해줘야한다. 이 경우 `inputs` 키에 환경변수 파일을 넣어준다.

```json
{
  "globalDependencies": [".env"], // All task hashes
  "tasks": {
    "build": {
      "inputs": ["$TURBO_DEFAULT$", ".env"] // Only the `build` task hash
    }
  }
}
```

### 여러개의 `.env` 파일들

```json
{
  "globalDependencies": [".env"], // All task hashes
  "tasks": {
    "build": {
      "inputs": ["$TURBO_DEFAULT$", ".env*"] // Only the `build` task hash
    }
  }
}
```

## Best Practices

### 패키지 내의 `.env` 파일들 사용

루트에서 `.env` 파일들을 사용하는 것은 권장되지 않는다. 패키지 단에서 직접 `.env` 파일을 선언하고 사용하는 것이 권장된다.

### `eslint-config-turbo` 사용

이 패키지를 사용하면 코드에서는 사용되지만 `turbo.json`에 리스팅되지 않은 환경변수들을 찾아내는데 도움이 된다.

### 런타임에 환경변수 만드는 것은 피하자

터보리포는 태스크가 시작될 때 환경변수를 해싱하는데 사용한다. 태스크가 실행되는 도중 환경변수가 생성되거나 한다면 터보리포는 해당 환경변수에 대해 알 수 없고, 태스크 해시에도 포함되지 않는다.

```json
{
  "scripts": {
    "dev": "export MY_VARIABLE=123 && next dev"
  }
}
```

위와 같이 설정된 경우 `MY_VARIABLE`에 대해서 터보리포는 알지 못한다.
