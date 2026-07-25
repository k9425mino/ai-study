# `.coderabbit.yaml` 설정 가이드

`.coderabbit.yaml`은 CodeRabbit의 리뷰 동작을 저장소별로 버전 관리하는 설정 파일입니다. 실제 적용할 파일은 반드시 저장소 루트에 두어야 합니다.

```text
repository/
├── .coderabbit.yaml
├── README.md
└── src/
```

## 시작 원칙

- 설정 없이 몇 개의 PR을 먼저 리뷰합니다.
- 기본값과 다른 동작이 필요한 키만 추가합니다.
- 스키마 URL을 첫 줄에 넣어 에디터 자동완성과 검증을 활용합니다.
- 설정 변경도 코드처럼 작은 PR로 리뷰하고 결과를 확인합니다.

```yaml
# yaml-language-server: $schema=https://coderabbit.ai/integrations/schema.v2.json
language: "ko-KR"

reviews:
  profile: "chill"
  auto_review:
    enabled: true
    drafts: false

chat:
  auto_reply: true
```

## 핵심 옵션

### `language`

리뷰 응답 언어를 지정합니다. 한국어는 `ko`와 `ko-KR`을 지원하며, 기본값은 `en-US`입니다. 기술 용어의 정확도와 팀의 소통 비용을 고려해 선택하세요.

### `reviews.profile`

현재 공식 스키마가 지원하는 값은 두 가지입니다.

- `chill`: 중요한 문제 위주로 비교적 적은 코멘트를 남깁니다.
- `assertive`: 더 적극적이고 세밀하게 리뷰합니다.

커뮤니티 자료에 등장하는 `strict`는 현재 공식 스키마의 유효한 값이 아니므로 사용하지 않습니다.

### `reviews.auto_review`

```yaml
reviews:
  auto_review:
    enabled: true
    drafts: false
    auto_incremental_review: true
    auto_pause_after_reviewed_commits: 5
    base_branches:
      - "develop"
      - "release/.*"
    ignore_usernames:
      - "dependabot[bot]"
    labels:
      - "!do-not-review"
```

- 기본 브랜치는 `base_branches` 배열이 비어 있어도 자동 리뷰 대상입니다.
- `base_branches`는 기본 브랜치 외에 추가할 대상이며 정규식을 지원합니다.
- `drafts: false`이면 Draft PR을 건너뜁니다.
- `auto_incremental_review`는 새 푸시마다 변경분을 다시 리뷰할지 정합니다.
- `auto_pause_after_reviewed_commits`는 과도한 증분 리뷰를 막습니다. `0`이면 자동 중지를 끕니다.
- `labels`에서 `!`로 시작하는 값은 해당 라벨의 PR을 제외합니다.

### `reviews.path_filters`

리뷰할 가치가 낮은 생성물과 의존성 파일을 제외합니다.

```yaml
reviews:
  path_filters:
    - "!**/dist/**"
    - "!**/node_modules/**"
    - "!**/generated/**"
    - "!**/*.lock"
    - "src/**"
```

- `!`로 시작하면 제외합니다.
- `!`가 없는 패턴은 명시적 포함입니다.
- 포함과 제외를 함께 쓸 때는 실제 리뷰 범위가 의도와 맞는지 작은 PR에서 확인합니다.
- CodeRabbit은 여러 빌드 산출물과 lock 파일을 기본으로 제외하므로, 프로젝트 고유 생성물 위주로 추가하세요.

### `reviews.path_instructions`

경로별로 검토 기준을 다르게 적용합니다.

```yaml
reviews:
  path_instructions:
    - path: "src/api/**"
      instructions: |
        인증과 권한 검사를 우선 확인하세요.
        외부 입력값의 검증 누락과 민감 정보 노출을 지적하세요.
    - path: "tests/**"
      instructions: |
        정상 경로뿐 아니라 오류와 경계 조건도 검증하는지 확인하세요.
    - path: "docs/**.md"
      instructions: |
        링크, 명령, 버전 정보가 실제 구현과 일치하는지 확인하세요.
```

지침은 추상적인 품질 요구보다 판별 가능한 규칙으로 씁니다. 모든 경로에 긴 규칙을 반복하지 말고, 해당 영역에서 실제로 자주 놓치는 항목만 추가합니다.

### 기존 지침 파일 활용

CodeRabbit은 `AGENTS.md`, `CLAUDE.md`, `GEMINI.md` 같은 지침 파일을 자동으로 감지해 리뷰 기준으로 활용할 수 있습니다. 이미 공통 규칙이 이 파일들에 있다면 `path_instructions`에 다시 복사하지 않습니다.

## 조직 단위 설정

여러 저장소를 운영한다면 중앙 설정과 설정 상속을 활용할 수 있습니다.

- 조직의 `coderabbit` 저장소에 공통 `.coderabbit.yaml`을 둡니다.
- 저장소별 설정은 필요한 차이만 오버라이드합니다.
- `inheritance: true`를 사용하면 중첩 객체는 병합되고, 경로별 지침 같은 배열은 안정적인 키를 기준으로 합쳐집니다.
- 보안·규정처럼 저장소가 임의로 해제하면 안 되는 정책은 조직의 Global Overrides를 사용합니다.

플랜과 Git 제공자에 따라 지원 범위가 다를 수 있으므로 적용 전 공식 문서를 확인하세요.

## 설정이 적용되지 않을 때

1. 파일명이 정확히 `.coderabbit.yaml`인지 확인합니다.
2. 저장소 루트에 있는지 확인합니다.
3. YAML 들여쓰기와 스키마 오류를 확인합니다.
4. PR의 대상 브랜치와 `base_branches` 조건을 확인합니다.
5. `path_filters`의 `!` 방향이 맞는지 확인합니다.
6. 새 커밋을 푸시한 뒤 리뷰를 다시 요청합니다.

## 예시 파일 사용법

[`.coderabbit.yaml.example`](./.coderabbit.yaml.example)을 저장소 루트의 `.coderabbit.yaml`로 복사한 뒤 실제 브랜치, 경로, 팀 규칙에 맞춰 줄이거나 수정하세요.

## 참고 자료

- [커뮤니티 `.coderabbit.yaml` 완전 가이드](https://www.coderabbit-users.kr/blog/coderabbit-yaml-complete-guide)
- [커뮤니티 `.coderabbit.yaml` 모범 사례](https://www.coderabbit-users.kr/best-practices/coderabbit-yaml-guide)
- [공식 YAML 설정 가이드](https://docs.coderabbit.ai/getting-started/yaml-configuration)
- [공식 설정 레퍼런스](https://docs.coderabbit.ai/reference/configuration)
- [공식 경로별 지침 가이드](https://docs.coderabbit.ai/configuration/path-instructions)
