태스크가 `turbo.json`에 등록되면, 저장소에서 스크립트를 실행할 수 있는 강력한 방법을 얻은 것과 같다.

- 자주 실행하는 태스크를 `package.json`의 `scripts`로 사용
- 전역 `turbo` 명령을 사용해서 빠르게 커스텀 태스크 실행하기
- 디렉토리/패키지이름/소스컨트롤 변경사항 등으로 태스크 필터링하기

`turbo`를 통해 태스크를 실행하게되면, dev 환경, CI 파이프라인 등 워크플로우를 실행하는 하나의 모델이 생기는 것이기 떄문에 아주 강력하다.

## `package.json` 내의 `scripts` 사용

자주 실행하는 태스크는 루트 `package.json`의 `scripts`로 등록해준다.

```json
{
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "test": "turbo run test",
    "lint": "turbo run lint"
  }
}
```

<aside>
ℹ️

`turbo`는 `turbo run`의 별칭으로 사용할 수 있지만, 추후에 서브커맨드들이 추가될 것을 고려하여 `turbo run`을 사용하는 것이 권장된다. 

</aside>

## 전역 `turbo` 사용하기

전역으로 `turbo`를 설치하면, 터미널에서 바로 명령어를 실행할 수 있게된다. 이를 통해 로컬 개발 경험을 더 좋게 만들 수 있다.

CI 파이프라인에서도 편리함이 더해지는데, 원하는 시기에 원하는 정확한 태스크를 실행할 수 있게되기 때문이다.

### 자동 패키지 스코핑

현재 패키지 디렉토리 내에 있다면, `turbo`는 자동으로 패키지 그래프를 통해 커맨드 스코프를 정한다. 이를 통해서 패키지를 위한 필터를 작성할 필요없이 빠르게 커맨드를 실행할 수 있게된다.

```bash
cd apps/docs
turbo build
```

위와 같이 명령어를 입력하면, `docs` 패키지의 `build` 태스크를 실행하게된다.

<aside>
ℹ️

만약 필터를 사용하게 되면, 자동 패키지 스코핑을 오버라이드하게된다.

</aside>

### 동작 커스터마이징

`turbo run`의 동작을 원하는대로 커스터마이징할 수 있는 옵션이 많이 있다. 전역 `turbo`를 실행할 때, 더 빠르게 워크플로우를 사용할 수 있다.

- 자주쓰는 커맨드 베리에이션
    - `turbo build` → `turbo build --filter=@repo/ui`
- 한번쓰고 마는 커맨드
    - `turbo build--dry` 같은 명령어는 굳이 `package.json`에 등록하고 사용할 필요 없다. 대신 터미널에서 그냥 바로 사용하면된다.
- `turbo.json` 컨피그 오버라이딩
    - `turbo` 로 명령어 실행시, 추가로 플래그를 붙여주면 오버라이드를 할 수 있다.

## 여러 태스크 실행하기

`turbo`를 통해 여러 태스크를 실행할 수 있다. 병렬화도 알아서 한다.

```bash
turbo run build test lint check-types
```

위 커맨드는 모든 태스크를 실행하고, task definition에 따라 가능한 빠르게 각 태스크를 실행시킨다.

<aside>
ℹ️

`turbo test lint` will run tasks exactly the same as `turbo lint test`.
If you want to ensure that one task blocks the execution of another, express that relationship in your [task configurations](https://turborepo.com/docs/crafting-your-repository/configuring-tasks#defining-tasks).

</aside>

## 필터 사용하기

`--filter` 를 통해서 태스크에 대한 다양한 필터링을 할 수 있다.

### 패키지를 기준으로 필터링

```bash
turbo build --filter=@acme/web
```

`--filter`를 안쓰고 원하는 패키지의 원하는 태스크만 실행할 수도 있다.

```bash
# Run the `build` task for the `web` package
turbo run web#build
 
# Run the `build` task for the `web` package, and the `lint` task for the `docs` package
turbo run web#build docs#lint
```

### 디렉토리를 기준으로 필터링

연관된 패키지들이 그루핑된 상태일 수 있다. 이 경우에는 glob 패턴을 통해서 해당 패키지들을 지정할 수 있다.

```bash
turbo lint --filter="./packages/utilities/*"
```

### 피의존성을 포함하여 필터링

특정 패키지 내에서 작업중일 때, 해당 패키지와 해당 패키지에 의존하고 있는 것들에 대해서 태스크를 실행하고 싶을 수 있다. `...` 마이크로신택스를 이때 사용하면된다.

```bash
turbo build --filter=...ui
```

### 의존성을 포함하여 필터링

패키지와 그 의존성들로 스코프를 제한하고싶다면, `...`를 패키지 이름 뒤에 붙여준다. 그러면 해당 패키지와 그 패키지의 의존성들에 대해 명령어가 실행된다.

```bash
turbo dev --filter=web...
```

### 소스컨트롤 변경사항에 따라 필터링하기

변경사항이 있을때만 태스크를 실행하려고 한다면, 소스컨트롤 필터를 사용하면된다. 소스컨트롤 필터는 `[]`로 감싸져야한다.

- 이전 커밋과 비교 : `turbo build --filter=[HEAD^1]`
- 메인 브랜치와 비교 : `turbo build --filter=[main...my-feature]`
- SHA를 통해 특정 커밋과 비교 : `turbo build --filter=[a1b2c3d...e4f5g6h]`
- 브랜치 이름을 통해서 특정 커밋과 비교 : `turbo build --filter=[your-feature...my-feature]`

### 필터 섞기

더 상세하게 필터를 설정하기 위해, 필터를 섞어서 쓸 수 있다.

```bash
turbo build --filter=...ui --filter={./packages/*} --filter=[HEAD^1]
```

이렇게 여러 필터를 선언하면, 합집합으로 엮인다. 즉, 필터중 하나로도 매칭되는게 있다면 태스크가 실행된다는 뜻이다.