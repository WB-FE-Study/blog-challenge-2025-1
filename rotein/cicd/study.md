# 개요

- WIP
  - 이 문서는 작성중인 문서입니다.
  - 이 문서는 학습 내용을 러프하게 정리하는 문서입니다.
    - 실제 블로그는 좀 더 실제 적용된 내용들을 위주로 작성하고자 합니다.
  - 아직 정리되지 않은 글이라서, 현재 간단한 문장으로만 쓰여 있습니다.

# 용어집

## 배포가 무엇인가요?

서비스를 다른 사람들이 인터넷을 통해 접근할 수 있게 만드는 것을 배포라고 합니다.

## EC2는 무엇인가요?

Elastic Compute Cloud - 클라우드 상의 컴퓨터를 빌려서 원격으로 사용할 수 있는 서비스입니다.

## CI/CD는 무엇인가요?

CI/CD는 각각 Continuous Integration(지속적 통합), Continuous Deployment(지속적 배포)의 약자입니다. 즉 테스트(Test), 통합(Merge), 배포(Deploy)의 과정을 자동화하는 걸 의미합니다.

# CI/CD

## CI/CD가 왜 필요한가요?

서비스를 운영하면서 새로운 기능을 추가할 때마다 직접 컴퓨터 서버(ex. AWS EC2)에 접속해서 새로운 코드를 다운받아 실행시켜주어야 하는 과정이 필요합니다. 매번 이 과정을 반복하는 것은 귀찮고 비효율적이기 때문에 좀 더 효율을 높이고 시간을 줄이기 위해 이 과정을 자동화해야 하는데, 그것이 바로 CI/CD입니다.

## CI/CD 의 프로세스

개발 → 커밋 → 빌드 → 테스트 → 배포

## CI/CD를 구축하는 툴

- Github Actions
- Jenkins
- Circle CI
- Travis CI
- ...

# Github Actions

## Github Actions 흐름

- 커밋 → 푸시 → 푸시된 코드를 감지하여 Github Actions 로직 실행(빌드, 테스트, 배포) → 최신 코드로 서버 재설정
- Workflow → Job → Step

## Github Actions 기본 문법

### Github Actions 관련 파일들의 위치

프로젝트 최상단에 정확히 아래와 같은 디렉터리를 추가합니다. Github Actions 에서 읽을 수 있는 디렉터리 구조이기 때문에 반드시 지켜야 합니다.

- `/.github`
- `/.github/workflows`

### workflows

`/.github/workflows` 디렉터리는 Github Action 에서 실행할 워크플로를 정의하는 곳입니다.
여기에 yml 문법을 작성하여 워크플로를 정의할 수 있습니다.

- `name:`
  - 말 그대로, Actions 에 찍히는 이름
- `on:`
  - 실행 트리거를 정의
- `jobs:`
  - 하나의 workflow 는 1개 이상의 job 으로 구성된다.
  - 여러 개의 job은 기본적으로 병렬로 수행된다.

### on

아래는 on 에 대한 예시다. 꽤 직관적이다.

```yml
on:
	push:
		branches:
			- main
```

### jobs

jobs 하위에 job을 정의하게 되는데, 2depth 에 job 의 식별값을 자유롭게 입력해준다.

```yml
jobs:
	Hello-World: # 식별값
		...
```

각 job 에는 어떤 프로퍼티가 정의될 수 있을까.

- `runs-on:` job이 실행 될 운영체제 환경을 정의한다.
  - e.g. `runs-on: ubuntu-latest`
- `steps:` job 수행의 가장 작은 단위. job 은 여러 step 으로 구성되어 있다.
  - `- name:` step 의 이름
  - `  run:` step 에서 실행할 리눅스 명령어를 여기 정의한다.
    - e.g. `run: echo "Hello World"`

### secrets

Github Actions 에서는 필요에 따라 비밀 환경변수(secret)를 사용할 수 있다.
secret 을 사용하면, 실제 Actions 로깅에는 별표(`***`)로 찍힌다.

#### yml 파일에서 secret 값 사용하는 법

`${{ secrets.SOME_SECRET_VALUE }}`

#### github 내 에서 secret 값 설정하는 곳

settings → Secrets and variables → actions → secret(tab) → New repository secret
