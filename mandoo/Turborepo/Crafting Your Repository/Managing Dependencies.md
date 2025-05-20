- **외부 의존성**은 NPM 레지스트리에서 오는 녀석들이다.
- **내부 의존성**은 저장소 내에서 기능을 공유할 수 있게 해준다.

```tsx
{
  "dependencies": {
    "next": "latest", // External dependency
    "@repo/ui": "workspace:*" // Internal dependency
  }
}
```

## 의존성 설치에 관한 Best Practice

### 필요한 곳에 의존성 설치하기

저장소 내에서 의존성을 설치할 때, 해당 의존성을 필요로하는 패키지 내에서 설치해야한다. 이는 외부/내부 의존성 모두에 해당하는 사실이다.

만약 하나의 의존성을 여러 패키지에 동시에 설치하고자 한다면, 아래와 같은 방법을 사용할수도 있다.

```bash
pnpm install jest --save-dev --recursive --filter=web --filter=@repo/ui --filter=@repo/web
```

이와 같은 사용은 몇가지 장점을 가지고 있다.

- 향상된 명확성
    - 의존성은 각 패키지의 `package.json`에 있으면 더 이해하기 쉽다. 개발자들은 어떤 패키지가 어떤 의존성을 사용하고 있는지 명확하게 파악할 수 있다.
- 향상된 유연성
    - 각 패키지가 모두 똑같은 버전의 외부 의존성을 사용하는 것은 비현실적이다.
    - 의존성을 사용하는 패키지에 설치함으로써, 각 패키지가 각각 버전관리를 할 수 있다. 원한다면 [버전을 통일하는 것도 가능](https://turbo.build/docs/crafting-your-repository/managing-dependencies#keeping-dependencies-on-the-same-version)하다.
- 더 나은 캐싱 능력
    - 저장소의 루트에 너무 많은 의존성을 설치하면, 워크스페이스 루트를 의존성을 추가/수정/삭제할때마다 캐시 미스가 발생하게 된다.
- 사용하지 않는 의존성 가지치기
    - 도커 사용자들은 Turborepo의 pruning 기능을 사용하여 도커 이미지에서 사용하지 않는 의존성을 삭제해 더 가벼운 이미지를 만들 수 있다.
    - 의존성들이 있어야하는 패키지에 설치되면, Turborepo가 락파일을 읽고 각 패키지에서 사용하지 않는 의존성을 삭제해줄 수 있다

### 루트에 적은 의존성 두기

루트에 있어야 하는 의존성은 저장소를 관리하는 도구들 뿐이다. 예를 들면 `turbo`, `husky`, `lint-staged` 등이 있을 수 있다.

## 의존성 관리하기

### Turborepo는 의존성을 관리하지 않는다

Turborepo는 의존성 관리에 어떤 역할도 하지 않는다. 당신이 선택한 패키지 매니저가 할 일이다.

어떤 외부 의존성 버전을 설치하고 symlink하고 resolve하는지는 패키지 매니저의 동작을 따라간다. 아래의 권장사항들은 Workspace에서 의존성을 관리하는 것에 관한 것이지, Turborepo에 의해 강제되지 않는다.

### 패키지 매니저에 따라 다른 Module Resolution 방법

패키지 매니저들은 각자 다른 module resolution 알고리즘을 가지고 있으며, 이로 인해 예측하기 어려운 동작상의 차이가 있을 수 있다.

Turborepo의 문서에서 패키지 매니저 별로 권장사항을 줄 것이지만, 이는 best effort일 뿐이며 저장소의 필요성에 맞게 적절한 패키지 매니저의 문서를 더 잘 읽어봐야할 수 있다.

### node_modules 위치

패키지 매니저/버전/세팅 + 워크스페이스의 의존성이 어디에 설치되어있냐에 따라서, `node_modules`와 의존성들이 워크스페이스 내의 여러곳에 위치할수도 있다. 루트의 `node_modules`, 패키지들의 `node_modules` 등 여러개가 있을 수 있다.

스크립트와 태스크들이 필요한 의존성을 잘 찾아쓰고 있다면, 패키지매니저가 올바르게 동작하고 있는 것이다.

### 의존성을 같은 버전으로 두기

몇몇 모노리포 관리자들은 모든 패키지의 모든 의존성을 같은 버전으로 두도록 강제하고 싶어한다. 방법이 몇가지 있다.

- purpose-built tooling 사용
    - `syncpack`, `manypckg`, `sherif` 등을 사용할 수 있다
- 패키지 매니저 사용
    
    ```bash
    pnpm up --recursive typescript@latest
    ```
    
- IDE 사용
    - regex 등을 이용해서 의존성 버전을 싹 수정한 다음, install 커맨드를 때린다