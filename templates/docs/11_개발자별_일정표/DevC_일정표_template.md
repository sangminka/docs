# 📋 {{PROJECT_NAME}} — Dev C 상세 일정표

> **문서 버전:** v1.0
> **작성일:** YYYY-MM-DD
> **담당자:** {{DEVC_NAME}}
> **역할:** 개발자 C
> **담당 모듈:** {{MODULE_C_NAME}} — 핵심 기능 2
> **개발 기간:** W1 ~ W4
> **연관 문서:** 00_개발_일정_총괄표 / 02_ERD / 04_API_명세서 / AGENTS.md

---

## 📌 변경 이력

| 버전 | 날짜 | 작성자 | 주요 변경 |
| --- | --- | --- |
| v1.0 | YYYY-MM-DD | Dev C | 초기 작성 |

---

## 0. Dev C 역할 핵심 선언

> **"외부 연동과 알림이 없으면 서비스가 살아있지 않다."**
> Dev C가 담당하는 **{{MODULE_C_NAME}}**은 시스템이 실제로 동작하고 있음을 사용자에게 보여주는 모듈이다.
> 외부 API 연동({{EXTERNAL_SERVICE}}) 또는 핵심 서비스 로직을 안정적으로 구현하는 것이 목표다.

---

## 1. 소유권 선언 (Code Ownership)

```
src/main/java/{{BASE_PACKAGE}}/
└── domain/
    └── {{MODULE_C}}/               ← Dev C 전담 소유
        ├── controller/
        │   └── {{ModuleC}}Controller.java
        ├── service/
        │   └── {{ModuleC}}Service.java
        ├── repository/
        │   └── {{ModuleC}}Repository.java
        ├── dto/
        │   ├── {{ModuleC}}Request.java
        │   └── {{ModuleC}}Response.java
        └── entity/
            └── {{ModuleC}}.java

src/main/resources/templates/
└── {{moduleC}}/                    ← Dev C 전담 (Mustache 뷰)
    ├── {{moduleC}}-list.mustache
    ├── {{moduleC}}-detail.mustache
    └── {{moduleC}}-form.mustache

src/test/java/{{BASE_PACKAGE}}/
└── domain/
    └── {{MODULE_C}}/               ← Dev C 작성
```

> **공개 인터페이스:**
> - `{{ModuleC}}Service.findBy{{관련키}}({{타입}} key)` — 타 모듈 참조 가능
> - 변경 시 #dev 사전 공지 필수

---

## 2. 이 모듈의 핵심 기술 과제

| 과제 | 난이도 | 완료 목표 |
| --- | --- | --- |
| {{MODULE_C}} 핵심 비즈니스 로직 | 높음 | W2 수 |
| 외부 서비스 연동 또는 이벤트 처리 | 중간 | W2 목 |
| 알림/이메일/비동기 처리 | 중간 | W2 금 |
| 검색·필터·집계 기능 | 중간 | W2 금 |
| 통합 테스트 | 낮음 | W3 수 |

---

## 3. 일별 상세 일정

### 🟦 W0 — 킥오프 참여

| 일자 | 활동 | 산출물 |
| --- | --- | --- |
| D-4 (수) | 킥오프 회의 참석. 모듈C 담당 확정. | 역할 확정 |
| D-3 (목) | ERD 모듈C 파트 설계 참여. | ERD 모듈C 파트 |
| D-2 (금) | API 명세서 모듈C 파트 초안. 외부 연동 필요 API 목록 정리. | API 명세 초안 |
| D-1 (토) | 개발 환경 세팅. 외부 API Key 발급 확인. | 로컬 빌드 성공 |

---

### 🟩 W1 — 모듈C 골격 + 외부 연동 준비

**Dev C W1 집중 목표:** "Entity + Repository + Service 골격 완성 + 외부 연동 설계 확정"

| 일자 | 오전 | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W1 월** | Stand-up. develop pull + 공통 레이어 확인. | **{{ModuleC}} Entity 작성**. 연관관계 설계. | @DataJpaTest 기본 1개. |
| **W1 화** | Stand-up. ERD Lock 확인. | **Repository 작성**. 핵심 쿼리 정의. | Repository 테스트. |
| **W1 수** | Stand-up. | **Service 설계** — 외부 연동 방식 결정. Lead의 `external/` 패키지와 협의. | 외부 연동 설계 #dev 공유. |
| **W1 목** | Stand-up. | Service 구현 시작. Mock 외부 API로 단위 테스트 구조 잡기. | Mockito Mock 구조 완성. |
| **W1 금** | Stand-up. | **W1 게이트 회의 (16:00)**. 골격 PR 제출. | W1 회고. |

**W1 Dev C 산출물**

| 산출물 | 완료 기준 |
| --- | --- |
| Entity | 필드, 연관관계, 제약조건 완성 |
| Repository | 핵심 조회 3개 이상 |
| Service 골격 | 메서드 시그니처 + 외부 연동 전략 확정 |
| Mock 외부 API 구조 | 실제 연동 전 Mock으로 Service 테스트 가능한 구조 |

---

### 🟨 W2 — 모듈C 핵심 기능 + 외부 연동 완성

**Dev C W2 집중 목표:** "외부 연동 완성 + 화면 완성 + 테스트 커버리지 60%+"

| 일자 | 오전 | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W2 월** | Stand-up. | **{{ModuleC}}Service 핵심 로직 구현**. 외부 API 실제 호출 연동 (Lead의 Client 활용). | 실제 응답 처리 로직. |
| **W2 화** | Stand-up. | 외부 API 오류 처리 — 타임아웃, 재시도, 폴백 전략. | 폴백 단위 테스트. |
| **W2 수** | Stand-up. | **{{ModuleC}}Controller 작성** + **Mustache 뷰 작성** (list, detail). | 화면 로컬 확인. |
| **W2 목** | Stand-up. | 폼 화면 + 인증 연동. 알림/이메일 처리 구현 (해당 시). | 통합 테스트 1개. |
| **W2 금** | Stand-up. | 중간 데모. **W2 게이트 회의 (16:00)**. | W2 회고. |

**W2 Dev C 산출물**

| 산출물 | 완료 기준 |
| --- | --- |
| 외부 연동 완성 | 실제 호출 성공 + 폴백 동작 |
| 핵심 CRUD 완성 | 전체 API Happy Path 동작 |
| 화면 완성 | list / detail / form 브라우저 동작 |
| Service 커버리지 | 60%+ (외부 API는 Mock으로 테스트) |

**W2 특별 규칙: 외부 API 테스트**

```java
// ✅ 외부 API는 반드시 Mock으로 단위 테스트
@ExtendWith(MockitoExtension.class)
class {{ModuleC}}ServiceTest {

    @Mock
    private {{ExternalApiClient}} externalClient;

    @InjectMocks
    private {{ModuleC}}Service service;

    @Test
    @DisplayName("외부 API 정상 응답 시 정상 처리")
    void 외부_API_정상_응답() {
        // Given
        given(externalClient.call(any())).willReturn(successResponse);

        // When
        var result = service.process(request);

        // Then
        assertThat(result).isNotNull();
    }

    @Test
    @DisplayName("외부 API 실패 시 폴백 처리")
    void 외부_API_실패_폴백() {
        // Given
        given(externalClient.call(any())).willThrow(new ExternalApiException());

        // When
        var result = service.processWithFallback(request);

        // Then — 폴백 결과 검증
        assertThat(result.isFallback()).isTrue();
    }
}
```

---

### 🟥 W3 — 통합·마감·안정화

**Dev C W3 집중 목표:** "모듈C 완전 통합 + 외부 연동 안정성 검증"

| 일자 | 오전 | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W3 월** | Stand-up. 통합 이슈 파악. | 타 모듈과 연동 확인. 외부 API 실패 시나리오 추가 테스트. | 발견된 버그 즉시 수정. |
| **W3 화** | Stand-up. Feature Freeze 확인. | 버그 수정만. Edge Case 처리. | 테스트 보강. |
| **W3 수** | Stand-up. | 통합 테스트 전체 통과. 커버리지 확인. | 리팩터링 (AI 활용). |
| **W3 목** | Stand-up. | 시연 시나리오 점검. 시드 데이터 준비. | 시연 리허설 준비. |
| **W3 금** | Stand-up. | **W3 게이트 + 시연 1차 리허설**. | W3 회고. 블로그 초안. |

**W3 Dev C 산출물**

| 산출물 | 완료 기준 |
| --- | --- |
| 외부 연동 안정성 | 타임아웃·실패 시 폴백 정상 동작 확인 |
| Service 커버리지 | 80%+ |
| 통합 테스트 | 핵심 시나리오 (외부 연동 포함) E2E 통과 |

---

### ⬜ W4 — 시연 준비

| 일자 | 활동 | 산출물 |
| --- | --- | --- |
| W4 월 | Q&A 준비 (외부 연동·폴백·비동기 관련). | Q&A 카드 |
| W4 화 | 시연 리허설 1회차. | 피드백 반영 |
| W4 수 | 백업 영상 녹화. 기술 블로그 1편 완성. | 블로그 1편 |
| W4 목 | 시연 리허설 2회차. | 대본 숙지 |
| W4 금 | 시연 D-Day. 회고 참여. | 회고 기여 |

---

## 4. 기술 기준 & 코딩 지침

### 4.1 외부 API 연동 기준

```java
// ✅ Lead가 제공하는 Client 활용
@Service
@RequiredArgsConstructor
public class {{ModuleC}}Service {

    private final {{ExternalApi}}Client externalClient; // Lead 소유
    private final {{ModuleC}}Repository repository;

    // ✅ 타임아웃 + 재시도 + 폴백 패턴
    public {{ModuleC}}Response processWithFallback({{ModuleC}}Request request) {
        try {
            var externalResult = externalClient.call(request.toExternalRequest());
            return process(externalResult);
        } catch (ExternalApiException | TimeoutException e) {
            log.warn("외부 API 실패. 폴백 처리: {}", e.getMessage());
            return processFallback(request);
        }
    }

    // ✅ 폴백 로직 — 시스템이 완전 멈추지 않도록
    private {{ModuleC}}Response processFallback({{ModuleC}}Request request) {
        // 최소 기능 유지 or 대기 상태 저장
    }
}

// ✅ 외부 API는 단위 테스트에서 반드시 Mock 처리 (실제 호출 금지)
```

### 4.2 비동기 처리 (해당 시)

```java
// ✅ 이메일·알림 등 비동기 처리
@Async
public CompletableFuture<Void> sendNotification(NotificationRequest request) {
    // 비동기 작업
    return CompletableFuture.runAsync(() -> {
        notificationSender.send(request);
    });
}

// ✅ @Async 설정 — @EnableAsync가 있는 Config 클래스 확인 (Lead 소유)
// ✅ 비동기 실패 시 → 로그 + 재시도 또는 Dead Letter 처리
```

### 4.3 검색·집계 기준

```java
// ✅ 동적 검색 조건 — QueryDSL 또는 JPQL Criteria
public Page<{{ModuleC}}Response> search({{ModuleC}}SearchCondition cond, Pageable pageable) {
    BooleanBuilder builder = new BooleanBuilder();
    if (StringUtils.hasText(cond.getKeyword())) {
        builder.and(q{{ModuleC}}.name.contains(cond.getKeyword()));
    }
    // 날짜 범위, 상태 필터 등 추가
    return repository.findAll(builder, pageable).map(this::toResponse);
}
```

---

## 5. AI 에이전트 활용 가이드 (Dev C 특화)

| 작업 | 도구 | 프롬프트 예시 |
| --- | --- | --- |
| 외부 API Mock 테스트 | Claude Code | "@{{ModuleC}}Service.java의 processWithFallback 메서드 단위 테스트. Mockito로 외부 API 성공/실패 케이스 각각. Given-When-Then 주석 필수." |
| 폴백 로직 | Claude 웹 | "{{외부 API}}가 실패했을 때 폴백으로 {{대안}}을 제공하는 Spring Service 메서드를 작성해줘. try-catch, 로깅 포함." |
| 동적 검색 | Claude Code | "JPQL로 {{ModuleC}} 동적 검색 구현. 조건: keyword(부분일치), status(상태), dateRange(날짜 범위). Pageable 적용." |
| Mustache 뷰 | Claude Code | "Bootstrap 5 기반 {{moduleC}} 목록 화면 작성. 검색 폼 + 테이블 + 페이징 UI 포함." |

---

## 6. 협업 포인트

| 협업 대상 | 시점 | 내용 |
| --- | --- | --- |
| **Lead** | W1 수 | 외부 API Client (`external/` 패키지) 사용 방법 확인 |
| **Dev A** | W1 금 | `getCurrentMember()` 사용법 확인 |
| **Dev B** | W2 목 | 모듈B ↔ 모듈C 연동 데이터 확인 |
| **Dev D** | W2 목 | 관리자 화면에서 모듈C 조회 API 확인 |
| **DevOps/QA** | W3 초 | 외부 연동 E2E 테스트 시나리오 협의 |

---

## 7. 블로커 에스컬레이션

```
막혔을 때 행동 순서:
1. 스스로 30분 이내 해결 시도 (검색, AI 보조)
2. 30분 이상 막히면 → #dev 채널에 "🔴 블로커: [내용] — [예상 영향]" 공지 + Lead DM
3. Lead 60분 내 응답 없으면 → 해당 기능 잠시 보류 + 다음 작업으로 전환
4. 일일 회고에서 반드시 공유

{{MODULE_C}}·외부 연동 관련 타 모듈·Lead 소유 코드 수정이 필요할 때:
→ #dev 채널에 변경 요청 공지 → 해당 소유자 승인 후 진행 (직접 수정 금지)
외부 API Client(`external/` 패키지) 변경 시: Lead 승인 필수
```


## 8. 기술 블로그 주제 가이드

| 주제 | 내용 | 어필 포인트 |
| --- | --- | --- |
| "외부 API 폴백 전략 구현" | Circuit Breaker 없이 폴백 만들기 | 운영 마인드 |
| "비동기(@Async) 처리 실전" | 알림 비동기화 + 실패 처리 | 성능 최적화 |
| "AI로 Mock 테스트 작성한 경험" | 좋은 프롬프트 vs 환각 사례 | AI 활용 역량 |
| "Spring에서 동적 검색 구현" | JPQL vs QueryDSL 비교 | 기술 선택 경험 |

---

## 9. 일일 체크리스트 (Dev C용)

### 매일 Stand-up 전 (09:00)

```
[ ] develop 최신화 (git pull origin develop)
[ ] 어제 작업한 브랜치 CI 상태 확인
[ ] 오늘 집중할 작업 1가지 결정 (너무 많이 잡지 않기)
[ ] 리뷰 요청 대기 중인 PR 없는지 확인
```

### 코드 작성 중

```
[ ] 외부 API 호출: 타임아웃 + 재시도 + 폴백 전략 있는가
[ ] 외부 API 단위 테스트: Mock 사용 (실제 호출 없이)
[ ] 서비스 메서드: @Transactional 적용
[ ] 목록 조회: Pageable 적용
[ ] 예외: BusinessException + ErrorCode
```

### PR 올리기 전

```
[ ] ./gradlew test 로컬 통과
[ ] 외부 API 실패 케이스 테스트 포함
[ ] PR 크기 400줄 이하
[ ] AI 코드 환각 체크 (존재하지 않는 메서드 없음)
[ ] API Key / 비밀번호 평문 하드코딩 없음
[ ] 테스트: 성공 케이스 + 실패 케이스(외부 API 포함) 각 1개 이상
```

---

## 10. 성공 기준 (Dev C 개인)

| 기준 | 목표 | 측정 방법 |
| --- | --- | --- |
| 외부 연동 안정성 | 폴백 포함 정상 동작 | 테스트 + 시연 |
| 핵심 기능 완성도 | W2 금까지 CRUD 100% | 데모 확인 |
| Service 커버리지 | 80%+ | JaCoCo |
| 기술 블로그 | 1편 이상 | 노션 링크 |

---

## 📋 Dev C 체크리스트 총괄

```
W1:
[ ] Entity + Repository develop 머지
[ ] 외부 연동 설계 확정 + #dev 공유
[ ] Mock 외부 API 테스트 구조 완성

W2:
[ ] 외부 연동 실제 동작 (폴백 포함)
[ ] 핵심 CRUD 전체 API 완성
[ ] 화면 3종 완성
[ ] Service 커버리지 60%+

W3:
[ ] 외부 API 실패 시나리오 통합 테스트 통과
[ ] Service 커버리지 80%+
[ ] 통합 E2E 통과

W4:
[ ] 시연 리허설 2회 통과
[ ] 기술 블로그 1편 발행
```
