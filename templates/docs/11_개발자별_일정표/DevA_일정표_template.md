# 📋 {{PROJECT_NAME}} — Dev A 상세 일정표

> **문서 버전:** v1.0
> **작성일:** YYYY-MM-DD
> **담당자:** {{DEVA_NAME}}
> **역할:** 개발자 A
> **담당 모듈:** 인증·회원(Auth) + {{MODULE_A_NAME}} 화면군
> **개발 기간:** W1 ~ W4
> **연관 문서:** 00_개발_일정_총괄표 / 02_ERD / 04_API_명세서 / AGENTS.md

---

## 📌 변경 이력

| 버전 | 날짜 | 작성자 | 주요 변경 |
| --- | --- | --- |
| v1.0 | YYYY-MM-DD | Dev A | 초기 작성 |

---

## 0. Dev A 역할 핵심 선언

> **"인증이 흔들리면 전체 시스템이 흔들린다."**
> Dev A는 팀 전체가 의존하는 **인증·회원 모듈의 안정성**에 최우선 책임을 진다.
> 인증 모듈이 W1에 안정화되어야 다른 팀원들이 인증을 전제로 개발할 수 있다.

---

## 1. 소유권 선언 (Code Ownership)

```
src/main/java/{{BASE_PACKAGE}}/
├── domain/
│   ├── auth/                       ← Dev A 전담 소유
│   │   ├── controller/
│   │   │   └── AuthController.java
│   │   ├── service/
│   │   │   └── AuthService.java
│   │   ├── repository/
│   │   │   └── (Member 관련 — DB 접근)
│   │   ├── dto/
│   │   │   ├── LoginRequest.java
│   │   │   ├── LoginResponse.java
│   │   │   ├── SignUpRequest.java
│   │   │   └── MemberResponse.java
│   │   └── entity/
│   │       └── Member.java
│   │
│   └── {{MODULE_A}}/               ← Dev A 전담 소유
│       ├── controller/
│       │   └── {{ModuleA}}Controller.java
│       ├── service/
│       │   └── {{ModuleA}}Service.java
│       ├── repository/
│       │   └── {{ModuleA}}Repository.java
│       ├── dto/
│       │   ├── {{ModuleA}}Request.java
│       │   └── {{ModuleA}}Response.java
│       └── entity/
│           └── {{ModuleA}}.java
│
src/main/resources/templates/
├── auth/                           ← Dev A 전담 (Mustache 뷰)
│   ├── login.mustache
│   ├── signup.mustache
│   └── mypage.mustache
└── {{moduleA}}/                    ← Dev A 전담
    ├── {{moduleA}}-list.mustache
    ├── {{moduleA}}-detail.mustache
    └── {{moduleA}}-form.mustache

src/test/java/{{BASE_PACKAGE}}/
└── domain/
    ├── auth/                       ← Dev A 작성 (단위 + 통합)
    └── {{MODULE_A}}/               ← Dev A 작성
```

> **다른 팀원에게 제공하는 공개 인터페이스:**
> - `AuthService.getCurrentMember()` — 현재 로그인 사용자 조회 (모든 팀원 사용 가능)
> - `MemberResponse` DTO — 타 모듈에서 회원 정보 참조 시 사용
> - **변경 시 반드시 Slack #dev 채널에 사전 공지 필수**

---

## 2. Auth·회원 모듈 핵심 기술 요구사항

> **시연 시 "인증·보안을 어떻게 구현했나"는 반드시 나오는 질문이다.**
> W1에 완성되어야 다른 5명이 인증을 전제로 개발을 시작할 수 있다.
> 팀 전체가 의존하는 `getCurrentMember()` 인터페이스는 서명 확정 후 변경 금지.

| 요구사항 | 구현 방식 | 완료 목표 | 왜 중요한가 |
| --- | --- | --- | --- |
| **비밀번호 단방향 해시** | BCrypt (strength 12) — `PasswordEncoder` Bean | W1 화 | 평문 저장 시 보안 사고 |
| **Spring Security 인증 흐름** | `CustomUserDetailsService` + `UsernamePasswordAuthenticationFilter` | W1 수 | 전 팀원 인증 기반 |
| **세션 + Redis 저장** | `spring.session.store-type=redis` — Lead 설정 협의 | W1 수 | 서버 재시작 시 로그인 유지 |
| **`getCurrentMember()` 공개 메서드** | `Authentication` → 이메일 → `Member` 조회 — **서명 확정 후 변경 금지** | W1 수 확정 | Dev B/C/D 전원 의존 |
| **이메일 중복 검증** | `existsByEmail()` + `BusinessException(DUPLICATE_EMAIL)` | W1 화 | 데이터 정합성 |
| **로그인 실패 처리** | `AuthenticationFailureHandler` → `redirect:/login?error` | W1 수 | UX + 보안 |
| **로그아웃 세션 무효화** | `LogoutSuccessHandler` → Redis 세션 삭제 | W1 수 | 세션 탈취 방지 |
| **Role 기반 접근 제어** | Lead의 `SecurityConfig` URL 매핑과 협의 필수 | W1 수 | 권한 누락 시 전체 보안 실패 |

```java
// ✅ 전 팀원이 사용하는 핵심 공개 메서드 — 서명 변경 시 반드시 Slack #dev 사전 공지
public Member getCurrentMember(Authentication authentication) {
    String email = ((UserDetails) authentication.getPrincipal()).getUsername();
    return memberRepository.findByEmail(email)
        .orElseThrow(() -> new BusinessException(ErrorCode.MEMBER_NOT_FOUND));
}

// ✅ SecurityConfig 경로 매핑 — Lead와 W1 화요일까지 확정
// /auth/**    → permitAll()
// /admin/**   → hasRole("ADMIN")
// /**         → authenticated()
```

> **Dev A → 팀 전체 공지 의무:**
> - `getCurrentMember()` 서명 변경 시: 48시간 전 `#dev` 채널 공지
> - `Member` Entity 컬럼 추가·변경 시: Lead와 사전 협의 + PR에 ERD 업데이트 포함


## 3. 일별 상세 일정

### 🟦 W0 — 킥오프 참여

| 일자 | 활동 | 산출물 |
| --- | --- | --- |
| D-4 (수) | 킥오프 회의 참석. 역할 확정. 01_계획서 v1 작성 Lead와 페어. | 역할 확정, 계획서 v1 |
| D-3 (목) | 02_ERD 리뷰 + 회원(Member) 엔티티 피드백. | ERD 피드백 반영 |
| D-2 (금) | 04_API 명세서 Auth 파트 초안 작성. | Auth API 명세 초안 |
| D-1 (토) | (선택) 개인 개발 환경 세팅. `./gradlew build` 성공 확인. | 로컬 빌드 성공 |

---

### 🟩 W1 — 인증 모듈 + 모듈A 골격 구축

**Dev A W1 집중 목표:** "W1 금요일까지 로그인·회원가입이 staging에서 동작한다"

| 일자 | 오전 (수업 전) | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W1 월** | Stand-up. develop pull + 환경 세팅. | **Member Entity 작성** (Lead가 올린 공통 Entity 확인 후 시작). `MemberRepository` 작성. `@DataJpaTest` 기본 테스트 1개. | 짧은 회고. 내일 작업 결정. |
| **W1 화** | Stand-up. ERD Lock 확인. | **AuthService 구현** — 회원가입 (비밀번호 BCrypt 인코딩). `MemberService.findByEmail()`. | AuthService 단위 테스트 작성 (MockBean 사용). |
| **W1 수** | Stand-up. | **AuthController 구현** — POST /auth/signup, POST /auth/login. Lead의 SecurityConfig와 경로 맞추기. | 로그인 흐름 로컬 테스트 (Postman 또는 화면). |
| **W1 목** | Stand-up. | **Mustache 뷰 작성** — login.mustache, signup.mustache. 기본 레이아웃 적용. `{{MODULE_A}}` 골격 시작 (Entity + Repository). | 화면 로컬 확인. |
| **W1 금** | Stand-up. | 인증 통합 테스트 1개 완성 (@SpringBootTest). **W1 게이트 회의 참석 (16:00)**. PR 올리기. | W1 회고. |

**W1 Dev A 산출물 (Done — develop merge 상태)**

| 산출물 | 파일 | 완료 기준 |
| --- | --- | --- |
| Member Entity | `entity/Member.java` | 필드 완성, JPA 관계 설정 |
| 회원가입 | `POST /auth/signup` | 중복 이메일 검증 + BCrypt 저장 |
| 로그인 | `POST /auth/login` | 인증 성공 → 세션/토큰 발급 |
| 내 정보 조회 | `GET /mypage` | 인증된 사용자만 접근 |
| 로그인 화면 | `login.mustache` | 화면 렌더링 + 폼 제출 동작 |
| AuthService 단위 테스트 | `AuthServiceTest.java` | 회원가입 성공/중복실패 케이스 포함 |
| 모듈A Entity 골격 | `{{ModuleA}}.java` | 필드 + 연관관계 초안 |

**W1 금 개인 게이트 체크리스트**
```
[ ] 회원가입 → 로그인 → 보호 API 접근 전체 흐름 동작
[ ] AuthService 단위 테스트: 성공 케이스 + 실패 케이스 각 1개 이상
[ ] Repository @DataJpaTest 최소 1개 통과
[ ] 비밀번호 평문 저장 없음 (BCrypt 확인)
[ ] 로그인 화면 로컬에서 브라우저로 동작 확인
```

---

### 🟨 W2 — 모듈A 핵심 기능 완성

**Dev A W2 집중 목표:** "모듈A의 핵심 CRUD + {{MODULE_A}} 화면군 완성"

| 일자 | 오전 | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W2 월** | Stand-up. 모듈A 설계 재확인. | **{{ModuleA}}Service 핵심 로직 구현** (Create + Read). @Transactional 적용. | Service 단위 테스트 작성. |
| **W2 화** | Stand-up. API 정합 미팅(Lead 소집 시 참석). | Update + Delete + 페이징 처리. `Pageable` 적용. | Controller 작성 시작. |
| **W2 수** | Stand-up. | **{{ModuleA}}Controller 완성** — 모든 CRUD 엔드포인트. MockMvc 테스트 작성. | Mustache 뷰 작성 (list, detail, form). |
| **W2 목** | Stand-up. | 화면 완성 + 인증 연동 (currentMember 연결). 예외 처리 보강. | 통합 테스트 1개 이상. |
| **W2 금** | Stand-up. | 중간 데모 준비. **W2 게이트 회의 참석 (16:00)**. PR 최종 정리. | W2 회고. 노션 기술 블로그 주제 선정. |

**W2 Dev A 산출물**

| 산출물 | 완료 기준 |
| --- | --- |
| {{ModuleA}} CRUD 완성 | Create/Read/Update/Delete 모두 API 동작 |
| 페이징 처리 | 목록 조회에 Pageable 적용, 결과 확인 |
| 화면 완성 | list / detail / form 화면 브라우저에서 동작 |
| MockMvc 테스트 | Controller 단위 테스트 주요 시나리오 커버 |
| 서비스 커버리지 | {{ModuleA}}Service 단위 테스트 커버리지 60%+ |

---

### 🟥 W3 — 통합·마감·안정화

**Dev A W3 집중 목표:** "모듈A 완전 통합 + 인증 안정성 보장"

| 일자 | 오전 | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W3 월** | Stand-up. 통합 이슈 파악. | 다른 모듈과 연동 테스트 (인증 토큰/세션 공유 확인). | 발견된 버그 즉시 수정. |
| **W3 화** | Stand-up. Feature Freeze 선언 확인. | **기능 동결** — 버그 수정만. 회원 관련 Edge Case 처리. | 남은 테스트 보강. |
| **W3 수** | Stand-up. | 통합 테스트 전체 통과 확인. 커버리지 목표 달성 확인. | 코드 리팩터링 (AI 활용). |
| **W3 목** | Stand-up. | 시연 시나리오 관련 Auth 파트 점검. 시드 데이터 작성 지원. | 시연 리허설 준비. |
| **W3 금** | Stand-up. | **W3 게이트 회의 + 시연 1차 리허설**. | W3 회고. 블로그 초안 작성. |

**W3 Dev A 산출물**

| 산출물 | 완료 기준 |
| --- | --- |
| 인증 안정성 | 6명 모두의 모듈에서 인증 연동 정상 동작 |
| 테스트 커버리지 | Service 80%+, 전체 모듈 70%+ |
| 통합 테스트 | 핵심 시나리오 (회원가입→로그인→기능사용) E2E 통과 |

---

### ⬜ W4 — 시연 준비

| 일자 | 활동 | 산출물 |
| --- | --- | --- |
| W4 월 | 시연 시나리오 내 Auth 파트 리허설. Q&A 준비 (인증 관련 예상 질문). | Q&A 카드 작성 |
| W4 화 | 시연 리허설 1회차 참여. | 피드백 반영 |
| W4 수 | 백업 영상 녹화 참여. 노션 기술 블로그 최소 1편 완성. | 블로그 1편 |
| W4 목 | 시연 리허설 2회차. | 시연 대본 숙지 |
| W4 금 | 시연 D-Day. 회고 참여. | 회고 기여 |

---

## 4. 기술 기준 & 코딩 지침

### 3.1 인증·회원 모듈 코딩 기준

```java
// ✅ 비밀번호 인코딩 — BCrypt만 사용
@Autowired private PasswordEncoder passwordEncoder;
String encoded = passwordEncoder.encode(rawPassword);

// ✅ 현재 사용자 조회 — 공개 메서드 형식 통일
public Member getCurrentMember(Authentication authentication) {
    String email = authentication.getName();
    return memberRepository.findByEmail(email)
        .orElseThrow(() -> new BusinessException(ErrorCode.MEMBER_NOT_FOUND));
}

// ✅ 서비스 메서드 명명 규칙 (Lead의 AGENTS.md 기준)
// 동사+명사 조합: createMember(), findMemberByEmail(), updateMemberProfile()

// ✅ 트랜잭션 규칙
@Transactional          // 쓰기 작업 (create, update, delete)
@Transactional(readOnly = true)  // 읽기 작업

// ✅ 예외 처리 — BusinessException 사용
if (memberRepository.existsByEmail(email)) {
    throw new BusinessException(ErrorCode.DUPLICATE_EMAIL);
}
```

### 3.2 {{MODULE_A}} 모듈 코딩 기준

```java
// ✅ 페이징 필수 — 목록 조회는 항상 Pageable
Page<{{ModuleA}}Response> findAll{{ModuleA}}(Pageable pageable);
// → @PageableDefault(size = 10, sort = "createdAt", direction = DESC)

// ✅ DTO ↔ Entity 변환 — 서비스 레이어에서
public {{ModuleA}}Response toResponse({{ModuleA}} entity) { ... }
// Entity를 Controller나 View로 직접 넘기지 않는다.

// ✅ 소유권 검증 — 수정/삭제 시 반드시 확인
if (!resource.getOwner().equals(currentMember)) {
    throw new BusinessException(ErrorCode.FORBIDDEN);
}
```

### 3.3 Mustache 뷰 기준

```html
{{! ✅ 인증 상태 분기 }}
{{#isLoggedIn}}
  <a href="/mypage">내 정보</a>
{{/isLoggedIn}}
{{^isLoggedIn}}
  <a href="/auth/login">로그인</a>
{{/isLoggedIn}}

{{! ✅ CSRF 토큰 (POST 폼에 항상 포함) }}
<form method="post" action="/auth/signup">
  <input type="hidden" name="_csrf" value="{{csrfToken}}">
  ...
</form>
```

---

## 5. AI 에이전트 활용 가이드 (Dev A 특화)

### Dev A가 AI를 활용하면 좋은 작업

| 작업 | 도구 | 프롬프트 예시 |
| --- | --- | --- |
| Entity 작성 | Claude Code | "@docs/02_ERD.md의 Member 엔티티를 JPA로 작성. createdAt/updatedAt Audit 포함. BCrypt 비밀번호 필드 포함." |
| 단위 테스트 작성 | Claude Code | "@AuthService.java를 보고, 회원가입 단위 테스트를 작성해줘. Mockito 사용. 성공/중복이메일실패 케이스 포함. Given-When-Then 주석 필수." |
| MockMvc 테스트 | Claude Code | "@{{ModuleA}}Controller.java의 전체 CRUD에 대한 MockMvc 테스트를 작성해줘. 인증 처리는 @WithMockUser 사용." |
| Mustache 폼 | Claude Code | "Bootstrap 5 기반의 회원가입 폼 mustache를 작성해줘. CSRF 토큰 포함, 필드 검증 에러 메시지 표시." |

### Dev A가 AI에 절대 맡기지 않는 것

- 🚫 JWT 서명 키 / 세션 설정 최종 결정 (Lead 권한)
- 🚫 비밀번호 해시 알고리즘 변경
- 🚫 인증 경로 매핑 (SecurityConfig — Lead 소유)

---

## 6. 협업 포인트

| 협업 대상 | 시점 | 내용 |
| --- | --- | --- |
| **Lead** | W1 화 | Security 경로 매핑 맞추기 (SecurityConfig ↔ AuthController) |
| **Lead** | W1 수 | `AuthService.getCurrentMember()` 공개 인터페이스 확정 |
| **Dev B/C/D** | W1 금 이후 | `getCurrentMember()` 사용법 공지 + 질문 대응 |
| **Dev D** | W2 목 | 관리자 화면에서 회원 조회 API 연동 확인 |
| **DevOps/QA** | W3 초 | 인증 관련 E2E 테스트 시나리오 협의 |

---

## 7. 블로커 에스컬레이션

```
막혔을 때 행동 순서:
1. 스스로 30분 이내 해결 시도
2. 30분 이상 막히면 → #dev 채널에 "🔴 블로커: [내용]" 공지 + Lead DM
3. Lead가 60분 내 응답 없으면 → 해당 기능 잠시 보류 + 다음 작업으로 넘어감
4. 일일 회고에서 반드시 공유

인증 관련 Lead 소유 코드 수정이 필요할 때:
→ #dev 채널에 "인증 모듈 변경 요청: [내용]" 공지 → Lead 승인 후 진행
```

---

## 8. 기술 블로그 주제 가이드

| 주제 | 내용 | 어필 포인트 |
| --- | --- | --- |
| "Spring Security 인증 흐름 이해" | 인증 과정 + 커스텀 포인트 | 보안 이해도 |
| "Mockito로 단위 테스트 작성하기" | Given-When-Then 실전 예제 | 테스트 역량 |
| "Mustache + Spring MVC SSR 구현" | 뷰 렌더링 흐름 | 풀스택 경험 |
| "AI가 만든 인증 코드의 보안 허점 찾기" | 환각 사례 + 검증 과정 | 비판적 AI 활용 |

---

## 9. 일일 체크리스트 (Dev A용)

### 매일 Stand-up 전 (09:00)

```
[ ] develop 최신화 (git pull origin develop)
[ ] 어제 작업한 브랜치 CI 상태 확인
[ ] 오늘 집중할 작업 1가지 결정 (너무 많이 잡지 않기)
[ ] 리뷰 요청 대기 중인 PR 없는지 확인
```

### 코드 작성 중

```
[ ] Service 메서드 명명: 동사+명사 (Lead AGENTS.md 기준)
[ ] 쓰기 메서드: @Transactional, 읽기: @Transactional(readOnly = true)
[ ] 목록 조회: Pageable 적용했는가
[ ] 예외: BusinessException + ErrorCode 사용했는가
[ ] Entity를 Controller/View에 직접 노출하지 않았는가
```

### PR 올리기 전

```
[ ] ./gradlew test 로컬 통과
[ ] 테스트: 성공 케이스 + 실패 케이스 각 1개 이상 포함
[ ] PR 크기 400줄 이하 (초과 시 분할)
[ ] AI 보조 코드면 환각 체크 (존재하지 않는 메서드/클래스 없음)
[ ] 비밀번호 평문 / API 키 하드코딩 없음
```

---

## 10. 성공 기준 (Dev A 개인)

| 기준 | 목표 | 측정 방법 |
| --- | --- | --- |
| 인증 모듈 안정성 | W1 금 이후 인증 관련 블로커 0건 | 팀 회고 |
| {{MODULE_A}} 완성도 | W2 금 까지 CRUD 100% 동작 | 데모 확인 |
| 테스트 커버리지 | Service 80%+ | JaCoCo 리포트 |
| PR 품질 | 재작업 요청 2건 이하 | GitHub PR 이력 |
| 기술 블로그 | 노션 1편 이상 | 노션 링크 |

---

## 📋 Dev A 체크리스트 총괄

```
W1:
[ ] Member Entity + Repository develop 머지
[ ] 회원가입 + 로그인 API 동작
[ ] AuthService 단위 테스트 (성공/실패 케이스) 완료
[ ] 모듈A Entity 골격 develop 머지

W2:
[ ] 모듈A CRUD 전체 API 완성
[ ] 화면 3종 (list/detail/form) 완성
[ ] MockMvc 테스트 완료
[ ] Service 커버리지 60%+

W3:
[ ] 타 모듈과 인증 연동 테스트 통과
[ ] Service 커버리지 80%+
[ ] Edge Case 처리 완료

W4:
[ ] 시연 리허설 2회 통과
[ ] 기술 블로그 1편 발행
```
