# CodeRabbit 로컬 리뷰 가이드

CodeRabbit CLI를 사용하면 PR을 만들거나 원격에 푸시하기 전에 로컬 Git 변경을 리뷰할 수 있습니다. PR 리뷰와 결과가 완전히 같지는 않지만, 구현 직후 버그와 보안 문제를 빠르게 찾는 사전 점검에 유용합니다.

## 설치와 인증

공식 설치 스크립트는 macOS, Linux, Windows의 WSL 환경에서 사용할 수 있습니다.

```bash
curl -fsSL https://cli.coderabbit.ai/install.sh | sh
```

설치 후 새 셸을 열고 인증합니다.

```bash
cr auth login
cr auth status
```

`cr`은 `coderabbit`의 짧은 별칭이며 두 명령은 동일하게 동작합니다. CLI는 초기화된 Git 저장소 안에서 실행해야 합니다.

## 자주 쓰는 명령

```bash
# 커밋된 변경과 미커밋 변경을 모두 리뷰
cr

# 미커밋 변경만 리뷰
cr --type uncommitted

# 커밋된 변경만 리뷰
cr --type committed

# 기준 브랜치를 명시
cr --base develop

# 사람이 읽기 좋은 일반 텍스트
cr --plain

# AI 에이전트가 처리하기 좋은 구조화된 결과
cr --agent --type uncommitted
```

현재 공식 CLI의 리뷰 범위는 `all`, `committed`, `uncommitted`입니다. 기본 기준 브랜치가 `main`이 아니라면 `--base`로 명시하세요.

## 권장 로컬 리뷰 루프

1. 기능을 작은 변경 단위로 구현합니다.
2. 프로젝트의 기존 테스트와 린트를 먼저 실행합니다.
3. `git diff`로 리뷰 범위를 확인합니다.
4. `cr --type uncommitted`를 실행합니다.
5. Critical과 Major 문제의 근거를 검토하고 필요한 수정만 반영합니다.
6. 테스트를 다시 실행하고 CodeRabbit 리뷰를 한 번 더 수행합니다.
7. 두 번째 리뷰에서 중대한 문제가 없으면 사소한 제안은 별도 작업으로 분리합니다.

반복 횟수와 종료 조건을 먼저 정하면 AI 리뷰의 사소한 지적을 끝없이 수정하는 상황을 피할 수 있습니다.

## AI 코딩 에이전트와 함께 사용하기

AI 에이전트에는 범위와 완료 조건을 명확히 전달합니다.

```text
현재 미커밋 변경을 구현한 뒤 cr --agent --type uncommitted를 실행해 주세요.
Critical과 Major 문제만 근거를 검토해 수정하고, 테스트 후 한 번만 재검토하세요.
두 번째 검토에서 중대한 문제가 없으면 종료하고 수정 내역을 요약하세요.
```

Claude Code 플러그인을 설치한 환경에서는 다음처럼 호출할 수도 있습니다.

```text
/coderabbit:review
/coderabbit:review committed
/coderabbit:review uncommitted
/coderabbit:review --base main
```

플러그인 명령은 설치된 버전에 따라 달라질 수 있으므로 [공식 Claude Code 통합 문서](https://docs.coderabbit.ai/cli/claude-code-integration)를 확인하세요.

## 문제 해결

### 변경을 찾지 못할 때

- `git status`로 추적 중인 변경이 있는지 확인합니다.
- `--type uncommitted` 또는 `--type committed`가 의도와 맞는지 확인합니다.
- 기본 브랜치가 다르면 `--base develop`처럼 지정합니다.
- CodeRabbit CLI는 코드 파일 중심으로 리뷰하므로 문서나 설정만 바뀐 경우 결과가 적을 수 있습니다.

### 리뷰가 오래 걸릴 때

- 리뷰 범위를 미커밋 변경으로 제한합니다.
- 큰 기능을 작은 브랜치와 커밋으로 나눕니다.
- AI 에이전트가 실행한다면 백그라운드 실행 후 주기적으로 상태를 확인하게 합니다.

### Windows에서 느릴 때

WSL에서 `/mnt/c/` 아래의 Windows 파일을 직접 다루면 느릴 수 있습니다. 성능이 중요하면 WSL의 Linux 파일시스템 아래에 저장소를 복제해 사용합니다.

## 참고 자료

- [CodeRabbit 최신 업데이트: 로컬 리뷰](https://www.coderabbit-users.kr/blog/coderabbit-latest-updates-local-review)
- [CodeRabbit CLI 공식 가이드](https://docs.coderabbit.ai/cli/index)
- [CodeRabbit CLI 명령 레퍼런스](https://docs.coderabbit.ai/cli/reference)
