# GitHub Spec Kit으로 SDD 개발하기

> 정리 기준일: 2026-07-26
> Spec Kit은 빠르게 발전하고 있습니다. 설치 전에 [공식 문서](https://github.github.io/spec-kit/)에서 최신 명령을 확인하세요.

## Spec Kit이란?

[GitHub Spec Kit](https://github.com/github/spec-kit)은 AI 코딩 에이전트가 즉석 프롬프트가 아니라 구조화된 스펙, 계획, 작업 목록을 따라 구현하도록 돕는 오픈소스 툴킷입니다.

기본 흐름은 다음 네 단계입니다.

```text
Spec → Plan → Tasks → Implement
```

실제 프로젝트에서는 품질 게이트를 포함한 다음 흐름을 권장합니다.

```text
Constitution
  → Specify
  → Clarify
  → Plan
  → Checklist
  → Tasks
  → Analyze
  → Implement
  → Converge
```

`Clarify`, `Checklist`, `Analyze`는 모든 작업의 필수 명령은 아니지만, 요구사항이 복잡하거나 잘못 구현했을 때 비용이 큰 기능에는 사용하는 편이 안전합니다.

## 1. 설치

### 사전 준비

- Git
- 사용할 AI 코딩 에이전트
- Python 도구를 격리 설치할 `uv`, `pipx` 또는 `pip`

현재 공식 문서는 PyPI의 `specify-cli` 설치를 지원합니다.

```bash
# uv 권장
uv tool install specify-cli

# 또는 pipx
pipx install specify-cli

# 또는 pip
pip install specify-cli
```

예전 블로그에는 GitHub 소스에서 직접 설치하는 명령이 나오지만, 특별히 개발 버전이 필요한 경우가 아니라면 패키지 릴리스를 사용하는 편이 재현하기 쉽습니다.

설치를 확인합니다.

```bash
specify --version
specify check
specify integration list
```

- `specify --version`: 설치 버전을 확인합니다.
- `specify check`: 필요한 AI 에이전트 도구가 설치되어 있는지 오프라인으로 확인합니다.
- `specify integration list`: 현재 버전이 지원하는 통합 키를 확인합니다.

설치된 기능이 공식 문서보다 오래된 것처럼 보이면 `specify self check`로 업데이트 여부를 확인합니다.

## 2. 프로젝트 초기화

### 새 프로젝트

```bash
specify init my-project --integration codex
cd my-project
```

### 기존 프로젝트

먼저 새 브랜치를 만들고 작업 트리가 깨끗한지 확인합니다. 비어 있지 않은 프로젝트는 초기화 파일과 기존 파일이 충돌할 수 있습니다.

```bash
specify init --here --integration codex
```

CLI가 비어 있지 않은 디렉터리에서 강제 병합을 요구한다면 생성·덮어쓰기 대상을 먼저 확인한 후에만 `--force`를 사용합니다.

### 대표 통합 키

| 도구 | 초기화 옵션 |
| --- | --- |
| Codex CLI | `--integration codex` |
| Claude Code | `--integration claude` |
| Gemini CLI | `--integration gemini` |
| GitHub Copilot | `--integration copilot` |
| Cursor | `--integration cursor-agent` |

통합 키는 버전에 따라 추가·변경될 수 있으므로 `specify integration list` 결과를 우선합니다.

## 3. 명령 호출 방식 확인

공식 문서는 공통 표기로 `/speckit.*`를 사용하지만 에이전트마다 실제 호출 방식이 다릅니다.

- 일반 슬래시 명령 통합: `/speckit.specify`
- Codex의 스킬 통합: `$speckit-specify`
- 일부 에이전트: `/skill:speckit-specify`

초기화 후 생성된 에이전트 명령이나 스킬 목록을 확인하고, 아래 예시의 `/speckit.*`를 현재 도구의 형식으로 바꿔 사용합니다.

## 4. Constitution: 프로젝트 원칙 정하기

Constitution은 개별 기능이 아니라 모든 작업에 적용할 불변 원칙을 정의합니다.

```text
/speckit.constitution
이 프로젝트는 다음 원칙을 따른다.
- 기존 공개 API의 하위 호환성을 유지한다.
- 모든 외부 입력은 경계에서 검증한다.
- 기능 변경에는 자동 테스트를 포함한다.
- 새 의존성은 추가 이유와 대안을 문서화한다.
```

좋은 원칙은 구체적이고 검증 가능합니다. “깨끗한 코드를 작성한다”처럼 해석이 넓은 문장보다 리뷰에서 준수 여부를 판별할 수 있는 문장을 사용합니다.

Constitution은 한 번 만든 뒤 방치하지 말고 팀의 실제 결정이 바뀔 때 갱신합니다. 다만 기능별 세부 요구사항을 Constitution에 넣어 거대한 규칙집으로 만들지 않습니다.

## 5. Specify: 무엇과 왜 정의하기

기능의 사용자 가치, 행동, 범위, 성공 조건을 설명합니다. 기술 스택과 구현 방법은 아직 넣지 않습니다.

```text
/speckit.specify
사용자는 이메일과 비밀번호로 로그인할 수 있어야 한다.
로그인 성공 후 대시보드로 이동하며 새로고침 후에도 세션이 유지되어야 한다.
5회 연속 실패하면 15분간 로그인을 제한한다.
OAuth, 2단계 인증, 비밀번호 재설정은 이번 범위에서 제외한다.
```

생성된 `spec.md`에서 다음을 검토합니다.

- 사용자 스토리가 실제 목표와 일치하는가?
- 인수 조건이 테스트 가능한가?
- 비범위가 명확한가?
- 실패와 경계 조건이 빠지지 않았는가?
- 기술 선택이 요구사항에 섞이지 않았는가?

첫 결과를 최종본으로 취급하지 않습니다. 잘못된 요구사항은 이후 모든 문서와 코드로 증폭됩니다.

## 6. Clarify: 애매한 부분 제거하기

```text
/speckit.clarify
권한, 세션 만료, 계정 잠금 해제 조건을 중심으로 명확화해 줘.
```

Clarify는 현재 스펙에서 중요한 질문을 골라 답을 `spec.md`에 반영합니다. 한 번에 모든 세부사항을 묻기보다 제품 정책, 데이터, 권한, 오류 처리처럼 초점을 바꿔 반복할 수 있습니다.

질문에 답할 수 없다면 AI가 추측하게 두지 말고 담당자 확인이 필요한 미정 항목으로 남깁니다.

## 7. Plan: 구현 방법 설계하기

요구사항이 합의된 뒤 기술 스택, 아키텍처, 데이터 모델, API, 마이그레이션, 테스트 전략을 정합니다.

```text
/speckit.plan
기존 Node.js와 TypeScript 구성을 유지한다.
세션은 현재 Redis 저장소를 사용하고 새 인증 라이브러리는 추가하지 않는다.
기존 로그인 API의 응답 형식을 유지한다.
계정 잠금 로직에는 단위 테스트와 API 통합 테스트를 포함한다.
```

Plan을 검토할 때는 다음을 확인합니다.

- 기존 코드 구조와 Constitution을 따르는가?
- 요청하지 않은 서비스나 추상화가 추가되지 않았는가?
- 새 의존성과 복잡성에 근거가 있는가?
- 데이터 이전과 하위 호환성 계획이 있는가?
- 테스트와 롤백 방법이 충분한가?

## 8. Checklist: 스펙의 품질 검사하기

```text
/speckit.checklist
인증 보안, 계정 잠금, 오류 메시지의 요구사항 완전성을 검사해 줘.
```

Checklist는 코드 테스트가 아니라 요구사항을 위한 테스트 목록입니다. 모호하거나 누락된 항목을 발견하면 체크만 남기지 말고 `Specify`나 `Clarify`로 돌아가 스펙 자체를 수정합니다.

## 9. Tasks: 실행 단위로 분해하기

```text
/speckit.tasks
```

Tasks는 계획을 의존성 순서가 있는 `tasks.md`로 변환합니다. 보통 Setup, 기반 작업, 우선순위별 사용자 스토리, 마무리 작업으로 나뉩니다.

좋은 작업 목록의 조건은 다음과 같습니다.

- 각 작업에 대상 파일이나 결과물이 명확합니다.
- 기반 작업이 사용자 기능보다 먼저 배치됩니다.
- 사용자 스토리마다 독립적으로 검증할 수 있습니다.
- 테스트가 해당 기능 작업과 함께 배치됩니다.
- 병렬 작업은 서로 같은 파일과 계약을 수정하지 않습니다.
- 각 작업의 완료 조건이 스펙 요구사항으로 추적됩니다.

실전 사례에서는 구현 단계가 `tasks.md`를 강하게 따르므로, Constitution과 스펙의 중요 규칙이 Tasks에 누락되지 않았는지 사람이 반드시 확인해야 합니다.

## 10. Analyze: 문서 간 모순 찾기

```text
/speckit.analyze
```

Analyze는 `spec.md`, `plan.md`, `tasks.md`를 읽기 전용으로 비교해 누락, 모순, 추적되지 않는 요구사항을 보고합니다.

- 요구사항 문제는 `Specify` 또는 `Clarify`에서 고칩니다.
- 기술 설계 문제는 `Plan`에서 고칩니다.
- 작업 누락은 원본 문서를 고친 뒤 `Tasks`를 다시 생성합니다.

보고서를 무시하고 바로 구현하지 말고, 수정 비용이 낮은 이 단계에서 모순을 해소합니다.

## 11. Implement: 단계별 구현하기

작은 기능은 전체를 실행할 수 있습니다.

```text
/speckit.implement
```

큰 기능은 범위를 제한해 단계별로 실행합니다.

```text
/speckit.implement
Setup과 기반 데이터 모델 작업만 구현하고 테스트한 뒤 중지해 줘.
```

각 단계에서 다음을 수행합니다.

1. 적용될 작업과 수정 파일을 확인합니다.
2. AI가 실행하려는 셸 명령과 권한을 검토합니다.
3. 기존 테스트와 새 테스트를 실행합니다.
4. 스펙 인수 조건을 대조합니다.
5. 코드 리뷰 후 다음 단계로 진행합니다.

AI가 생성한 코드를 기다렸다가 마지막에 한 번 검토하기보다, 사용자 스토리 또는 작업 묶음마다 중간 승인을 두는 편이 안전합니다.

## 12. Converge: 구현 누락 확인하기

```text
/speckit.converge
```

Converge는 구현 결과를 스펙·계획·작업 목록과 대조합니다. 코드를 직접 수정하지 않으며, 누락이 있으면 `tasks.md`에 추가 작업을 덧붙입니다.

```text
Implement → Converge → 누락 작업 추가 → Implement → Converge
```

Converged 결과가 나와도 자동 테스트, 사람 리뷰, 보안 검토를 생략하는 의미는 아닙니다. Spec Kit의 산출물끼리 일치한다는 품질 게이트로 사용합니다.

## 실전 권장 흐름

```text
1. 작은 기능 브랜치 생성
2. Constitution 검토
3. Specify 작성
4. Clarify와 사람 승인
5. Plan 작성과 아키텍처 리뷰
6. Checklist로 요구사항 검증
7. Tasks 생성과 범위 검토
8. Analyze로 모순 제거
9. Implement를 작은 단계로 실행
10. 테스트와 코드 리뷰
11. Converge로 누락 확인
12. 스펙과 함께 PR 생성
```

## 자주 발생하는 문제

### AI가 과잉 설계할 때

Plan에서 최소 변경, 기존 패턴 재사용, 새 의존성 제한을 명시합니다. 요청하지 않은 컴포넌트를 목록화하고 추가 근거를 설명하게 합니다.

### Constitution을 무시할 때

원칙이 Tasks와 검증 항목으로 내려왔는지 확인합니다. 추상적인 원칙은 작업 단계에서 사라지기 쉬우므로 테스트나 체크리스트로 구체화합니다.

### 구현 결과가 요구사항과 다를 때

코드만 패치하지 말고 `spec.md`의 모호함부터 찾습니다. 스펙을 고친 뒤 Plan과 Tasks를 다시 정렬하고 Analyze를 재실행합니다.

### 기존 프로젝트 초기화가 걱정될 때

깨끗한 브랜치 또는 복사본에서 먼저 실행하고 생성 파일을 검토합니다. `--force`는 대상 파일과 복구 방법을 확인한 경우에만 사용합니다.

### 명령이 문서와 다를 때

Spec Kit 버전과 통합 방식을 확인합니다.

```bash
specify --version
specify version --features
specify integration status
specify self check
```

블로그의 `--ai` 옵션과 짧은 `/specify`, `/plan` 명령은 이전 버전 예시일 수 있습니다. 현재 공식 CLI의 `--integration`과 에이전트가 실제로 노출한 명령을 우선합니다.

## 참고 자료

- [카카오페이: SDD(spec-kit) 에이전트 코딩 실전기](https://tech.kakaopay.com/post/ifkakao-agentic-coding/)
- [GitHub SpecKit 4단계 활용 소개](https://aspdotnet.tistory.com/3454)
- [Spec Kit 완벽 가이드](https://velog.io/@sammy0329/spec-kit-%EC%99%84%EB%B2%BD-%EA%B0%80%EC%9D%B4%EB%93%9C-AI%EC%99%80-%ED%95%A8%EA%BB%98%ED%95%98%EB%8A%94-Spec-Driven-Development)
- [GitHub Spec Kit 공식 문서](https://github.github.io/spec-kit/)
- [공식 Agentic SDD 명령 레퍼런스](https://github.github.io/spec-kit/reference/agentic-sdd.html)
- [공식 통합 목록](https://github.github.io/spec-kit/reference/integrations.html)
- [공식 설치 안내](https://github.github.io/spec-kit/install/pypi.html)
