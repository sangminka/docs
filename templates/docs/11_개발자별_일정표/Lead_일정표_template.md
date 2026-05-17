# 📋 {{PROJECT_NAME}} — Lead (책임개발자) 상세 일정표

> **문서 버전:** v1.0
> **작성일:** YYYY-MM-DD
> **담당자:** {{LEAD_NAME}}
> **역할:** 책임개발자 (Tech Lead)
> **개발 기간:** W0(킥오프) ~ W4(시연)
> **연관 문서:** 00_개발_일정_총괄표 / 01_프로젝트_계획서 / 03_아키텍처_정의서 / AGENTS.md

---

## 📌 변경 이력

| 버전 | 날짜 | 작성자 | 주요 변경 |
| --- | --- | --- |
| v1.0 | YYYY-MM-DD | Lead | 초기 작성 |

---

## 0. Lead 역할 핵심 선언

> **"내가 막히면 팀 전체가 막힌다."**
> Lead는 기능 개발 속도보다 **팀이 병렬로 달릴 수 있는 기반**을 만드는 것이 최우선이다.
> 나의 블로커를 팀보다 먼저 공유하고, 모든 의사결정에 **24시간 이내 응답**을 보장한다.

### Lead만 가진 권한 & 책임

| 권한 | 책임 |
| --- | --- |
| `main` / `develop` 브랜치 PR 머지 | 코드 품질·보안·컨벤션 최종 게이트 |
| 아키텍처·기술스택 최종 결정 | ADR 작성 + 팀 설명 의무 |
| 운영 서버 접근 | 운영 사고 시 1차 대응 |
| AGENTS.md 최종 편집권 | AI 환각 패턴 수집 + 주간 보강 |
| Scope 추가/삭제 결정 | MVP Lock + Out-Scope 백로그 관리 |
| PR 머지 거부권 | 이유 명시 + 24시간 내 재검토 |

---

## 1. 소유권 선언 (Code Ownership)

> 아래 파일/패키지는 **Lead만 수정**한다. 다른 팀원이 수정이 필요하면 반드시 Lead에게 먼저 알린다.

```
src/main/java/{{BASE_PACKAGE}}/
├── config/                 ← 전담 소유 (Security, JPA, Redis, Cors 등)
│   ├── SecurityConfig.java
│   ├── JpaConfig.java
│   ├── RedisConfig.java
│   └── WebConfig.java
├── common/                 ← 전담 소유
│   ├── exception/          (BusinessException, GlobalExceptionHandler)
│   ├── dto/                (ApiResponse, PageResponse)
│   └── util/               (DateUtil, StringUtil 등)
├── security/               ← 전담 소유 (인증·인가 로직)
│   ├── CustomUserDetails.java
│   ├── JwtTokenProvider.java (또는 SessionManager)
│   └── CustomUserDetailsService.java
└── external/               ← 전담 소유 (외부 API 클라이언트)
    └── {{EXTERNAL_API}}/
        ├── {{API}}Client.java
        └── {{API}}Response.java

AGENTS.md                   ← 전담 소유
.github/workflows/          ← DevOps/QA와 공동 소유 (Lead 최종 승인)
docker-compose*.yml         ← Lead + DevOps/QA 공동
application-prod.properties ← Lead 전담 (운영 민감 설정)
```

> **침범 규칙:** 다른 팀원이 위 경로를 수정하는 PR을 올리면, Lead를 **반드시 리뷰어에 추가**해야 한다.
> Lead 없이 merge 금지.

---

## 2. 일별 상세 일정

### 🟦 W0 — 킥오프 & 문서 폭주 주간

| 일자 | 오전 | 오후 | 산출물 (Done 기준) |
| --- | --- | --- | --- |
| **D-5 (화)** | 주제 선정 회의 주도 (1시간). 한 줄 정의문 확정. | BOOTSTRAP 문서 팀 공유. AGENTS.md 초안 작성 시작. | 주제 정의문 Slack 공지 완료 |
| **D-4 (수)** | **킥오프 회의 진행 (2시간)**. 역할 확정, AGENTS.md 검토, 도구 합의. | 01_계획서 v1 작성 (목표·Scope·MVP 5±2개). Dev A와 페어. | AGENTS.md commit, 역할표 노션 박제 |
| **D-3 (목)** | 02_ERD v1 작성 주도 (Dev B와 페어). 엔티티 6~10개 확정. | 03_아키텍처 v1 작성. 패키지 구조 확정 후 skeleton 레포 init. | ERD v1 + 패키지 구조 commit |
| **D-2 (금)** | Security 설계 확정 (인증 방식: JWT vs Session). 09_Git 규칙 + 브랜치 보호 설정. | 04_API 명세서 공통 규칙 (URL 패턴, 응답 형식, 인증 헤더). Spring Boot 프로젝트 init + Gradle 설정. | `main` 브랜치 보호 + PR 템플릿 설정 완료 |
| **D-1 (토)** | (선택) 외부 API 조사·Spike (API Key 발급, 요청/응답 구조 확인). | AGENTS.md 최종 보강 (팀 피드백 반영). application*.properties 3종 구조 설계. | 외부 API Spike 결과 #dev 채널 공유 |

**W0 종료 게이트 (D-1 저녁 전 완료)**
- [ ] `AGENTS.md` develop 머지, 6명 모두 pull 완료
- [ ] 01·02·03·09 v1 노션 등록
- [ ] GitHub: main 보호, PR 템플릿, 이슈 라벨 세팅
- [ ] 6명 역할·소유 모듈 README.md 명시

---

### 🟩 W1 — 인프라 + 공유 골격 구축 주간

**Lead W1 집중 목표:** "팀이 독립적으로 달릴 수 있는 공통 기반을 금요일까지 완성한다"

| 일자 | 오전 (수업 전) | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W1 월** | Stand-up 진행 (09:00, 15분). 전체 일정 재확인. | **Entity + Repository 선행 작성** (공통 사용 엔티티 먼저). Spring Security 기본 설정. | 짧은 회고 (10분). 각 Dev에게 Entity 확정 버전 공유. |
| **W1 화** | Stand-up. ERD 최종 확인 → **W1 화 마감 = ERD Lock**. | Security + JWT/Session 구현. `CustomUserDetails`, `JwtTokenProvider`. application*.properties 3종 완성. | 인증 기본 동작 확인. #dev 채널에 API Base URL 공지. |
| **W1 수** | Stand-up + AI 사용 사례 공유. | `GlobalExceptionHandler`, `BusinessException` 계층 완성. `ApiResponse<T>` DTO 작성. | 공통 레이어 PR 올리기 (Self-Review 먼저). |
| **W1 목** | Stand-up. Dev들 모듈 골격 진행 상황 점검. | **외부 API Spike** — 실제 HTTP 호출 테스트. 폴백 전략 문서화. | API 연동 초안 결과 #dev 공유. |
| **W1 금** | Stand-up. | 전체 PR 리뷰 + 머지. CI 동작 확인. **W1 게이트 회의 (16:00, 60분)**. | W1 회고 기록. 다음 주 분배 재확인. |

**W1 Lead 산출물 (Done 기준 — 모두 develop에 merge된 상태)**

| 산출물 | 파일 경로 | 완료 기준 |
| --- | --- | --- |
| Spring Security 설정 | `config/SecurityConfig.java` | 인증 필요 경로·허용 경로 모두 동작 |
| 인증 구현 | `security/*.java` | 로그인 → 토큰/세션 발급 → 보호 API 호출 성공 |
| 공통 예외 처리 | `common/exception/*.java` | 400/401/403/404/500 모두 `ApiResponse` 형식 |
| 공통 DTO | `common/dto/*.java` | Dev들이 import해서 사용 가능 |
| 외부 API Client 뼈대 | `external/{{API}}Client.java` | Mock으로 단위 테스트 1개 통과 |
| application*.properties | 3종 (dev/test/prod) | Profile 전환 동작 확인 |
| AGENTS.md W1 보강 | `AGENTS.md` | 공통 예외·DTO·패키지 구조 반영 |

**W1 금 게이트 체크리스트**
```
[ ] CI: 모든 PR 빌드·테스트·린트 green
[ ] develop 브랜치 보호 규칙 동작 (PR 없이 push 불가)
[ ] 6명 모두 로컬에서 ./gradlew bootRun --args='--spring.profiles.active=dev' 성공
[ ] Repository 단위 테스트 커버리지 30%+
[ ] 외부 API: 실제 응답 받기 성공 (또는 대체 Mock 완성)
[ ] 다음 주 모듈 의존성 충돌 없음 확인 (공통 DTO 확정)
```

---

### 🟨 W2 — 외부 API 본 연동 + 리뷰 집중 주간

**Lead W2 집중 목표:** "외부 API 연동 완성 + 모든 PR 24시간 내 리뷰"

| 일자 | 오전 | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W2 월** | Stand-up. 모듈별 진행 상황 리뷰. | 외부 API 본 연동 시작. `{{API}}Service.java` 구현. | PR 리뷰 최소 2건. |
| **W2 화** | Stand-up. API 불일치 이슈 있으면 즉시 API 정합 미팅 소집 (1시간). | 외부 API 오류 처리 + 폴백 구현. Circuit Breaker 고려. | 모듈 간 인터페이스 확정 공지. |
| **W2 수** | Stand-up. | 공통 레이어 추가 필요 사항 반영 (Dev들 요청 수렴). 보안 점검 1회차. | PR 리뷰 집중. |
| **W2 목** | Stand-up. | 외부 API 연동 단위 테스트 완성 (Mock 기반). API 명세서 v1 최종 Fix. | 04_API 명세서 Lock 공지. |
| **W2 금** | Stand-up. | 중간 데모 준비. **W2 게이트 회의 (16:00, 60분)**. | W2 회고 + AGENTS.md 보강. |

**W2 Lead 산출물**

| 산출물 | 완료 기준 |
| --- | --- |
| 외부 API 연동 완성 | 실제 호출 + 폴백 모두 동작. 단위 테스트(Mock) 통과 |
| API 명세서 v1 Lock | 04_API.md `v1-FINAL` 태그, 팀 전체 합의 |
| 보안 점검 1차 | XSS·CSRF·SQL Injection·인증 우회 체크리스트 통과 |
| PR 리뷰 누적 | W2 기간 중 모든 PR 24시간 이내 리뷰 완료 |

---

### 🟥 W3 — 통합·UI 마감·운영 배포 주간

**Lead W3 집중 목표:** "W3 금요일 운영 서버에서 시연 시나리오 통과"

| 일자 | 오전 | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W3 월** | Stand-up. 전체 라우팅 연결 상태 확인. | 통합 이슈 디버깅 지원. 공통 레이어 긴급 수정. | MVP 90% 달성 여부 점검. |
| **W3 화** | Stand-up. **기능 동결(Feature Freeze) 선언** — Slack + 노션. | 남은 핵심 버그 수정. 운영 배포 환경 최종 점검 (DevOps/QA와 합동). | 배포 스크립트 테스트. |
| **W3 수** | Stand-up. | 통합 테스트 통과 여부 확인. 스테이징 배포. | 통합 테스트 실패 항목 처리. |
| **W3 목** | Stand-up. | **운영 배포** (DevOps/QA와 합동). 시드 데이터 삽입. | 시연 시나리오 1차 리허설 준비. |
| **W3 금** | Stand-up. | **시연 1차 리허설 (16:00)**. W3 게이트 회의. | W3 회고. 백업 영상 녹화 준비 지시. |

**W3 Lead 산출물**

| 산출물 | 완료 기준 |
| --- | --- |
| 운영 배포 | 외부 URL로 접속 + 로그인 + 핵심 기능 동작 |
| 시드 데이터 | 시연 시나리오 전체 커버하는 데이터 삽입 완료 |
| 보안 점검 2차 | 운영 환경 민감 정보 노출 없음 확인 |
| 시연 1차 리허설 | 시연 시나리오 성공률 80%+ |

---

### ⬜ W4 — 시연 준비 주간

| 일자 | 활동 | 산출물 |
| --- | --- | --- |
| **W4 월** | Q&A 30선 작성 + 시연 대본 v1 리뷰. | 시연 대본 v1 확정 |
| **W4 화** | **시연 리허설 1회차 주관 (60분)**. 피드백 정리. | 리허설 피드백 문서 |
| **W4 수** | **백업 영상 녹화** (장애 대비 필수). 18_README 최종. | 백업 영상 저장 + README 완성 |
| **W4 목** | 시연 리허설 2회차. 발표자 컨디션 확인. | 시연 대본 최종 |
| **W4 금(시연일)** | D-30분 헬스체크. 시연 진행. 즉시 21_회고 시작. | 회고 문서 초안 |

---

## 3. 기술 기준 & 지침

### 3.1 아키텍처 결정 기준 (ADR 기준)

다음 기준에 해당하면 **반드시 ADR을 작성**한다:

- 기술 스택 추가 (새 라이브러리, 외부 서비스)
- 패키지 구조 변경
- 인증 방식 변경
- 배포 구조 변경
- 데이터베이스 스키마 공통 필드 변경

ADR 형식:
```markdown
## ADR-{{번호}}: {{제목}}
- **날짜:** YYYY-MM-DD
- **상태:** 제안 / 승인 / 폐기
- **컨텍스트:** 왜 이 결정이 필요한가
- **결정:** 무엇을 선택했는가
- **결과:** 예상 트레이드오프
```

### 3.2 PR 리뷰 기준 (Lead 리뷰 게이트)

Lead는 다음 기준으로 모든 PR을 검토한다:

```
필수 통과 항목:
[ ] CI green (빌드 + 테스트 + 린트)
[ ] 모듈 소유권 침범 없음 (공통 레이어 수정 시 ADR 또는 사전 협의)
[ ] 보안 체크 (시크릿 하드코딩, SQL Injection, XSS, 인증 우회)
[ ] 컨벤션 준수 (AGENTS.md 기준)
[ ] 테스트 누락 없음 (기능 추가 시 테스트 동반 필수)
[ ] PR 크기 400줄 이하 (초과 시 분할 요청)
[ ] AI 생성 PR이면 환각 체크리스트 적용

거부 사유 명시 후 24시간 내 재검토:
- 거부 사유를 PR 코멘트에 구체적으로 작성
- 수정 방향 제안 포함
```

### 3.3 보안 코드 기준

```java
// ✅ 인증 필요 엔드포인트: SecurityConfig에 명시
http.authorizeHttpRequests(auth -> auth
    .requestMatchers("/api/public/**").permitAll()
    .requestMatchers("/api/admin/**").hasRole("ADMIN")
    .anyRequest().authenticated()
);

// ✅ 모든 사용자 입력: 검증 필수
@Valid @RequestBody CreateReservationRequest request
// + Bean Validation 어노테이션 사용 (@NotBlank, @Size 등)

// ✅ 비밀번호: BCrypt만 사용
passwordEncoder.encode(rawPassword); // 항상 이 방식

// ✅ 민감 설정: 환경변수로
@Value("${external.api.key}")
private String apiKey;
// application.properties에 절대 하드코딩 금지
```

---

## 4. AI 에이전트 활용 가이드 (Lead 특화)

### Lead가 AI를 써도 되는 영역

| 영역 | 도구 | 프롬프트 패턴 |
| --- | --- | --- |
| AGENTS.md 초안 작성 | Claude 웹 | "이 프로젝트의 AGENTS.md를 작성해줘. 기술스택: {{스택}}, 패키지: {{패키지}}, 규칙: {{규칙}}" |
| ERD → Entity 변환 | Claude Code | "@docs/02_ERD.md 참조. {{엔티티명}} Entity를 JPA로 작성해줘. Audit 컬럼 포함." |
| 보일러플레이트 생성 | Claude Code | "GlobalExceptionHandler를 작성해줘. BusinessException 계층 구조와 ApiResponse<T> 반환 형식을 따를 것." |
| 외부 API Client 뼈대 | Claude Code | "{{API}} HTTP 클라이언트를 Spring WebClient로 작성해줘. 타임아웃 3초, 재시도 2회, 폴백 포함." |
| ADR 초안 | Claude 웹 | "{{기술 결정}} 에 대한 ADR을 작성해줘. 컨텍스트: {{배경}}. 대안: {{A vs B}}." |

### Lead가 AI에 절대 맡기지 않는 것

- 🚫 JWT 서명 키 생성·관리
- 🚫 SecurityConfig 최종 경로 매핑 결정
- 🚫 운영 DB 스키마 변경 SQL 최종 실행
- 🚫 PR 최종 머지 승인

---

## 5. 협업 포인트

| 협업 대상 | 시점 | 내용 |
| --- | --- | --- |
| **Dev A** | W1 화 | Security 경로 매핑 확정 (AuthController ↔ SecurityConfig) |
| **Dev A** | W1 수 | `getCurrentMember()` 공개 인터페이스 서명 확정 + 전 팀원 공지 |
| **Dev B** | W1 수 | 트랜잭션 전략 + 동시성 처리 방식 합의 (비관적/낙관적 락) |
| **Dev C** | W1 수 | 외부 API Client 위치·사용법 확인 (`external/` 패키지 역할 분리) |
| **Dev D** | W1 월 | 공통 레이아웃 Mustache 구조 + 권한별 메뉴 분기 방식 합의 |
| **DevOps/QA** | W0 금 | CI 첫 가동 + main/develop 브랜치 보호 규칙 공동 설정 |
| **DevOps/QA** | W3 목 | 운영 배포 합동 실행 + 헬스체크 확인 |
| **전원** | W2 목 | API 명세서 v1 Lock — 변경 없음 합의 후 공지 |
| **전원** | W3 금 | 시연 1차 리허설 주관 |


## 6. 블로커 에스컬레이션 프로세스

Lead가 막혔을 때:

```
1단계 (막힌 즉시): #dev 채널에 "🚨 블로커: [내용] — [예상 영향]" 공지
2단계 (1시간 내 해결 안 될 때): 팀원과 페어 코딩 or 분리 작업 제안
3단계 (반나절 해결 안 될 때): 해당 기능을 Out-Scope로 이관 검토
```

Lead가 다른 팀원의 블로커를 받았을 때:

```
접수 즉시: Slack 스레드에 "접수함 — N시간 이내 답변"
30분 이내 페어 코딩 or 방향 제시
24시간 내 해결 안 되면: 해당 기능 Out-Scope 검토 or 일정 재산정
```

---

## 7. 기술 블로그 주제 가이드 (포트폴리오)

Lead가 W3~W4에 작성할 노션 기술 블로그 주제:

| 주제 | 내용 | 어필 포인트 |
| --- | --- | --- |
| "6명 팀의 Spring Security 설계" | 인증 방식 결정 과정, ADR | 기술 의사결정 능력 |
| "외부 API 폴백 전략" | Circuit Breaker 또는 폴백 구현 | 운영 마인드 |
| "AI 에이전트 협업 회고" | 잘된 것 / AI 환각 사례 | AI 활용 역량 |
| "6명 병렬 개발에서 머지 지옥 피하기" | 모듈 소유권 + PR 전략 | 협업 경험 |

---

## 8. 일일 체크리스트 (Lead용)

### 매일 Stand-up 전 (09:00)

```
[ ] 어제 리뷰 대기 PR 있으면 오늘 오전 중 처리
[ ] 어제 말한 블로커 해결됐는가
[ ] develop 최신화 + CI 상태 확인
[ ] 오늘 집중할 Lead 작업 1가지 결정
```

### 매일 저녁 회고 전 (18:00)

```
[ ] 오늘 PR 리뷰 완료했는가 (24시간 이내 기준)
[ ] AGENTS.md 업데이트 필요한 패턴 발견했는가
[ ] 내일 팀이 막힐 수 있는 것 미리 준비했는가
[ ] AI 환각 발견 시 #dev 채널 공유했는가
```

### 매주 금요일 통합 미팅 준비

```
[ ] 주간 AI 사용량 요약 준비 (1분 정리)
[ ] 이번 주 ADR 정리
[ ] 다음 주 Critical Path 재확인
[ ] AGENTS.md 주간 보강 완료
```

---

## 9. 성공 기준 (Lead 개인)

| 기준 | 목표 | 측정 방법 |
| --- | --- | --- |
| PR 리뷰 응답 시간 | 24시간 이내 100% | GitHub Insights |
| CI green 유지 | develop 브랜치 red 하루 이하 | Actions 대시보드 |
| 공통 레이어 안정성 | 공통 레이어로 인한 다른 팀원 블로커 0건 | 회고 기록 |
| 보안 이슈 0건 | 운영 배포 시 민감 정보 노출 0 | 수동 점검 |
| 시연 준비 | W3 금 리허설 성공률 100% | 리허설 기록 |

---

## 📋 Lead 체크리스트 총괄

```
W0:
[ ] AGENTS.md develop 머지 + 팀 전원 pull 확인
[ ] 역할·소유 모듈 README 박제
[ ] main/develop 브랜치 보호 규칙 설정

W1:
[ ] Security + 인증 develop 머지
[ ] 공통 예외·DTO develop 머지
[ ] 외부 API 뼈대 + Spike 결과 공유
[ ] AGENTS.md W1 보강 완료

W2:
[ ] 외부 API 본 연동 완성 (폴백 포함)
[ ] API 명세서 v1 Lock + 팀 공지
[ ] 보안 점검 1차 완료

W3:
[ ] Feature Freeze 선언 (W3 화)
[ ] 운영 배포 완료
[ ] 시드 데이터 삽입 완료
[ ] 시연 1차 리허설 통과

W4:
[ ] 백업 영상 녹화 완료
[ ] 시연 2차 리허설 통과
[ ] 회고 문서 초안 작성
```
