# {{PROJECT_NAME}} — AI 에이전트 협업 가이드

> **문서 버전:** v1.0
> **작성일:** YYYY-MM-DD
> **개발 기간:** {{DURATION}}
> **팀 구성:** 책임개발자 1명 + 개발자 {{N}}명 + AI 에이전트
> **연관 문서:** 03 아키텍처 정의서 / 09 Git 규칙 / 12 코드 리뷰 규칙

---

## 📌 변경 이력

| 버전 | 날짜 | 작성자 | 주요 변경 |
| --- | --- | --- | --- |
| v1.0 | YYYY-MM-DD | Lead | 초기 작성 |

---

## 0. 이 문서의 목적

AI 에이전트(Claude Code, Cursor, GitHub Copilot 등)를 사용해 개발할 때 **팀 전체가 일관된 결과물을 만들기 위한 규칙**을 정의한다. AI는 능력 있는 신입 개발자라고 가정하고 운영한다 — 명확한 컨텍스트·검증 단계·리뷰 게이트가 모두 필요하다.

---

## 1. 사용 도구 & 역할 분담

### 1.1 팀 표준 AI 에이전트

| 도구 | 용도 | 사용 권장 시점 |
| --- | --- | --- |
| **Claude Code** | 멀티파일 리팩터링, 테스트 작성, 코드 리뷰, 명세→코드 | 큰 단위 기능 구현, 디버깅 |
| **Cursor / Windsurf** | IDE 안에서 컨텍스트 인식 자동완성 | 일상 개발(코드 옆에서) |
| **GitHub Copilot** | 라인 단위 자동완성, 빠른 보일러플레이트 | 반복 코드 작성 |
| **ChatGPT/Claude (웹)** | 설계 토론, 학습, 임시 검색 | 설계 단계, 학습 |

> **운영 원칙:** "한 작업 = 한 도구" 가 깔끔하다. 같은 PR을 여러 AI로 동시에 손대면 컨벤션이 깨진다.

### 1.2 사람 vs AI 책임 매트릭스

| 작업 | 사람 (필수) | AI (가능) | 비고 |
| --- | --- | --- | --- |
| 요구사항 분석·설계 | ✅ | 보조 | 의사결정은 사람 |
| ERD·API 명세 초안 | ✅ | 초안 생성 | 최종 확정은 책임개발자 |
| 보일러플레이트·CRUD | 검토 | ✅ | AI가 빠르게 |
| 비즈니스 로직 구현 | ✅ 작성 | 보조 | 도메인 이해 필요 |
| 단위 테스트 작성 | 케이스 정의 | ✅ 코드 작성 | Given-When-Then 사람이 정의 |
| 외부 API 연동 | ✅ | 보조 | 키 관리·폴백은 사람 |
| 보안·인증·인가 | ✅ | 검토만 | AI 단독 작성 금지 |
| 환경설정·배포 | ✅ | 보조 | 운영 책임 사람 |
| 코드 리뷰 (1차) | ✅ | 보조 | AI는 자동 검토 도구 |
| 코드 리뷰 (최종 승인) | ✅ | 불가 | 사람만 |

### 1.3 절대 AI에 맡기지 않는 것

- 🚫 인증·인가 로직, JWT/세션 검증
- 🚫 결제·금액 계산, 트랜잭션 경계
- 🚫 비밀번호 해시, 암호화 키 관리
- 🚫 데이터 마이그레이션 스크립트의 최종 실행 (검증은 가능)
- 🚫 운영 DB·운영 서버 직접 명령

---

## 2. 프로젝트 컨텍스트 파일 구조

### 2.1 파일 구성 (저장소 루트)

```
{{repo}}/
├── AGENTS.md                ← 모든 AI 에이전트 공통 규칙 (Single Source of Truth)
├── CLAUDE.md                ← Claude Code 전용 추가 규칙 (선택)
├── .github/
│   └── copilot-instructions.md  ← Copilot 전용 규칙 (선택)
├── .cursorrules             ← Cursor 전용 규칙 (선택, 또는 .cursor/rules/)
├── docs/agent_context/      ← AI에게 참조시킬 보조 문서 (긴 가이드는 여기로 분리)
└── ...
```

> **원칙:** `AGENTS.md`를 단일 소스로 유지. 도구별 파일은 그 도구만의 차이점만 적고 `AGENTS.md`를 참조한다.

### 2.2 `AGENTS.md` 권장 분량 — **150줄 이내**

500줄 넘는 컨텍스트 파일은 모델이 절반 이상 무시한다. 짧고 핵심만 적는다. 자세한 설계는 `02_ERD`, `03_아키텍처`, `04_API` 문서에 있고 AI는 그쪽을 직접 읽는다.

### 2.3 `AGENTS.md` 표준 섹션 (이 순서 유지)

```markdown
# {{PROJECT_NAME}} — AI Agent Instructions

## 1. 프로젝트 개요 (3~5줄)
무엇을 / 왜 / 누구를 위해.

## 2. 기술 스택 (버전 명시)
- Backend: Spring Boot 3.x, JDK 21, Gradle
- Frontend: ...
- DB: MySQL 8 / Redis 7

## 3. 아키텍처 핵심
- 패키지 구조 한 줄 요약
- 자세한 내용은 docs/03_아키텍처_정의서.md 참조

## 4. 코드 규칙 (린터가 못 잡는 것만)
- Controller 반환: SSR=String / REST=ResponseEntity<T>
- Service 메서드명: 동사+명사 (createReservation)
- 예외: BusinessException 상속, 전역 핸들러에서 처리

## 5. 테스트 규칙
- Service 80%+, Repository 70%+
- Given-When-Then 주석 필수
- 외부 API는 Mock (실제 호출 금지)

## 6. 명령어 (AI가 자주 쓰는 것만)
- `./gradlew test` — 전체 테스트
- `./gradlew test --tests "*ReservationServiceTest"` — 단일 테스트
- `./gradlew bootRun --args='--spring.profiles.active=dev'` — 로컬 실행

## 7. AI 작업 시 반드시 지킬 것
- 새 기능은 반드시 테스트 동반
- 보안·인증 코드는 사람 검토 전 merge 금지
- 외부 라이브러리 추가 전 책임개발자 승인
- 1000줄 넘는 PR 만들지 말 것 (분할 요청)

## 8. 반드시 피할 것
- 기존 컨벤션과 다른 새 패턴 도입
- 주석으로만 된 "TODO" 코드
- console.log / printStackTrace 남기기
- 환경변수 하드코딩
```

### 2.4 도메인 단어집 (선택)

도메인 특수 용어는 별도 `docs/agent_context/glossary.md`로 분리하고 AGENTS.md에서 한 줄로 참조. AI가 "예약/슬롯/접수" 같은 용어를 잘못 쓰면 단어집을 보강한다.

---

## 3. 프롬프트 엔지니어링 5계명

### 3.1 컨텍스트 → 작업 → 제약 → 검증 (CTPV 패턴)

```
[Context]    이 프로젝트는 {{PROJECT_NAME}}이고, ReservationService에 작업.
             ERD: docs/02_ERD.md, API: docs/04_API.md 참조.
[Task]       전화 예약 생성 메서드를 추가해줘.
[Constraints] - 트랜잭션 경계: @Transactional
              - 중복 슬롯 검증 필수
              - 기존 createReservation과 같은 패턴
[Verify]     테스트 추가하고 ./gradlew test 통과 확인 후 보고.
```

### 3.2 5계명

1. **작게 자른다** — 한 번에 하나의 기능. "전체 예약 시스템 만들어줘"는 금지.
2. **참조 파일 명시** — `@docs/02_ERD.md` `@src/.../ReservationService.java`처럼 구체적 파일을 가리킨다.
3. **Plan First, Code Later** — 큰 작업은 "먼저 계획만 보여줘"로 시작. 승인 후 구현.
4. **체크리스트 강제** — "구현 후 다음 4가지를 확인했다고 보고: 테스트 통과 / 린트 통과 / 기존 패턴 준수 / 보안 체크"
5. **자기 비판 요청** — 결과 코드를 받은 뒤 "이 코드의 잠재적 문제 3가지를 스스로 찾아서 알려줘"로 1차 자기 검토.

### 3.3 좋은 프롬프트 vs 나쁜 프롬프트

| ❌ 나쁨 | ✅ 좋음 |
| --- | --- |
| "예약 시스템 만들어줘" | "ReservationService.createReservation의 로직을 따라, 전화 예약용 createPhoneReservation을 추가. 슬롯 검증 + 트랜잭션 + 테스트 포함." |
| "이거 더 좋게 해줘" | "이 메서드의 N+1 쿼리를 fetch join으로 해결. 변경 후 동일 테스트가 통과해야 함." |
| "버그 고쳐줘" | "이 테스트(ReservationServiceTest:42)가 NullPointerException으로 실패. ReservationServiceImpl:78 라인을 보고 원인 분석 → 수정안 → 테스트 통과 순서로 진행." |

---

## 4. AI 산출물 검증 절차

### 4.1 6단계 검증 (모든 AI PR은 통과 필수)

```
Step 1. 컴파일/빌드  ./gradlew build
Step 2. 단위 테스트   ./gradlew test
Step 3. 린트/포맷     ./gradlew check (또는 npm run lint)
Step 4. 기존 컨벤션  네이밍·패키지·반환 타입이 기존 파일과 일치
Step 5. 보안 체크    ✅ B 체크리스트 (12_코드_리뷰_규칙 참조)
Step 6. 사람 리뷰    PR에서 최소 1명 approve
```

### 4.2 AI 환각(Hallucination) 체크리스트

AI는 **존재하지 않는 메서드/라이브러리/플래그**를 만들어낸다. 머지 전에 반드시:

- [ ] import된 클래스가 실제로 존재하는가 (IDE 빨간 줄 확인)
- [ ] 호출한 메서드 시그니처가 라이브러리 공식 문서와 일치하는가
- [ ] 의존성이 build.gradle에 명시되어 있는가
- [ ] 환경변수/설정 키 이름이 실제 application.properties와 일치하는가
- [ ] 테스트가 실제로 실행되는가 (skip되거나 항상 통과되지 않는가)

### 4.3 AI가 자주 만드는 실수 (사전 방지)

| 패턴 | 사전 방지 방법 |
| --- | --- |
| 트랜잭션 경계 누락 | AGENTS.md에 "Service public 메서드는 @Transactional" 명시 |
| 예외를 RuntimeException으로 통째로 던짐 | 도메인 예외 계층을 AGENTS.md에 명시 |
| LocalDateTime 타임존 무시 | "DB는 UTC, 표시는 KST" 명시 |
| 페이징 없이 findAll() | "리스트 조회는 Pageable 필수" 명시 |
| 영문 주석/메시지 | "사용자 메시지는 한국어, 코드 주석은 한국어" 명시 |
| 테스트 케이스가 항상 통과 | "실패 케이스 1개 이상 포함" 명시 |
| 보안 우회 (인증 생략) | "@PreAuthorize 또는 SecurityConfig 매핑 필수" 명시 |

---

## 5. Git 워크플로우 통합

### 5.1 AI 작업 커밋 표시

- 커밋 메시지에 AI 보조 표시 금지 (Co-Authored-By 등). 작성자가 책임진다.
- AI를 썼더라도 **사람이 읽고 수정한 코드만** commit.
- 예외: 큰 리팩터링은 PR 본문에 "AI(Claude Code)로 1차 생성 후 사람 검토" 한 줄.

### 5.2 AI 보조 PR 라벨

GitHub 이슈/PR 라벨 추가:
- `ai-generated` — 코드의 50% 이상이 AI 생성
- `ai-assisted` — 일부 AI 보조 (자동완성 수준)

라벨이 있으면 리뷰어는 **환각 체크리스트(4.2)** 를 추가 적용.

### 5.3 AI 결과물의 PR 크기 관리

AI가 한 번에 1500줄을 만들어줘도 **PR은 400줄 이하로 분할**. AI가 만든 큰 덩어리를 그대로 PR로 올리면 리뷰가 무력화된다.

분할 기준:
- 엔티티/Repository 1개 PR
- Service + 단위 테스트 1개 PR
- Controller + Mustache/View 1개 PR

---

## 6. 비용 & 토큰 관리

### 6.1 컨텍스트 윈도우 절약

- AGENTS.md는 짧게 (150줄 이내)
- 한 세션에서 너무 많은 파일 열지 말 것 — 새 작업은 새 세션
- 대용량 로그·CSV는 직접 붙여넣지 말고 `head -100 file.log` 결과만
- 코드 변경이 많아진 세션은 정리 → 새 세션 시작

### 6.2 팀 사용량 모니터링

- 책임개발자가 매주 금요일 통합 미팅에서 토큰/요청 사용량 공유
- 비용 폭주 시 즉시 공지 (한 세션 100k+ 토큰)
- 무료 한도 도달 멤버는 다른 도구로 우회 (Cursor 무료 → Copilot)

---

## 7. AI를 활용한 정형 워크플로우 (Slash Commands / Skills)

자주 반복하는 작업은 **slash command** 또는 **skill**로 등록해 매번 같은 프롬프트를 다시 쓰지 않게 한다.

### 7.1 권장 등록 워크플로우

| 명령 | 용도 |
| --- | --- |
| `/scaffold-feature` | ERD 엔티티명을 받아 Entity / Repository / Service / Test 골격 생성 |
| `/api-from-spec` | 04_API 명세서 섹션을 받아 Controller + DTO 생성 |
| `/test-for` | 파일명을 받아 단위 테스트 자동 생성 |
| `/review` | PR diff에 대해 12_코드_리뷰 체크리스트 적용 |
| `/docs-sync` | 코드 변경에 따라 04_API / 06_화면 문서 업데이트 안내 |

> **저장 위치:** `.claude/commands/` (Claude Code), `.cursor/rules/` (Cursor) — Git에 commit해서 팀이 공유.

### 7.2 통합 미팅 시 회고

매주 금요일 회고에서 "이번 주 가장 자주 쓴 프롬프트 → 다음 주에 명령어로 등록"을 의제로.

---

## 8. AI 사용 학습 & 공유

### 8.1 팀원별 학습 자원

- Claude Code: <https://code.claude.com/docs/>
- Cursor: <https://docs.cursor.com/>
- Copilot: <https://docs.github.com/copilot>

### 8.2 노션 기술 블로그 연동

각 팀원이 일주일에 1편 이상 작성:
- "AI를 잘 다룬 사례" — 어떤 프롬프트가 잘 통했나
- "AI가 만든 버그" — 환각·실수 패턴
- "프롬프트 엔지니어링" — 자주 쓰는 패턴

이 글들은 그대로 **포트폴리오 기술 블로그 콘텐츠**가 된다.

---

## 9. 보안·법적 주의사항

### 9.1 절대 AI에 입력 금지

- 🚫 운영 DB 비밀번호, API 키, 토큰
- 🚫 사용자 개인정보 (실 데이터)
- 🚫 회사 NDA 코드, 사내 비공개 문서
- 🚫 다른 사람 PR을 그대로 학습용 데이터처럼 던지기

### 9.2 라이선스

- AI 생성 코드의 저작권은 일반적으로 사용자에게 귀속되나, 외부 라이선스(특히 GPL) 코드를 그대로 재현하는 경우가 있다 → 4.2 환각 체크에서 라이브러리 출처 확인.
- AI에게 외부 코드 패턴을 모방시킬 때 라이선스 호환성 확인.

### 9.3 .gitignore 추가

```
# AI 도구 로컬 설정
.cursor/
!.cursor/rules/
.claude/local/
*.local.md
AGENTS.override.md
```

---

## 10. 트러블슈팅

| 문제 | 대처 |
| --- | --- |
| AI가 AGENTS.md를 무시 | 너무 길거나 모순됨 → 50% 줄여보기 |
| 같은 실수 반복 | AGENTS.md "반드시 피할 것"에 추가 |
| 컨벤션이 자꾸 깨짐 | linter 자동화로 강제 (CLAUDE.md에 적지 말고 도구로) |
| 테스트가 항상 통과 | "실패 케이스 1개 포함"을 프롬프트에 강제 |
| 큰 리팩터링이 절반에서 망함 | Plan First → 단계 분할 → 한 단계씩 |
| 컨텍스트 토큰 폭증 | 새 세션 시작, 관련 파일만 다시 첨부 |

---

## 📋 AI 협업 일일 체크리스트 (개발자용)

```
[ ] 작업 시작 전 develop 최신화
[ ] 새 기능 → 작은 단위로 쪼갰는가
[ ] 프롬프트에 컨텍스트·제약·검증 단계 포함했는가
[ ] AI 결과물 — 빌드/테스트/린트 통과
[ ] 환각 체크 (존재하지 않는 메서드/라이브러리)
[ ] 보안·인증 부분은 본인이 다시 검토
[ ] PR 크기 400줄 이하
[ ] 커밋 메시지 사람이 작성 (Conventional Commits)
[ ] 테스트가 실제 실패 케이스 포함
```
