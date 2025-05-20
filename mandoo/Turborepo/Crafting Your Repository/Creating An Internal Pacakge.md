내부 패키지는 워크스페이스의 빌딩블럭으로써, 저장소 내에서 여러가지 코드와 기능들을 공유할 수 있는 강력한 방법을 제공한다. Turborepo는 자동으로 내부 패키지들간의 관계를 `package.json`을 통해서 파악하고, 내부적으로 패키지 그래프를 만들어서 저장소의 워크플로우를 최적화할 방법을 찾아낸다.

## 만드는 법

1. 빈 디렉토리 생성 (`./packages/math`)
2. `package.json` 파일 추가
    
    ```json
    {
      "name": "@repo/math",
      "type": "module",
      "scripts": {
        "dev": "tsc --watch",
        "build": "tsc"
      },
      "exports": {
        "./add": {
          "types": "./src/add.ts",
          "default": "./dist/add.js"
        },
        "./subtract": {
          "types": "./src/subtract.ts",
          "default": "./dist/subtract.js"
        }
      },
      "devDependencies": {
        "@repo/typescript-config": "workspace:*",
        "typescript": "latest"
      }
    }
    ```
    
    - `exports` → 이 패키지의 엔트리포인트를 여러개 지정할 수 있음, 다른 패키지들이 이걸 통해 접근
    - `devDependencies`의 `@repo/typescript-config` → 다른 내부 패키지인 `typescript-config`를 의존성으로 가지고 있음 → 이걸 통해 Turborepo가 의존성 파악
3. `tsconfig.json` 파일 추가
    
    ```json
    {
      "extends": "@repo/typescript-config/base.json",
      "compilerOptions": {
        "outDir": "dist",
        "rootDir": "src"
      },
      "include": ["src"],
      "exclude": ["node_modules", "dist"]
    }
    ```
    
    - 앞서 `devDependencies`로 지정해둔 `typescript-config` 패키지의 config extend
    - `outDir` → 컴파일된 결과물을 둘 곳
    - `rootDir` → `outDir`가 `src` 폴더와 똑같은 구조를 가지도록 설정
    - `include` + `exclude` → `extends`를 통해서 확장 안되기 때문에, 지정해줌
4. `src` 디렉토리에 소스코드 추가
    
    ```tsx
    export const add = (a: number, b: number) => a + b;
    ```
    
5. 애플리케이션에 패키지 추가
    
    ```json
      "dependencies": {
    +   "@repo/math": "workspace:*",
        "next": "latest",
        "react": "latest",
        "react-dom": "latest"
      },
    ```
    
    - `web` 의 `package.json`에 위처럼 세팅해주면, 이제 코드 내에서 `import`해서 사용할 수 있게 됨
    
    ```tsx
    import { add } from '@repo/math/add';
     
    function Page() {
      return <div>{add(1, 2)}</div>;
    }
     
    export default Page;
    ```
    
6. `turbo.json` 수정
    1. `@repo/math` 라이브러리의 아티팩트를 `turbo.json`의 `build` 태스크의 `outputs`에 추가해준다. 이렇게 하면 빌드 아웃풋이 Turborepo에 의해 캐싱될 수 있음
    
    ```tsx
    {
      "tasks": {
        "build": {
          "dependsOn": ["^build"],
          "outputs": [".next/**", "!.next/cache/**", "dist/**"]
        }
      }
    }
    ```
    
7. `turbo build` 실행

## 내부 패키지 Best Practice

### 1 패키지, 1 목적

내부 패키지를 만들때는, 패키지가 하나의 “목적”을 가져야함. 이것이 엄격한 규칙이라고 할 수는 없지만, 강력히 권장되는 사항임. 이렇게 했을때 얻는 이점은..

- 이해하기 쉽다
    - 저장소가 커짐에 따라, 작업하는 개발자들이 더 쉽게 코드를 찾을 수 있음
- 패키지 별 의존성 줄이기
    - 각 패키지 별 의존성이 적을 수록 Turborepo가 더 효과적으로 패키지 그래프의 의존성을 줄일 수 있음

### 애플리케이션 패키지는 공유 코드를 포함하지 않음

- 애플리케이션 패키지를 만들 때, 패키지 내에 공유 코드를 두는 것을 피해야 함.
    - 대신, 공유 코드에 대한 별도의 패키지를 만들고 애플리케이션 패키지가 그 패키지에 의존하도록 하는 것이 좋음
- 애플리케이션 패키지는 다른 패키지에 설치되어 사용하는 것으로 의도되지 않았음. 대신, Package Graph의 entrypoint로 취급되어야 함