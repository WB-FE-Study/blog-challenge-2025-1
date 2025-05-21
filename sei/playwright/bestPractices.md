# Best Practices

[Best Practices | Playwright](https://playwright.dev/docs/best-practices)

## 테스트 철학

자동화 테스트는 애플리케이션 코드가 **최종 사용자 입장에서 제대로 동작하는지 검증**해야 하며, 함수 이름, 배열 여부, 특정 요소의 CSS 클래스처럼 **사용자가 일반적으로 사용하지 않거나 보지 못하거나 알지 못하는 구현 세부사항에 의존해서는 안 됩니다**. 최종 사용자는 화면에 렌더링된 것을 보고 상호작용하므로, 테스트도 **렌더링된 결과만 보고 상호작용하도록 작성하는 것이 이상적**입니다.

## 테스트는 가능한 독립적으로

각 테스트는 서로 완전히 독립적으로 동작해야 하며, 자신만의 로컬 스토리지, 세션 스토리지, 데이터, 쿠키 등을 사용해야 합니다. 테스트의 격리는 재현 가능성을 높이고, 디버깅을 쉽게 만들며, 연쇄적인 테스트 실패를 방지하는 데 도움이 됩니다.

반복되는 테스트 구문을 줄이기 위해 `before`및 `after` 훅을 사용할 수 있습니다. 테스트 파일 내에서 `before` 훅을 추가하면, 각 테스트 전에 공통 작업을 실행할 수 있습니다. 이렇게 하면 각 테스트가 다른 테스트에 의존하지 않고 독립적으로 동작할 수 있습니다.

다만 테스트가 충분히 단순한 경우엔, 코드를 약간 중복하더라도 테스트의 가독성과 유지보수성을 높일 수 있습니다.

```tsx
import { test } from '@playwright/test';

test.beforeEach(async ({ page }) => {
  // Runs before each test and signs in each page.
  await page.goto('https://github.com/login');
  await page.getByLabel('Username or email address').fill('username');
  await page.getByLabel('Password').fill('password');
  await page.getByRole('button', { name: 'Sign in' }).click();
});

test('first', async ({ page }) => {
  // page is signed in.
});

test('second', async ({ page }) => {
  // page is signed in.
});
```

## 써드 파티 의존성을 피하세요

자신이 통제할 수 있는 것만 테스트하세요. 외부 사이트나 제3자 서버로 연결되는 링크처럼 직접 통제할 수 없는 요소를 테스트하려고 하지 마세요. 이런 테스트는 시간이 오래 걸리고 테스트 속도를 저하시킬뿐만 아니라, 링크된 페이지의 콘텐츠나 쿠키 배너, 오버레이 페이지 등으로 인해 예상치 못한 테스트 실패가 발생할 수 있습니다.

대신, **Playwright의 Network API**를 활용해 필요한 응답을 보장하는 방식으로 처리하세요.

```tsx
await page.route('**/api/fetch_data_third_party_dependency', route => route.fulfill({
  status: 200,
  body: testData,
}));
await page.goto('https://example.com');
```

## Database를 사용하는 테스트

데이터베이스와 함께 테스트를 진행할 경우, 테스트에서 사용하는 데이터를 반드시 통제할 수 있어야 합니다.

변경되지 않는 스테이징 환경에서 테스트를 수행하도록 하며, 외부 요인에 의해 데이터가 바뀌지 않도록 주의하세요.

또한 시각적 회귀 테스트를 수행할 경우, 운영 체제와 브라우저 버전이 동일해야 일관된 결과를 얻을 수 있습니다.

## Best Practices

## Use locators

e2e 테스트를 작성하려면 먼저 웹 페이지에서 요소를 찾아야 합니다. 이를 위해 Playwright의 내장 로케이터를 사용할 수 있습니다.

로케이터는 자동 대기와 재시도 가능을 기본적으로 제공합니다.

자동 대기란 Playwright가 클릭 등의 동작을 수행하기 전에, **해당 요소가 화면에 보이고 활성화되어 있는지 등 다양한 조건을 자동으로 확인**한다는 의미입니다.

테스트의 견고함과 유지보수성을 높이기 위해 “사용자에게 보이는 속성”이나 “명시적인 contracts”를 우선적으로 사용하는 것이 좋습니다.

```tsx
page.getByRole('button', { name: 'submit' });
```

## Use chaining and flitering

로케이터는 체이닝(chaining)을 통해 페이지의 특정 영역으로 탐색 범위를 좁힐 수 있습니다.

```tsx
const product = page.getByRole('listitem').filter({ hasText: 'Product 2' });
```

또한, 텍스트나 다른 로케이터를 기준으로 필터링할 수도 있습니다.

```tsx
await page
  .getByRole('listitem')
  .filter({ hasText: 'Product 2' })
  .getByRole('button', { name: 'Add to cart' })
  .click();
```

이렇게 하면 특정 리스트 항목 안에 있는 ‘Add to cart’ 버튼만 정확히 클릭할 수 있어, 테스트의 정확성과 안정성을 높일 수 있습니다.

## XPath나 CSS 선택자보다 사용자에게 보이는 속성을 우선적으로 사용하세요

DOM 구조는 쉽게 변경될 수 있기 때문에, DOM 구조에 의존하는 테스트는 실패할 가능성이 높습니다. 예를 들어, 아래와 같이 CSS 클래스를 기반으로 버튼을 선택하는 경우를 생각해보세요. 디자이너가 스타일을 수정하면 클래스명이 변경되어 테스트가 깨질 수 있습니다.

```tsx
// 👎 좋지 않은 예시
page.locator('button.buttonIcon.episode-actions-later');
// 👍 좋은 예시
page.getByRole('button', { name: 'submit' });
```

이처럼 역할(role)과 이름(name) 같은 사용자 중심 속성을 사용하면 테스트가 더 안정적이고 유지보수하기 쉬워집니다.

## locator 생성하기

Playwright에는 test generator가 내장되어 있어, 테스트 코드를 자동으로 생성하고 로케이터도 자동으로 선택해줍니다.

이 생성기는 페이지를 분석하여 역할(role), 텍스트(text), test ID 등을 우선적으로 활용해 가장 적합한 로케이터를 찾아줍니다.

로케이터와 일치하는 요소가 여러 개 있을 경우, Playwright는 더 정교한 로케이터를 생성하여 대상 요소를 고유하게 식별할 수 있도록 개선하므로, 로케이터 때문에 테스트가 실패할 걱정을 줄일 수 있습니다.

## `codegen`으로 로케이터 생성하기

특정 페이지에서 로케이터를 자동으로 생성하고 싶다면, 아래와 같이 codegen 명령어 뒤에 URL을 입력하여 실행하세요.:

```bash
npx playwright codegen https://example.com
```

이후 브라우저가 열리고 상호작용을 하면, 해당 동작에 맞는 테스트 코드와 로케이터가 자동으로 생성됩니다.

```bash
pnpm exec playwright codegen playwright.dev
```

위 명령어를 실행하면 새 브라우저 창과 함께 Playwright 인스펙터가 열립니다.

로케이터를 선택하려면 먼저 Record 버튼을 클릭해 녹화를 중지해야 합니다. `codegen` 명령어는 기본적으로 실행 시 자동으로 녹화를 시작하기 때문에, 로케이터 선택 기능을 사용하려면 녹화를 멈춰야 합니다.

녹화를 중지하면 ‘Pick Locator’ 버튼이 활성화되어 클릭할 수 있습니다.

그 후 브라우저 창에서 원하는 요소 위에 마우스를 올리면, 커서 아래에 해당 요소의 로케이터가 하이라이트되어 나타납니다.

요소를 클릭하면 해당 로케이터가 Playwright 인스펙터에 추가됩니다.

이 로케이터는

- 복사해서 테스트 파일에 붙여넣거나,
- 인스펙터에서 텍스트 등을 수정해 로케이터를 탐색/조정할 수도 있고,
- 수정 결과는 브라우저 창에 바로 반영되어 확인할 수 있습니다.

![image.png](attachment:320b5319-7a69-459f-9cfd-117c5e4fde8f:image.png)

## **VS Code extension으로 locator 생성하기**

VS Code Extension으로 테스트를 기록하고 locator를 생성할 수도 있습니다. VS Code 확장 Extension은 개발자가 테스트를 작성하고, 실행하고, 디버깅하는 데 훌륭한 개발자 경험을 제공합니다.

![image.png](attachment:e83bbfad-a883-4389-b8de-f1f4aca07f2d:image.png)

## Web First Assertion을 사용하세요

Assertion(단언)은 기대한 결과와 실제 결과가 일치하는지 확인한 방법입니다.

Playwright에서 제공하는 Web First Assertion을 사용하면, **기대한 조건이 충족될 때까지 자동으로 기다려줍니다.**

예를 들어, 어떤 버튼을 클릭한 뒤 알림 메시지가 나타나는 경우, 메시지가 0.5초 후에 나타나더라도 `toBeVisible()` 같은 Web First Assertion은 기다렸다가 조건이 만족되면 통과합니다.

```bash
// 👍 좋은 예시
await expect(page.getByText('welcome')).toBeVisible();

// 👎 나쁜 예시 - 수동으로 대기 없이 바로 확인
expect(await page.getByText('welcome').isVisible()).toBe(true);
```

## 수동 assertion을 피하세요

아래와 같이 **`await`가 `expect` 안에 들어간 수동 방식**은 사용하지 마세요.

이 경우 Playwright는 **기다리지 않고 바로 로케이터 상태를 확인**하고, 그 순간 조건이 충족되지 않으면 테스트는 실패합니다.

```bash
// 👎 좋지 않은 예시
expect(await page.getByText('welcome').isVisible()).toBe(true);

// 👍 권장 방식
await expect(page.getByText('welcome')).toBeVisible();
```

이 방식이 테스트를 **더 안정적이고 flaky하지 않게** 만들어줍니다.

## Debugging 설정

## Local debugging

로컬 디버깅을 위해서는 VSCode에서 테스트를 실시간으로 디버깅하는 것을 권장합니다.

이를 위해 Playwright 전용 VSCode 확장 프로그램을 설치하세요.

테스트를 디버그 모드로 실행하려면, 디버깅하고 싶은 테스트 코드 옆 줄 번호를 우클릭한 후 “Debug Test”를 선택하면 됩니다.

이렇게 하면 브라우저 창이 열리고, 브레이크포인트가 설정된 위치에서 실행이 일시 중지되어 디버깅을 진행할 수 있습니다.

![image.png](attachment:8d26cded-be37-4816-b2b5-d43ce91d4dee:image.png)

VS Code에서 테스트 코드의 **로케이터를 클릭하거나 수정하면**, 해당 로케이터가 **브라우저 창에서 하이라이트**되며,

페이지 내에서 **일치하는 다른 로케이터들도 함께 표시**되어 테스트를 **실시간으로 디버깅**할 수 있습니다.

![image.png](attachment:db9dc779-c91c-4b4c-be22-20b5fadcb8c1:image.png)

`-debug` 플래그를 사용하여 테스트를 실행하면, **Playwright Inspector를 통해 테스트를 디버깅할 수도 있습니다.**

```bash
pnpm exec playwright test --debug
```

그 후에는 테스트를 한 단계씩 실행(step through)하면서, 동작 가능성 로그(actionability logs)를 확인하고, **로케이터를 실시간으로 수정**해볼 수 있습니다.

수정한 로케이터는 **브라우저 창에서 하이라이트**되며, **일치하는 요소가 있는지, 몇 개인지도 시각적으로 확인**할 수 있습니다.

![image.png](attachment:6c7528a4-f0ad-402a-9a7b-3a03aff6e3aa:image.png)

특정 테스트를 디버깅하려면 `--debug` 플래그 뒤에 테스트 파일의 이름과 라인 넘버를 추가합니다.

```bash
pnpm exec playwright test example.spec.ts:9 --debug
```

## CI Debugging

CI 실패한다면, 비디오나 스크린샷 대신 Playwright [trace viewer](https://playwright.dev/docs/trace-viewer)를 사용하세요. Trace Viewer는 테스트의 전체 실행 과정을 기록한 **로컬 PWA(Progressive Web App)** 형태로 제공되며, **쉽게 공유할 수 있습니다.**

Trace Viewer를 사용하면, 테스트 실행 **타임라인 시각화,** 각 동작에 대한 **DOM 스냅샷 확인** (개발자 도구처럼 검사 가능), **네트워크 요청 내역 과 상세 실행 정보를 확인할 수 있습니다.**

![image.png](attachment:c48aca15-9261-42bd-8bec-ed187263dcc0:image.png)

Trace는 Playwright 설정 파일에서 구성되며, CI 환경에서는 테스트가 실패했을 때 첫 번째 재시도에서 자동으로 trace가 기록되도록 설정되어 있습니다.

모든 테스트에서 trace를 항상 활성화하는 것은 성능에 큰 부담이 되기 때문에 권장하지 않습니다. 하지만 로컬 개발 중에는 다음과 같이 `--trace` 플래그를 사용하여 직접 trace를 실행할 수 있습니다:

```bash
npx playwright test --trace=on
```

이렇게 하면 테스트 실행 중의 **세부적인 실행 과정과 상태를 시각적으로 분석**할 수 있어, 문제 해결에 큰 도움이 됩니다.

```bash
pnpm exec playwright show-report
```

![image.png](attachment:0ec8dec8-84bb-4aee-ae72-23ffa06d5da9:image.png)

Traces는 테스트 파일 이름 옆에 있는 아이콘을 클릭하거나, 각 테스트의 리포터를 열고, 아래로 스크롤하면 있는 Traces 섹션에서 열 수 있습니다.

![image.png](attachment:5293a4bd-fa53-4568-8f55-2401370c794b:image.png)

## Playwright 툴 사용하기

Playwright는 테스트 작성 시 다양한 툴을 사용할 수 있습니다.

- [**VS Code 확장 프로그램**](https://playwright.dev/docs/getting-started-vscode)은 테스트를 작성하고 실행하며 디버깅하는 데 훌륭한 개발자 경험을 제공합니다.
- [Test generator](https://playwright.dev/docs/codegen)는 테스트를 자동으로 생성하고 로케이터를 선택해줍니다.
- [Trace Viewer](https://playwright.dev/docs/trace-viewer)는 테스트의 전체 실행 과정을 로컬 PWA로 제공하며 쉽게 공유할 수 있습니다. Trace Viewer를 통해 타임라인 확인, 각 동작에 대한 DOM 스냅샷 검사, 네트워크 요청 확인 등 다양한 기능을 사용할 수 있습니다.
- [UI Mode](https://playwright.dev/docs/test-ui-mode)는 테스트를 탐색하고 실행하며 디버깅할 수 있는 **타임 트래블 경험**과 **watch 모드**를 함께 제공합니다. 모든 테스트 파일은 사이드바에 로드되며, 각 파일과 `describe`블록을 확장해 개별 테스트를 실행, 확인, 관찰, 디버깅할 수 있습니다.
- Playwright의 [TypeScript](https://playwright.dev/docs/test-typescript)는 별도의 설정 없이 바로 사용할 수 있으며, 더 나은 IDE 통합 기능을 제공합니다. IDE는 사용할 수 있는 기능을 모두 보여주고, 잘못된 부분도 바로 알려줍니다. TypeScript 경험이 없어도 괜찮고, 코드 전체를 TypeScript로 작성할 필요도 없습니다. 단지 `.ts` 확장자로 테스트 파일을 만들기만 하면 됩니다.

## 모든 브라우저에서 테스트하기

Playwright를 사용하면 **어떤 플랫폼을 사용하든 모든 브라우저에서 사이트를 쉽게 테스트**할 수 있습니다.

브라우저 전반에 걸쳐 테스트를 수행하면, **모든 사용자가 앱을 문제없이 사용할 수 있는지 확인**할 수 있습니다.

설정 파일(config)에서 **프로젝트 이름과 사용할 브라우저 또는 디바이스를 지정**하여 다양한 환경에서 테스트를 설정할 수 있습니다.

```tsx
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],
});
```

## Playwright 의존성을 최신 상태로 유지하세요

Playwright 버전을 **최신 상태로 유지하면**, 앱을 **최신 브라우저 버전에서 미리 테스트**할 수 있고, 브라우저가 실제로 공개되기 전에 발생할 수 있는 문제를 **사전에 발견**할 수 있습니다.

```bash
pnpm install --save-dev @playwright/test@latest
```

최신 버전과 변경 사항은 [릴리스 노트(Release Notes)](https://playwright.dev/docs/release-notes)를 참고하세요.

현재 설치된 Playwright 버전은 아래 명령어로 확인할 수 있습니다:

```bash
pnpm exec playwright --version
```

## CI에서 테스트 실행하기

**CI/CD를 설정하고 테스트를 자주 실행하세요.** 테스트는 **자주 실행할수록 좋습니다.** 이상적으로는 **모든 커밋과 풀 리퀘스트마다 테스트를 실행**해야 합니다.

Playwright는 [**GitHub Actions용 워크플로우](https://playwright.dev/docs/ci-intro)가 기본 제공**되므로, 별도의 설정 없이도 CI에서 테스트를 자동으로 실행할 수 있습니다.

또한, Playwright는 원하는 [**다양한 CI 환경](https://playwright.dev/docs/ci)에서도 설정**할 수 있습니다.

CI에서는 Linux 환경 사용을 권장합니다. 비용이 저렴하고 실행 속도도 빠릅니다. 개발자는 로컬에서 원하는 환경을 사용해도 되지만, **CI에서는 Linux를 표준으로 사용**하세요. CI 속도를 높이기 위해 [**Sharding(분산 실행)**](https://playwright.dev/docs/test-sharding) 설정도 고려해보세요.

## CI에서 브라우저 다운로드 최적화

CI에서만큼은 필요한 브라우저만 설치하세요. 만약 Chromium만 테스트한다면, Chromium만 설치하세요.

```bash
# Instead of installing all browsers
npx playwright install --with-deps

# Install only Chromium
npx playwright install chromium --with-deps
```

이렇게 하면 CI machines의 디스크 공간을 아낄 수 있습니다.

## 테스트 lint

오류를 조기에 발견하기 위해, 테스트 코드에도 TypeScript와 ESLint를 사용하는 것을 권장합니다. Playwright API는 대부분 비동기 함수이므로, `await`를 빠뜨리는 실수를 방지하기 위해 `@typescript-eslint/no-floating-promises` 규칙을 사용하는 것이 좋습니다. CI에서는 `tsc --noEmit` 명령어를 실행하여 함수들이 정확한 시그니처로 호출되었는지 타입 검사를 수행할 수 있습니다.

## 분산 실행과 병렬 실행 이용하기

Playwright는 기본적으로 테스트를 [병렬](https://playwright.dev/docs/test-parallel)로 실행합니다. 한 파일 내의 테스트는 같은 worker 프로세스에서 순차적으로 실행됩니다. 한 파일 내에 여러 개의 독립 테스트가 존재할 때, 이들을 병렬로 실행하고 싶을 수 있습니다.

```tsx
import { test } from '@playwright/test';

test.describe.configure({ mode: 'parallel' });

test('runs in parallel 1', async ({ page }) => { /* ... */ });
test('runs in parallel 2', async ({ page }) => { /* ... */ });
```

Playwright는 테스트 스위트를 분할([shard](https://playwright.dev/docs/test-parallel#shard-tests-between-multiple-machines))하여 **여러 머신에서 병렬로 실행**할 수 있습니다.

```tsx
pnpm exec playwright test --shard=1/3
```

## 생산성 팁

## Soft Assertion 사용하기

테스트가 실패하면 Playwright는 **VS Code, 터미널, HTML 리포트 또는 Trace Viewer**에서 **어느 부분이 실패했는지 알려주는 오류 메시지**를 제공합니다.

하지만 **Soft Assertion**을 사용하면, 테스트가 실패하더라도 **즉시 실행을 중단하지 않고**, 테스트가 끝난 후에 **실패한 assertion 목록을 한 번에 보여줍니다**.

```tsx
// 실패해도 테스트를 중단하지 않고 계속 진행함
await expect.soft(page.getByTestId('status')).toHaveText('Success');

// 이후 단계도 계속 실행됨
await page.getByRole('link', { name: 'next page' }).click();
```

Soft Assertion을 사용하면 **여러 조건을 한 번에 확인하고**, **테스트 흐름을 유지하면서 진단할 수 있어 디버깅에 유용**합니다.