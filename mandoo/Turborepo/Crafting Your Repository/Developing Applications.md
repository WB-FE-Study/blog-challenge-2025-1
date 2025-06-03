대부분 dev 태스크는 코드의 변경사항을 추적하는, 오래걸리는 태스크들이다. Turborepo는 TUI와 다른 기능들을 통해 이를 지원한다.

## 개발 태스크 구성하기

`turbo.json`에 dev 태스크를 정의한다는 것은 오랜기간 실행될 태스크가 있다는 것을 알리는 것과 같다. 예시로 Dev 서버 실행, 테스트 실행, 애플리케이션 빌드 등이 있다.

두개의 프로퍼티와 함께 태스크를 정의해주면 된다.

```json
{
  "tasks": {
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

- `cache: false`
    - 이 태스크의 결과물을 캐싱하지 말라는 뜻
    - 변경사항이 자주 발생할 것 이고, 캐싱이 큰 효과 없을 것이다
- `persistent: true`
    - 직접 멈출때까지 태스크를 실행하라는 뜻
    - TUI에게 태스크를 오래 실행되고 상호작용 가능한 형태로 실행되어야 함을 알려줌

### `dev` 이전에 셋업 태스크 실행하기

개발 환경 세팅이나 패키지 프리빌드 등의 과정이 필요하다면, `dependsOn`을 통해서 명시해주자.

```json
{
  "tasks": {
    "dev": {
      "cache": false,
      "persistent": true,
      "dependsOn": ["//#dev:setup"]
    },
    "//#dev:setup": {
      "outputs": [".codegen/**"]
    }
  }
}
```

### 특정 애플리케이션만 실행하기

`--filter` 플래그를 사용하면 패키지 그래프에서 특정 태스크 부분집합만 실행할 수 있다.

```bash
turbo dev --filter=web
```

## TUI 사용하기

### 뷰 커스터마이즈

키보드를 통해 빠르게 UI를 탐색할 수 있다.

| **Keybind** | **Action** |
| --- | --- |
| `m` | Toggle popup listing keybinds |
| `↑`/`↓` | Select the next/previous task in the task list |
| `j`/`k` | Select the next/previous task in the task list |
| `p` | Toggle selection pinning for selected task |
| `h` | Toggle visibility of the task list |
| `c` | When logs are highlighted, copy selection to the system clipboard |
| `u`/`d` | Scroll logs `u`p and `d`own |

### 태스크와 상호작용하기

상호작용 가능함으로 마크된 태스크와 상호작용할 수 있다.

| **Keybind** | **Action** |
| --- | --- |
| `i` | Begin interacting |
| `Ctrl+z` | Stop interacting |

## Watch 모드

많은 도구들은 빌트인 watcher를 가지고 있으며, 소스코드의 변경사항에 자동으로 반응할 것이다. 하지만 없는 도구들도 분명 있다.

`turbo watch`를 사용하면 아무 도구든 상관없이 의존성을 감시하는 감시자를 추가해준다. 소스코드의 변경사항은 `turbo.json`에서 정의된 태스크 그래프를 따라서 전파된다.

```json
{
  "tasks": {
    "dev": {
      "persistent": true,
      "cache": false
    },
    "lint": {
      "dependsOn": ["^lint"]
    }
  }
}

{
  "name": "@repo/ui"
  "scripts": {
    "dev": "tsc --watch",
    "lint": "eslint ."
  }
}

{
  "name": "web"
  "scripts": {
    "dev": "next dev",
    "lint": "eslint ."
  },
  "dependencies": {
      "@repo/ui": "workspace:*"
    }
}
```

위의 예시에서 `lint` 태스크는 소스코드 변경사항이 있을때마다 필요한 패키지의 `lint` 태스크를 실행해줄 것이고, `dev` 태스크는 persistent로 지정되어있기에 재실행되지 않을 것이다.

## 한계

### Teardown Tasks

`dev` 태스크가 끝날때 특정 스크립트를 실행하고 싶을 수 있다. 하지만, Turborepo는 이를 할 수 없다.

대신, `turbo dev:teardown` 스크립트를 만들고 직접 실행해주는 방법을 사용해야한다.
