# OpenSpec으로 SDD 개발하기

> 정리 기준일: 2026-07-26
> OpenSpec은 빠르게 발전하고 있습니다. 설치하거나 업데이트하기 전에 [공식 문서](https://github.com/Fission-AI/OpenSpec)를 확인하세요.

## OpenSpec이란?

[OpenSpec](https://github.com/Fission-AI/OpenSpec)은 AI 코딩 에이전트와 사람이 구현 전에 요구사항과 설계를 합의하도록 돕는 가벼운 SDD(Spec-Driven Development) 프레임워크입니다.

대화 기록에만 요구사항을 남기면 AI가 누락된 조건을 추측하거나 구현 도중 범위를 바꾸기 쉽습니다. OpenSpec은 변경마다 제안서, 요구사항, 기술 설계, 작업 목록을 Markdown 파일로 만들고 AI가 이를 기준으로 구현하게 합니다.

기본 흐름은 다음과 같습니다.

```text
Explore(선택) → Propose → 사람의 검토 → Apply → Sync(선택) → Archive
```

OpenSpec의 특징은 다음과 같습니다.

- 단계 사이를 자유롭게 오가며 산출물을 수정할 수 있습니다.
- 새 프로젝트뿐 아니라 기존 프로젝트에도 적용할 수 있습니다.
- 요구사항은 평범한 Markdown과 시나리오로 작성합니다.
- 변경 중인 명세와 현재 시스템의 명세를 분리해 관리합니다.
- 특정 IDE나 AI 모델에 종속되지 않고 여러 코딩 에이전트와 함께 사용할 수 있습니다.

OpenSpec은 문서를 많이 만드는 것 자체가 목적이 아닙니다. 구현 전에 사람과 AI가 같은 변경 범위, 성공 조건, 기술 방향을 공유하도록 만드는 것이 목적입니다.

## 1. 설치

### 사전 준비

- Node.js 20.19.0 이상
- npm 또는 호환 패키지 관리자
- Git
- OpenSpec이 지원하는 AI 코딩 에이전트

OpenSpec CLI를 전역으로 설치합니다.

```bash
npm install -g @fission-ai/openspec@latest
```

설치를 확인합니다.

```bash
openspec --version
openspec --help
```

공식 문서는 pnpm, yarn, bun, Nix를 이용한 설치 방법도 제공합니다. 팀에서는 설치 방식을 하나로 정하고 프로젝트 문서에 기록하는 편이 좋습니다.

## 2. 프로젝트 초기화

OpenSpec을 사용할 프로젝트 루트로 이동한 뒤 초기화합니다.

```bash
cd your-project
openspec init
```

초기화 과정에서 사용하는 AI 코딩 도구를 선택합니다. OpenSpec은 선택한 도구가 읽을 수 있는 명령 또는 스킬 파일을 프로젝트에 설치합니다.

초기화가 끝나면 다음을 확인합니다.

```bash
openspec list
openspec view
```

- `openspec list`: 현재 진행 중인 변경을 표시합니다.
- `openspec view`: 변경과 명세를 탐색하는 대시보드를 엽니다.

### 기존 프로젝트에 적용할 때

초기화 전에 작업 트리가 깨끗한지 확인하고 별도 브랜치에서 실행하는 것이 안전합니다.

```bash
git status
git switch -c chore/init-openspec
openspec init
git diff
```

생성된 파일을 검토한 뒤 커밋합니다. 기존 프로젝트에서는 먼저 작은 변경 하나로 전체 흐름을 시험하고, 팀 규칙과 명세 범위를 점차 확장하는 편이 좋습니다.

## 3. 터미널 명령과 채팅 명령 구분하기

OpenSpec에는 실행 위치가 다른 두 종류의 명령이 있습니다.

| 종류 | 실행 위치 | 예 |
| --- | --- | --- |
| CLI 명령 | PowerShell, 터미널, 셸 | `openspec init`, `openspec list` |
| 워크플로 명령 | AI 코딩 에이전트의 채팅 입력창 | `/opsx:propose`, `/opsx:apply` |

`/opsx:propose` 같은 명령은 터미널 명령이 아닙니다. 별도의 OpenSpec 대화형 셸을 실행하는 것도 아닙니다. Claude Code, Codex, Cursor 등 초기화할 때 선택한 AI 도구의 채팅창에 입력합니다.

도구에 따라 명령 표시 방식이나 호출 방법이 다를 수 있습니다. 초기화 후 생성된 에이전트 명령 또는 스킬 목록을 우선 확인하세요.

## 4. 생성되는 폴더와 파일

기본 `spec-driven` 스키마를 사용하면 프로젝트에 다음 구조가 만들어집니다.

```text
openspec/
├── specs/                         # 현재 시스템 동작의 기준 명세
│   └── <domain>/
│       └── spec.md
├── changes/                       # 진행 중인 변경
│   └── <change-name>/
│       ├── proposal.md            # 왜, 무엇을 바꾸는가
│       ├── specs/                 # 이번 변경의 델타 명세
│       │   └── <domain>/
│       │       └── spec.md
│       ├── design.md              # 어떻게 구현할 것인가
│       └── tasks.md               # 구현 작업 목록
├── changes/archive/               # 완료된 변경의 이력
└── config.yaml                    # 프로젝트 설정(선택)
```

### `openspec/specs/`

현재 시스템이 보장해야 하는 동작을 도메인별로 기록한 기준 명세입니다. 예를 들어 인증은 `specs/auth/spec.md`, 결제는 `specs/payments/spec.md`처럼 나눌 수 있습니다.

### `openspec/changes/`

앞으로 반영할 변경을 독립된 폴더로 관리합니다. 아직 구현되지 않은 요구사항을 현재 시스템의 기준 명세와 섞지 않기 때문에 진행 중인 작업과 현재 사실을 구분할 수 있습니다.

### 변경 산출물

| 산출물 | 답해야 하는 질문 |
| --- | --- |
| `proposal.md` | 왜 필요한가? 무엇을 바꾸고 무엇은 제외하는가? |
| `specs/` | 사용자가 관찰할 동작과 성공·실패 조건은 무엇인가? |
| `design.md` | 어떤 구조와 기술적 결정으로 구현하는가? |
| `tasks.md` | 어떤 순서로 구현하고 어떻게 완료를 확인하는가? |

산출물은 일방향 단계가 아닙니다. 구현 중 새로운 사실을 알게 되면 먼저 관련 명세나 설계를 고치고, 영향을 받는 작업 목록과 코드를 다시 맞춥니다.

## 5. 프로필과 워크플로 선택

### 기본 `core` 프로필

처음 사용할 때는 기본 `core` 프로필이 가장 단순합니다.

```text
/opsx:explore    선택: 아이디어와 문제 탐색
      ↓
/opsx:propose    변경 생성과 계획 산출물 일괄 작성
      ↓
/opsx:apply      작업 목록을 따라 구현
      ↓
/opsx:sync       선택: 델타 명세를 기준 명세에 먼저 병합
      ↓
/opsx:archive    명세 동기화 확인 후 변경 보관
```

공식 README와 CLI 레퍼런스가 안내하는 기본 명령은 다음과 같습니다.

| 명령 | 용도 |
| --- | --- |
| `/opsx:explore` | 변경을 만들지 않고 문제와 선택지를 탐색 |
| `/opsx:propose` | 변경 폴더와 계획 산출물을 한 번에 생성 |
| `/opsx:apply` | `tasks.md`를 기준으로 구현 |
| `/opsx:sync` | 델타 명세를 현재 기준 명세에 병합 |
| `/opsx:archive` | 완료된 변경을 이력 폴더로 이동 |

공식 워크플로 명령 레퍼런스에는 기존 산출물을 수정하고 일관성을 조정하는 `/opsx:update`도 `core` 명령으로 소개되어 있지만, README와 CLI의 기본 워크플로 목록에는 빠져 있습니다. 버전별 구성이 다를 수 있으므로 초기화 후 실제 생성된 명령 목록을 기준으로 사용하세요. 명령이 없다면 파일을 직접 수정한 뒤 AI에게 관련 산출물의 일관성을 검토하게 할 수 있습니다.

### 확장 워크플로

산출물을 단계별로 만들거나 구현 검증을 워크플로 명령으로 수행하려면 확장 명령을 활성화합니다.

```bash
openspec config profile
openspec update
```

첫 명령에서 사용할 워크플로를 선택하고, 프로젝트 루트에서 `openspec update`를 실행해 에이전트 지침을 갱신합니다.

확장 워크플로의 대표 명령은 다음과 같습니다.

| 명령 | 용도 |
| --- | --- |
| `/opsx:new` | 빈 변경 구조를 생성 |
| `/opsx:continue` | 의존 관계에 따라 다음 산출물 하나를 생성 |
| `/opsx:ff` | 필요한 계획 산출물을 빠르게 일괄 생성 |
| `/opsx:verify` | 구현과 명세·설계·작업 목록의 일치 여부 검증 |
| `/opsx:bulk-archive` | 여러 완료 변경을 충돌 확인과 함께 보관 |
| `/opsx:onboard` | 실제 프로젝트에서 전체 흐름을 안내하는 튜토리얼 |

확장 흐름은 보통 다음과 같습니다.

```text
/opsx:new
    ↓
/opsx:continue 또는 /opsx:ff
    ↓
사람의 산출물 검토
    ↓
/opsx:apply
    ↓
/opsx:verify
    ↓
/opsx:archive
```

처음에는 `core`로 시작하고, 산출물별 승인이나 `/opsx:verify`가 필요할 때 확장 명령을 추가하는 방식을 권장합니다.

## 6. 기본 워크플로 따라 하기

아래 예제는 애플리케이션에 다크 모드를 추가하는 상황입니다.

### 1단계: Explore로 문제 탐색하기

요구사항이나 기술 방향이 불명확하면 AI 채팅창에서 탐색부터 시작합니다.

```text
/opsx:explore
다크 모드를 추가하고 싶다.
현재 스타일 구조를 조사하고 CSS 변수, 테마 라이브러리 사용 방식을 비교해 줘.
새 의존성은 가능하면 추가하지 않고 시스템 설정을 기본값으로 사용하고 싶다.
```

`explore`는 코드베이스를 조사하고 선택지와 장단점을 정리하지만 변경 산출물이나 구현 코드를 만들지 않습니다. 방향이 정해지면 변경 이름, 범위, 성공 조건을 정리해 `propose`로 전환합니다.

이미 구현할 내용이 명확하면 이 단계는 생략할 수 있습니다.

### 2단계: Propose로 변경 계획 만들기

AI 채팅창에서 다음과 같이 요청합니다.

```text
/opsx:propose add-dark-mode
설정 화면에 밝은 테마와 어두운 테마 선택 기능을 추가한다.
선택 값이 없으면 운영체제 설정을 따르고, 사용자 선택은 브라우저에 보존한다.
이번 변경에서는 사용자 계정 간 설정 동기화는 제외한다.
```

기본 `spec-driven` 스키마에서는 다음 산출물이 생성됩니다.

```text
openspec/changes/add-dark-mode/
├── proposal.md
├── specs/
│   └── ui/
│       └── spec.md
├── design.md
└── tasks.md
```

AI가 “구현 준비 완료”라고 말해도 바로 `apply`하지 말고 네 파일을 사람이 먼저 검토합니다.

### 3단계: 산출물 검토하기

#### `proposal.md`

- 해결하려는 사용자 문제가 명확한가?
- 포함 범위와 제외 범위가 구분되어 있는가?
- 기존 기능과 호환성에 미치는 영향이 적혀 있는가?
- 변경 이름이 한 가지 목적만 나타내는가?

#### 델타 `spec.md`

- 각 요구사항이 관찰 가능한 동작으로 작성되어 있는가?
- 정상, 실패, 경계 시나리오가 포함되어 있는가?
- `WHEN`과 `THEN`이 테스트 가능한가?
- 구현 방법이 요구사항에 섞여 있지 않은가?

#### `design.md`

- 기존 아키텍처와 패턴을 재사용하는가?
- 새 의존성이나 추상화가 꼭 필요한가?
- 데이터 흐름, 오류 처리, 마이그레이션 방법이 명확한가?
- 테스트와 롤백 방법이 있는가?

#### `tasks.md`

- 작업 순서와 의존 관계가 타당한가?
- 각 작업의 결과물과 완료 조건이 분명한가?
- 요구사항과 테스트 작업이 추적 가능한가?
- 한 번에 검토하기 어려울 정도로 큰 작업이 남아 있지 않은가?

수정이 필요하면 파일을 직접 고친 뒤 AI에게 나머지 산출물을 맞추도록 요청합니다. 설치된 명령 목록에 `/opsx:update`가 있다면 다음처럼 사용할 수 있습니다.

```text
/opsx:update add-dark-mode
테마 저장 위치를 localStorage에서 쿠키로 변경했다.
proposal, design, tasks 사이의 불일치를 찾아 하나씩 수정안을 제시해 줘.
```

변경의 목적 자체가 달라졌다면 기존 변경을 억지로 확장하지 말고 새 변경을 만드는 편이 안전합니다.

### 4단계: Apply로 구현하기

검토가 끝나면 AI 채팅창에서 구현을 시작합니다.

```text
/opsx:apply add-dark-mode
```

AI는 산출물을 읽고 `tasks.md`의 작업을 순서대로 구현하며 완료한 항목을 표시합니다. 작업 범위가 크면 한 번에 모두 구현하게 하지 말고 다음처럼 구간을 나눕니다.

```text
/opsx:apply add-dark-mode
테마 상태 관리와 단위 테스트 작업까지만 구현하고 결과를 보고해 줘.
```

각 구간이 끝날 때 다음을 확인합니다.

1. 수정 파일과 목적
2. 실행한 검증 명령과 결과
3. 완료 처리된 작업
4. 남은 작업과 새로 발견한 위험
5. 명세나 설계 변경 필요 여부

구현 중 요구사항이나 설계가 바뀌면 코드만 고치지 말고 먼저 산출물을 갱신합니다.

### 5단계: 구현과 명세 검증하기

프로젝트의 테스트, 린트, 타입 검사, 빌드를 직접 실행합니다. OpenSpec 검증은 프로젝트 테스트를 대체하지 않습니다.

```bash
# 프로젝트에 실제로 존재하는 명령만 실행
npm test
npm run lint
npm run build
```

CLI로 변경 상태와 문서 형식을 확인할 수 있습니다.

```bash
openspec list
openspec show add-dark-mode
openspec validate add-dark-mode
```

확장 워크플로를 활성화했다면 AI 채팅창에서 구현과 산출물의 일치 여부를 검사합니다.

```text
/opsx:verify add-dark-mode
```

`verify`는 다음 세 관점으로 검사합니다.

- 완전성: 모든 작업과 요구사항이 구현되었는가?
- 정확성: 구현이 명세의 의도와 경계 조건을 만족하는가?
- 일관성: 코드가 설계 결정과 기존 패턴을 따르는가?

경고가 있어도 보관은 가능하지만, 이유를 확인하지 않은 채 무시하지 않습니다. 테스트 누락은 테스트를 추가하고, 실제 구현이 타당하게 바뀌었다면 `design.md`와 관련 산출물을 수정합니다.

### 6단계: Sync로 기준 명세 갱신하기

`sync`는 변경 폴더의 델타 명세를 `openspec/specs/`의 현재 기준 명세에 병합합니다.

```text
/opsx:sync add-dark-mode
```

다음 상황에서 별도로 실행하면 유용합니다.

- 장기 변경이 끝나기 전에 기준 명세를 먼저 갱신해야 할 때
- 병렬 변경이 새 기준 명세를 참조해야 할 때
- 명세 병합 결과를 구현 보관과 분리해 검토하고 싶을 때

작은 변경은 `archive` 과정에서 동기화 여부를 묻기 때문에 `sync`를 생략해도 됩니다. `sync`만 실행하면 변경은 여전히 활성 상태로 남습니다.

### 7단계: Archive로 변경 마무리하기

테스트, 코드 리뷰, 명세 검토가 끝나면 AI 채팅창에서 변경을 보관합니다.

```text
/opsx:archive add-dark-mode
```

OpenSpec은 산출물과 작업 완료 상태를 확인하고, 아직 반영하지 않은 델타 명세가 있으면 동기화를 제안합니다. 이후 변경 폴더를 날짜가 포함된 보관 경로로 옮깁니다.

```text
openspec/changes/archive/2026-07-26-add-dark-mode/
```

보관 후에는 다음을 확인합니다.

```bash
openspec list
git diff
```

- 활성 변경 목록에서 해당 변경이 사라졌는가?
- `openspec/specs/`에 최종 동작이 반영되었는가?
- 보관 폴더에 제안, 명세, 설계, 작업 이력이 남아 있는가?
- 코드와 OpenSpec 산출물을 같은 커밋이나 PR 범위로 추적할 수 있는가?

## 7. 좋은 델타 명세 작성하기

델타 명세는 현재 명세에 비해 무엇이 달라지는지 표현합니다.

```markdown
# UI 명세 변경

## ADDED Requirements

### Requirement: 테마 선택
시스템은 사용자가 밝은 테마와 어두운 테마 중 하나를 선택할 수 있게 해야 한다.

#### Scenario: 사용자가 어두운 테마를 선택
- **GIVEN** 설정 화면을 연 인증된 사용자가 있고
- **WHEN** 사용자가 어두운 테마를 선택하면
- **THEN** 화면은 즉시 어두운 테마로 바뀌고
- **AND** 다음 방문에도 선택이 유지된다.

#### Scenario: 저장된 선택이 없음
- **GIVEN** 저장된 테마 선택이 없고
- **WHEN** 애플리케이션을 처음 열면
- **THEN** 운영체제의 색상 모드를 기본값으로 사용한다.
```

주요 변경 구분은 다음과 같습니다.

| 구분 | 의미 |
| --- | --- |
| `ADDED Requirements` | 새로운 요구사항 추가 |
| `MODIFIED Requirements` | 기존 요구사항의 동작 변경 |
| `REMOVED Requirements` | 기존 요구사항 제거 |
| `RENAMED Requirements` | 요구사항 이름 변경 |

좋은 요구사항은 구현 기술보다 외부에서 관찰할 수 있는 결과를 설명합니다.

```text
나쁜 예: React Context와 localStorage를 사용한다.
좋은 예: 사용자가 선택한 테마는 브라우저를 다시 열어도 유지된다.
```

기술 선택은 `design.md`에 기록합니다. 요구사항과 설계를 분리하면 구현 방법을 바꾸더라도 사용자에게 약속한 동작은 유지할 수 있습니다.

## 8. 실전 운영 원칙

### 변경 하나에는 목적 하나만 담기

`add-dark-mode-and-refactor-auth`처럼 관계없는 작업을 묶지 않습니다. 독립적으로 검토하고 배포할 수 있는 단위로 나눕니다.

### 구현 전에 사람의 승인점 두기

최소한 `proposal.md`, 델타 명세, `design.md`, `tasks.md`를 읽은 뒤 구현합니다. 보안, 데이터 마이그레이션, 공개 API 변경은 담당자의 명시적 승인을 받습니다.

### 명세를 현재 사실로 유지하기

구현 과정에서 사실이 바뀌면 산출물을 먼저 수정합니다. 보관된 변경은 과거의 의사결정을 설명하고, `openspec/specs/`는 현재 시스템의 동작을 설명해야 합니다.

### 검증 명령을 작업에 구체적으로 적기

“테스트한다”보다 실행할 명령과 기대 결과를 적습니다.

```markdown
- [ ] `npm test -- theme` 실행 결과가 성공한다.
- [ ] 저장된 설정이 없는 브라우저에서 운영체제 테마를 따른다.
- [ ] 새로고침 후 사용자가 고른 테마가 유지된다.
```

### 작은 변경으로 도입하기

기존 대규모 프로젝트의 모든 동작을 처음부터 명세화하지 않습니다. 앞으로 변경할 영역의 현재 동작을 먼저 기록하고, 변경이 생길 때마다 기준 명세를 확장합니다.

### 코드와 산출물을 함께 리뷰하기

PR에는 코드뿐 아니라 다음도 포함합니다.

- 이번 변경의 `proposal.md`
- 델타 명세
- 기술 결정과 대안
- 완료된 작업 목록
- 동기화된 기준 명세 또는 보관 결과

## 9. 자주 발생하는 문제

### `/opsx:*` 명령을 터미널에서 실행했다

슬래시 명령은 AI 코딩 에이전트의 채팅창에서 실행합니다. 터미널에서는 `openspec ...` CLI 명령만 실행합니다.

### 슬래시 명령이 보이지 않는다

다음을 순서대로 확인합니다.

1. 프로젝트 루트에서 `openspec init`을 실행했는가?
2. 초기화할 때 현재 AI 도구를 선택했는가?
3. 에이전트를 다시 시작하거나 명령 목록을 새로 불러왔는가?
4. OpenSpec을 업그레이드한 뒤 `openspec update`를 실행했는가?
5. 확장 명령이라면 `openspec config profile`에서 해당 워크플로를 선택했는가?

### `/opsx:verify`가 없다

`verify`는 기본 `core` 프로필이 아닌 확장 워크플로 명령입니다.

```bash
openspec config profile
openspec update
```

프로필을 바꾼 뒤 AI 도구를 다시 시작하거나 명령을 새로 불러옵니다.

### AI가 산출물을 만들자마자 구현을 시작한다

요청에 승인 조건을 명시합니다.

```text
/opsx:propose add-rate-limit
계획 산출물만 작성하고 구현하지 마.
각 파일의 핵심 결정을 요약한 뒤 내 승인을 기다려.
```

### 명세와 구현이 달라졌다

차이의 원인을 먼저 판단합니다.

- 구현이 잘못되었다면 코드를 명세에 맞춥니다.
- 구현 과정에서 더 나은 결정을 내렸다면 `design.md`, 델타 명세, `tasks.md`를 먼저 수정합니다.
- 요구 목적이 달라졌다면 새 변경으로 분리합니다.

수정 후 프로젝트 테스트와 `/opsx:verify`를 다시 실행합니다.

### 변경 이름을 인식하지 못한다

활성 변경 이름을 확인하고 명시적으로 전달합니다.

```bash
openspec list
openspec show <change-name>
```

### 보관 전에 델타 명세가 충돌한다

병렬 변경이 같은 도메인 명세를 수정했는지 확인합니다. 먼저 완료된 변경을 동기화한 뒤 다른 변경의 델타를 새 기준에 맞게 갱신하고 다시 검증합니다. 여러 변경을 함께 처리해야 하고 확장 명령을 사용 중이라면 `/opsx:bulk-archive`로 충돌 검사를 받을 수 있습니다.

### OpenSpec 문서 형식 검증이 실패한다

```bash
openspec validate <change-name>
openspec show <change-name>
```

오류가 가리키는 요구사항 제목, 변경 구분, 시나리오 형식을 확인합니다. 형식만 억지로 맞추기 전에 요구사항의 의미가 명확한지도 함께 검토합니다.

## 10. 업데이트와 설정

CLI 패키지를 업데이트합니다.

```bash
npm install -g @fission-ai/openspec@latest
```

각 프로젝트에서 AI 지침과 명령을 갱신합니다.

```bash
cd your-project
openspec update
```

업데이트 전후에는 다음을 확인합니다.

```bash
openspec --version
git diff
```

생성 파일이 바뀌었다면 현재 팀 워크플로에 영향을 주는지 검토한 뒤 커밋합니다.

OpenSpec은 명령 이름과 버전으로 구성된 익명 사용 통계를 수집하며, 인자·경로·파일 내용·개인정보는 수집하지 않는다고 안내합니다. CI에서는 자동으로 비활성화됩니다. 로컬에서도 끄려면 환경에 맞게 다음 변수 중 하나를 설정합니다.

```bash
export OPENSPEC_TELEMETRY=0
# 또는
export DO_NOT_TRACK=1
```

PowerShell에서는 다음처럼 설정할 수 있습니다.

```powershell
$env:OPENSPEC_TELEMETRY = "0"
```

## 완료 체크리스트

- [ ] 터미널 명령과 AI 채팅 명령의 실행 위치를 구분했는가?
- [ ] 변경의 목적과 제외 범위가 `proposal.md`에 명확한가?
- [ ] 요구사항에 정상, 실패, 경계 시나리오가 있는가?
- [ ] 기술 설계가 기존 구조와 제약을 반영하는가?
- [ ] 작업마다 결과물과 검증 방법이 있는가?
- [ ] 구현 전에 사람이 산출물을 검토했는가?
- [ ] 프로젝트의 실제 테스트, 린트, 빌드를 실행했는가?
- [ ] 구현과 산출물의 불일치를 해소했는가?
- [ ] 델타 명세가 현재 기준 명세에 반영되었는가?
- [ ] 완료된 변경을 보관하고 Git 이력에 남겼는가?

## 참고 자료

- [OpenSpec GitHub 저장소](https://github.com/Fission-AI/OpenSpec)
- [OpenSpec Getting Started](https://github.com/Fission-AI/OpenSpec/blob/main/docs/getting-started.md)
- [OpenSpec 명령 실행 방식](https://github.com/Fission-AI/OpenSpec/blob/main/docs/how-commands-work.md)
- [OpenSpec 워크플로 명령 레퍼런스](https://github.com/Fission-AI/OpenSpec/blob/main/docs/commands.md)
- [OpenSpec CLI 레퍼런스](https://github.com/Fission-AI/OpenSpec/blob/main/docs/cli.md)
- [OpenSpec 핵심 개념](https://github.com/Fission-AI/OpenSpec/blob/main/docs/concepts.md)
- [OpenSpec 기존 프로젝트 적용 가이드](https://github.com/Fission-AI/OpenSpec/blob/main/docs/existing-projects.md)
- [OpenSpec 문제 해결](https://github.com/Fission-AI/OpenSpec/blob/main/docs/troubleshooting.md)
