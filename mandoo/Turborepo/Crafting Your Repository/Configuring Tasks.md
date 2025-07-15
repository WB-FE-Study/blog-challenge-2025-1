Turborepo는 항상 `turbo.json` 컨피그와 패키지 그래프에 설정된 순서대로 태스크를 실행한다. 그 와중에 병렬화할 수 있는 태스크가 있다면, 알아서 병렬화하게된다. 이는 당연히 태스크를 한번에 하나씩 실행하는 것보다 빠른 결과를 낳게 된다.

## Getting Started

저장소 루트의 `turbo.json` 파일을 통해서 Turborepo가 실행할 태스크들을 정의한다. 태스크가 정의되었다면, `turbo run` 명령어를 통해서 하나 이상의 태스크를 실행할 수 있게된다.

- 아예 새로 시작하는 단계라면, `create-turbo`를 이용해서 새로운 저장소를 만들고 `turbo.json` 파일을 수정하는 것을 추천한다.
- 이미 저장소가 존재하는 상황이라면, `turbo.json` 파일을 루트에 만들면 된다.

## 태스크 정의하기

`tasks` 객체의 각 키는 `turbo run`에 의해서 실행될 수 있는 독립적인 태스크 객체다. Turborepo가 각 패키지의 `package.json` 파일의 스크립트들을 탐색해서 동일한 이름의 태스크가 있는지 찾아보게된다.

`turbo.json`의 `tasks` 객체를 추가해줌으로 써 간단하게 태스크를 추가할 수 있다. 아래는 의존성이 없고 output도 없는 `build` 태스크를 추가한 것이다.

```json
{
  "tasks": {
    "build": {} // Incorrect!
  }
}
```

이 시점에 `turbo run build`를 실행하면, Turborepo는 패키지 내에 존재하는 모든 `build` 스크립트를 병렬로 실행하고 아무 output도 캐싱하지 않는다. 이는 에러로 이어질 가능성이 높다. 제대로 의도한대로 동작하게 하려면 몇가지 디테일을 추가해줘야한다.

## 알맞은 순서로 태스크 실행하기

`dependsOn` 키는 특정 태스크가 실행되기 전에 어떤 태스크들을 먼저 실행되게 할지 지정할 때 사용하면 좋다. 만약 각 애플리케이션 들의 `build` 명령이 다른 라이브러리들의 `build` 명령어가 수행된 이후 실행되길 원한다면, 아래와 같이 설정해주면 된다.

```json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"] 
    }
  }
}
```

이렇게 해주면 의존성들의 `build`가 먼저 실행되게 된다. 하지만 여기까지만 설정해두면 빌드 아웃풋에 대한 설정을 한게 없기 때문에, 아무것도 캐싱되지 않는다.

## `^`를 통해 설정된 의존성

`^` 를 통해 의존성을 설정하면, Turborepo에게 의존성 그래프 최하단부터 실행하라고 알려주는 셈이다. 만약 애플리케이션이 `ui` 라이브러리에 의존하고 있고 해당 라이브러리에 `build` 태스크가 있다면, 해당 `build` 태스크가 끝난 후에 애플리케이션의 `build` 태스크가 실행된다.

이를 통해서 의존성의 `build` 부터 수행되게 한 뒤 실제 애플리케이션의 `build` 명령어가 실행되도록 할 수 있기에, 저장소가 커질수록 더욱 중요성이 커진다.

## 같은 패키지 내의 태스크 의존성

`dependsOn` 키에 일반 문자열을 넣으면, 같은 패키지 내의 다른 스크립트가 먼저 실행된 뒤 실행하도록 할 수 있다.

```json
{
  "tasks": {
    "test": {
      "dependsOn": ["build"] 
    }
  }
}
```

## 특정 패키지 내의 특정 태스크에 의존

```json
{
  "tasks": {
    "lint": {
      "dependsOn": ["utils#build"] 
    }
  }
}
```

위와 같이 설정하면 `utils` 패키지의 `build` 스크립트에 의존하게된다.

의존하고있는 태스크를 좀 더 자세히 명시할수도 있다.

```json
{
  "tasks": {
    "web#lint": {
      "dependsOn": ["utils#build"] 
    }
  }
}
```

위와 같이 설정하면 `web` 패키지의 `lint` 태스크는 `utils` 패키지의 `build` 태스크에 의존하게 된다.

## 의존성 없음

의존성이 없느느 태스크는 `dependsOn` 키를 생략하거나 빈 배열을 넘기면 된다.

```json
{
  "tasks": {
    "spell-check": {
      "dependsOn": [] 
    }
  }
}
```

## `outputs` 명시

<aside>
ℹ️

Turborepo는 태스크의 아웃풋을 캐싱해서 같은 일을 두번하지 않게 막아준다.

</aside>

`outputs` 키는 Turborepo에게 캐싱해야하는 파일이나 디렉토리를 알려준다. 이 키가 명시되지 않으면, Turborepo는 아무 파일도 캐싱하지 않을 것이다. 즉, 이후 실행시에 Cache Hit가 떠도 아무 아웃풋도 복구하지 않는다.

```json
{
  "tasks": {
    "build": {
      "outputs": [".next/**", "!.next/cache/**"] 
    }
  }
}
```

glob 패턴은 패키지에 상대적이다. 즉, `dist/**`는 각 패키지의 `dist` output을 모두 핸들링한다.

## `inputs` 명시

태스크의 캐시 해싱에 사용할 파일들을 `inputs` 키를 통해 명시할 수 있다. 디폴트로, Turborepo는 Git에 의해 트래킹되는 모든 파일들을 포함한다. 하지만 얼마든지 어떤 파일들을 포함할 지 추가로 `inputs` 키에 명시할 수 있다.

```json
{
  "tasks": {
    "spell-check": {
      "inputs": ["**/*.md", "**/*.mdx"] 
    }
  }
}
```

위처럼 태스크를 명시해주면, `md/mdx` 파일이 변경되었을때만 `spell-check` 태스크의 Cache Miss가 발생한다.

<aside>
ℹ️

이 기능을 사용하면 디폴트 `inputs` 동작이 취소된다. 즉, `.gitignore` 파일에 의해 무시되는 파일들도 `inputs`에 포함되게 된다. 만약 디플트 동작을 활용하고싶다면, `$TURBO_DEFAULT$` 마이크로신택스를 사용하면된다.

</aside>

## `$TURBO_DEFAULT$`로 디폴트 동작 복구하기

```json
{
  "tasks": {
    "build": {
      "inputs": ["$TURBO_DEFAULT$", "!README.md"] 
    }
  }
}
```

위와 같이 선언하면 디폴트 `inputs` 동작 + [`README.md`](http://README.md) 외의 파일들이 input이 된다.

## 루트 태스크 등록하기

루트에 있는 `package.json`에 선언된 스크립트 또한 `turbo`를 통해 실행할 수 있다. 예를들어, `lint:root` 태스크를 실행하고 싶다면 아래와 같이 선언해주면 된다.

```json
{
  "tasks": {
    "lint": {
      "dependsOn": ["^lint"]
    },
    "//#lint:root": {} 
  }
}
```

루트 태스크를 언제 사용할까?

- 워크스페이스 루트의 린팅/포매팅
- Turborepo로 점진적 마이그레이션을 하고 있을 때
- 패키지 스콥 외의 스크립트들

## 고급 사용 사례

### Package Configuration 사용하기

Package Configuration은 패키지 내에 위치하는 `turbo.json` 파일들이다. 이를 통해서 해당 패키지의 태스크에만 필요한 동작들을 저장소의 다른 부분에 영향을 끼치지 않고 정의할 수 있다.

### 런타임 의존성을 가지고 있는, 실행시간이 긴 태스크들

항상 함께 실행되어야 하는 태스크가 있는 태스크가 있을 수 있다. 이때는 `with` 키를 사용한다.

```json
{
  "tasks": {
    "dev": {
      "with": ["api#dev"],
      "persistent": true,
      "cache": false
    }
  }
}

```

### 사이드 이펙트 수행하기

어떤 태스크는 어떤 조건에서든 실행되어야 할 수 있다. 이때는 `cache: false`로 설정해준다.

```json
{
  "tasks": {
    "deploy": {
      "dependsOn": ["^build"],
      "cache": false
    },
    "build": {
      "outputs": ["dist/**"]
    }
  }
}
```

### 병렬로 실행할 수 있는 의존성 태스크

다른 패키지에 의존하고 있지만 병렬로 실행할 수 있는 태스크가 있을 수 있다. 예를들면 linter가 있을 수 있다. 그래서 아래와 같이 의존성을 선언하고 싶을 수 있는데..

```json
{
  "tasks": {
    "check-types": {} // Incorrect!
  }
}
```

이렇게 하면 태스크가 병렬로 실행되지만 의존성의 소스코드 변화를 인식하지 못한다. 의도치 않게 Cache Hit가 발생하여 linting이 제대로 되지 않을 수 있다. 그래서 아래와 같이 선언하면..

```json
{
  "tasks": {
    "check-types": {
      "dependsOn": ["^check-types"] // This works...but could be faster!
    }
  }
}
```

의도한대로 linting이 실행되지만 태스크가 병렬로 실행되지 않는다. 두 요구사항을 모두 만족시키려면, Transit Node를 활용해야한다.

```json
{
  "tasks": {
    "transit": {
      "dependsOn": ["^transit"]
    },
    "check-types": {
      "dependsOn": ["transit"]
    }
  }
}

```

위처럼 선언해주면 `check-types` 태스크는 `transit` 태스크에 의존하고 있고, 이 `transit` 태스크가 내부 의존성들의 `transit` 태스크에 의존하고 있기 때문에 병렬로 태스크가 실행되도록 할 수 있다.