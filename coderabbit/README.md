# CodeRabbit 학습 노트

CodeRabbit의 GitHub PR 리뷰, 로컬 CLI 리뷰, `.coderabbit.yaml` 설정, Change Stack 활용법을 빠르게 찾아볼 수 있도록 정리한 한국어 문서입니다.

> 정리 기준일: 2026-07-26
> 기능과 플랜은 바뀔 수 있으므로 실제 도입 전에는 [CodeRabbit 공식 문서](https://docs.coderabbit.ai/)를 함께 확인하세요.

## 문서 목록

| 문서 | 내용 |
| --- | --- |
| [CodeRabbit 사용 가이드](./01-usage-guide.md) | 개념, GitHub 연동, PR 리뷰 흐름, 장단점 |
| [로컬 리뷰 가이드](./02-local-review.md) | CLI 설치, 인증, 리뷰 범위, AI 에이전트 연동 |
| [`.coderabbit.yaml` 가이드](./03-yaml-guide.md) | 핵심 옵션, 경로 필터, 경로별 지침, 설정 상속 |
| [Change Stack과 Code Peek](./04-change-stack-and-code-peek.md) | Cohort, Code Peek, Chat Agent, 심각도 필터 |
| [운영 모범 사례](./05-best-practices.md) | 단계적 도입, 노이즈 관리, 사람 리뷰와의 역할 분담 |
| [설정 예시](./.coderabbit.yaml.example) | 저장소 루트로 복사해 수정할 수 있는 YAML |

## 가장 짧은 시작 순서

1. CodeRabbit GitHub App을 설치하고 대상 저장소에 권한을 부여합니다.
2. 기본 설정으로 몇 개의 PR을 리뷰해 봅니다.
3. 리뷰가 너무 많거나 팀 규칙을 놓치면 `.coderabbit.yaml`을 추가합니다.
4. PR을 올리기 전에는 `cr --type uncommitted`로 로컬 변경을 먼저 확인합니다.
5. CodeRabbit의 지적은 근거를 검토한 뒤 반영하며, 최종 판단은 사람이 내립니다.

## 참고 자료

- [CodeRabbit으로 GitHub PR 코드 리뷰 자동화하기](https://www.coderabbit-users.kr/blog/coderabbit-introduction)
- [CodeRabbit 최신 업데이트: 로컬에서 미리 코드 리뷰 받기](https://www.coderabbit-users.kr/blog/coderabbit-latest-updates-local-review)
- [`.coderabbit.yaml` 모든 옵션 정리](https://www.coderabbit-users.kr/blog/coderabbit-yaml-complete-guide)
- [Code Search와 Peek 활용](https://www.coderabbit-users.kr/blog/code-search-peek-in-coderabbit-review)
- [`.coderabbit.yaml` 모범 사례](https://www.coderabbit-users.kr/best-practices/coderabbit-yaml-guide)
