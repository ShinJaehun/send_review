# codex_review

Codex CLI + Web 인터페이스를 병행하여 사용하는 환경에서\
**spec → 구현 → review → 토론** 흐름을 안정적으로 관리하기 위한 Bash
도구 세트입니다.

------------------------------------------------------------------------

# 목적

이 도구의 목적은 다음 문제를 해결하는 것입니다:

-   코딩 머신 ↔ 웹 인터페이스 머신 분리 작업
-   작업 명세(spec) 동기화
-   git diff + commit 메시지 기반 리뷰 패킷 생성
-   토큰 절약을 위한 커밋 메시지 중심 토론

------------------------------------------------------------------------

# 구성 요소

    codex_review/
     ├── bin/
     │    ├── pull_spec
     │    ├── make_review
     │    └── send_review
     └── README.md

설정 파일:

    ~/.config/codex_review/config

------------------------------------------------------------------------

# 개념 구조

## spec.md

-   "현재 작업 계약서"
-   프로젝트 루트에 위치
-   항상 하나만 존재
-   pull_spec로 overwrite됨
-   git이 변경 이력을 관리

## review 파일

-   작업 결과 패킷
-   commit 메시지 + stat + diff 포함
-   timestamp prefix로 누적 관리
-   기본적으로 git에 포함하지 않음

------------------------------------------------------------------------

# 설치

## 1. 저장소 클론

``` bash
git clone <your_repo> ~/project/codex_review
```

## 2. 실행 권한 부여

``` bash
chmod +x ~/project/codex_review/bin/*
```

## 3. PATH 추가 (권장)

``` bash
export PATH="$HOME/project/codex_review/bin:$PATH"
```

.bashrc 또는 .zshrc에 추가 추천.

------------------------------------------------------------------------

# 설정

파일:

    ~/.config/codex_review/config

예시:

``` bash
CODEX_REVIEW_TARGET_USER="shinjaehun"
CODEX_REVIEW_TARGET_HOST="192.168.0.1"
CODEX_REVIEW_TARGET_DIR="~/review_diffs"

CODEX_REVIEW_SPEC_REMOTE="~/review_diffs/spec.md"

CODEX_REVIEW_PROJECT_DIR="$HOME/project/suksuk_project"
CODEX_REVIEW_SPEC_LOCAL="$CODEX_REVIEW_PROJECT_DIR/spec.md"

CODEX_REVIEW_OUT_DIR="/tmp"
```

------------------------------------------------------------------------

# SSH 준비 (최초 1회)

로컬에서:

``` bash
ssh-keygen
ssh-copy-id shinjaehun@192.168.0.1
```

테스트:

``` bash
ssh shinjaehun@192.168.0.1
```

비밀번호 없이 접속되면 OK.

------------------------------------------------------------------------

# 작업 흐름

## 시작

``` bash
cd ~/project/suksuk_project
pull_spec
```

동작: - 원격의 spec.md를 로컬 프로젝트 루트로 overwrite - 오늘 작업 명세 확정

------------------------------------------------------------------------

## Codex 작업

Codex CLI에서:

spec.md 기준으로 작업 수행

------------------------------------------------------------------------

## 작업 종료

``` bash
send_review HEAD
```

또는

``` bash
send_review b9e0c65..HEAD
```

동작:

1.  make_review 실행
2.  review 파일 생성
    -   timestamp 포함
    -   commit 메시지 항상 포함
3.  원격으로 scp 전송

------------------------------------------------------------------------

# review 파일 구조

    === REF or RANGE ===

    === COMMIT (subject + body) ===

    === LOG ===

    === STAT ===

    === DIFF ===

------------------------------------------------------------------------

# 토큰 절약 전략

웹 인터페이스에서:

1.  먼저 commit 메시지 섹션만 복사
2.  필요하면 STAT 추가
3.  그래도 필요하면 특정 파일 diff만 복사

→ 전체 diff를 항상 붙일 필요 없음\
→ 토큰 소비 최소화

------------------------------------------------------------------------

# spec 위치

프로젝트 루트:

    $HOME/project/suksuk_project/spec.md

항상 이 파일이 현재 작업의 SSOT.

------------------------------------------------------------------------

# review 파일 위치

기본:

    /tmp

원하면 config에서 변경 가능:

``` bash
CODEX_REVIEW_OUT_DIR="$HOME/review_diffs_local"
```

※ review 파일은 git에 포함시키지 않는 것을 권장.

------------------------------------------------------------------------

# Troubleshooting

## pull_spec 에러

-   SSH 키 확인
-   remote spec 경로 확인

``` bash
ssh shinjaehun@192.168.0.1
ls ~/review_diffs
```

------------------------------------------------------------------------

## send_review 실패

-   원격 디렉토리 권한 확인
-   scp 직접 테스트

``` bash
scp test.txt shinjaehun@192.168.0.1:~/review_diffs
```

------------------------------------------------------------------------

# 철학

-   spec = 입력(계약)
-   commit message = 의도
-   diff = 구현 증거
-   review = 교환 패킷
-   git = 과거 기록 저장소

------------------------------------------------------------------------

# 권장 운영 방식

로컬: 구현 전용\
원격: 설계 + 토론 전용

spec → 구현 → review → 토론 → 다음 spec

이 루프를 반복.

------------------------------------------------------------------------
