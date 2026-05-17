# 📋 {{PROJECT_NAME}} — Dev B 상세 일정표

> **문서 버전:** v1.0
> **작성일:** YYYY-MM-DD
> **담당자:** {{DEVB_NAME}}
> **역할:** 개발자 B
> **담당 모듈:** {{MODULE_B_NAME}} — 핵심 기능 1
> **개발 기간:** W1 ~ W4
> **연관 문서:** 00_개발_일정_총괄표 / 02_ERD / 04_API_명세서 / AGENTS.md

---

## 📌 변경 이력

| 버전 | 날짜 | 작성자 | 주요 변경 |
| --- | --- | --- |
| v1.0 | YYYY-MM-DD | Dev B | 초기 작성 |

---

## 0. Dev B 역할 핵심 선언

> **"이 프로젝트의 심장은 내 모듈이다."**
> Dev B가 담당하는 **{{MODULE_B_NAME}}**은 시연 시나리오의 핵심이다.
> 기능 완성도·트랜잭션 안전성·동시성 처리가 면접과 시연에서 가장 많이 질문받는 영역임을 기억한다.

---

## 1. 소유권 선언 (Code Ownership)

```
src/main/java/{{BASE_PACKAGE}}/
└── domain/
    └── {{MODULE_B}}/               ← Dev B 전담 소유
        ├── controller/
        │   └── {{ModuleB}}Controller.java
        ├── service/
        │   └── {{ModuleB}}Service.java
        ├── repository/
        │   └── {{ModuleB}}Repository.java
        ├── dto/
        │   ├── {{ModuleB}}Request.java
        │   ├── {{ModuleB}}Response.java
        │   └── {{ModuleB}}SearchCondition.java
        └── entity/
            └── {{ModuleB}}.java

src/main/resources/templates/
└── {{moduleB}}/                    ← Dev B 전담 (Mustache 뷰)
    ├── {{moduleB}}-list.mustache
    ├── {{moduleB}}-detail.mustache
    └── {{moduleB}}-form.mustache

src/test/java/{{BASE_PACKAGE}}/
└── domain/
    └── {{MODULE_B}}/               ← Dev B 작성 (단위 + 통합)
```

> **공개 인터페이스 (다른 팀원이 사용할 수 있는 것):**
> - `{{ModuleB}}Service.find{{ModuleB}}ById(Long id)` — 타 모듈에서 참조 가능
> - `{{ModuleB}}Response` DTO
> - 변경 시 #dev 채널 사전 공지 필수

---

## 2. 이 모듈에서 반드시 구현할 기술 포인트

> 시연 시 면접관이 가장 많이 물어보는 영역. W2까지 반드시 완성한다.

| 포인트 | 왜 중요한가 | 구현 방법 |
| --- | --- | --- |
| **트랜잭션 안전성** | 핵심 기능의 데이터 정합성 | `@Transactional` 경계 명확히, 롤백 조건 정의 |
| **동시성 처리** | {{동시에 같은 자원 접근 시나리오}} | 비관적 락(`@Lock`) 또는 낙관적 락(`@Version`) |
| **중복 방지** | {{중복 발생 시나리오}} | 유니크 제약 + 서비스 레이어 검증 |
| **페이징·검색** | 대량 데이터 성능 | `Pageable` + QueryDSL 또는 JPQL |
| **상태 관리** | {{상태 변화 시나리오}} | Enum 상태값 + 상태 전이 검증 |

---

## 3. 일별 상세 일정

### 🟦 W0 — 킥오프 참여

| 일자 | 활동 | 산출물 |
| --- | --- | --- |
| D-4 (수) | 킥오프 회의 참석. 모듈B 담당 확정. | 역할 확정 |
| D-3 (목) | 02_ERD v1 작성 Lead와 페어. 모듈B 관련 엔티티 설계. | ERD 모듈B 파트 |
| D-2 (금) | 04_API 명세서 모듈B 파트 초안. 상태 Enum 정의. | API 명세 초안 |
| D-1 (토) | (선택) 개인 개발 환경 세팅. `./gradlew build` 성공 확인. | 로컬 빌드 성공 |

---

### 🟩 W1 — 모듈B 골격 구축

**Dev B W1 집중 목표:** "Entity + Repository + Service 골격 + 핵심 로직 설계 완료"

| 일자 | 오전 | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W1 월** | Stand-up. develop pull. Lead의 공통 Entity/DTO 확인. | **{{ModuleB}} Entity 작성**. 상태 Enum 정의. 연관관계 매핑. | @DataJpaTest 기본 테스트 1개. |
| **W1 화** | Stand-up. ERD Lock 확인. | **{{ModuleB}}Repository 작성**. 핵심 쿼리 메서드 정의. JPQL 또는 QueryDSL 준비. | Repository 단위 테스트 작성. |
| **W1 수** | Stand-up. | **{{ModuleB}}Service 핵심 로직 설계**. 트랜잭션 경계·동시성 전략 결정 → #dev 공유. | AI로 Service 골격 생성 후 비즈니스 로직 검토. |
| **W1 목** | Stand-up. | Service 구현 시작 (Create 메서드). 예외 조건 목록화. | Service 단위 테스트 시작 (Mockito). |
| **W1 금** | Stand-up. | **W1 게이트 회의 (16:00) 참석**. 모듈B 골격 PR 제출. | W1 회고. |

**W1 Dev B 산출물**

| 산출물 | 파일 | 완료 기준 |
| --- | --- | --- |
| Entity | `entity/{{ModuleB}}.java` | 필드, 상태 Enum, 연관관계 완성 |
| Repository | `repository/{{ModuleB}}Repository.java` | 핵심 조회 메서드 3개 이상 |
| Service 골격 | `service/{{ModuleB}}Service.java` | 메서드 시그니처 + 트랜잭션 전략 확정 |
| @DataJpaTest | 레포 테스트 | CRUD 기본 동작 확인 |

---

### 🟨 W2 — 모듈B 핵심 기능 완성

**Dev B W2 집중 목표:** "핵심 기능 전체 Happy Path 동작 + 동시성 처리 완성"

| 일자 | 오전 | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W2 월** | Stand-up. | **{{ModuleB}}Service 전체 구현** (CRUD + 상태 전이). 동시성 처리 구현. | Service 단위 테스트 집중 (Mockito). |
| **W2 화** | Stand-up. API 정합 미팅 참석 (Lead 소집 시). | **{{ModuleB}}Controller 작성** — 전체 CRUD 엔드포인트. @Valid 적용. | MockMvc 테스트 시작. |
| **W2 수** | Stand-up. | **Mustache 뷰 작성** — list, detail, form. 페이징 UI 포함. | 화면 로컬 테스트. |
| **W2 목** | Stand-up. | 화면 + 인증 연동 (Dev A의 `getCurrentMember()` 사용). 검색 기능 추가. | 통합 테스트 1개 완성. |
| **W2 금** | Stand-up. | 중간 데모 준비. **W2 게이트 회의 (16:00)**. PR 정리. | W2 회고. |

**W2 Dev B 산출물**

| 산출물 | 완료 기준 |
| --- | --- |
| 핵심 기능 전체 CRUD | Create/Read(목록+상세)/Update/Delete + 상태 전이 API 동작 |
| 동시성 처리 | 중복/경쟁 조건 시나리오 테스트 통과 |
| 화면 완성 | list / detail / form 화면 브라우저에서 동작 + 페이징 UI |
| Service 커버리지 | 60%+ (JaCoCo 확인) |
| MockMvc 테스트 | 주요 엔드포인트 전체 커버 |

**W2 특별 미션: 동시성 테스트**

```java
// 동시성 검증 예시 — 반드시 통과해야 함
@SpringBootTest
class {{ModuleB}}ConcurrencyTest {

    @Autowired {{ModuleB}}Service {{moduleBService}};
    @Autowired {{ModuleB}}Repository repository;

    @Test
    @DisplayName("동시에 같은 자원에 접근해도 중복이 발생하지 않는다")
    void 동시_접근_중복_방지() throws InterruptedException {
        // Given
        int threadCount = 10;
        ExecutorService executorService = Executors.newFixedThreadPool(threadCount);
        CountDownLatch latch = new CountDownLatch(threadCount);
        AtomicInteger successCount = new AtomicInteger(0);
        AtomicInteger failCount = new AtomicInteger(0);

        // When
        for (int i = 0; i < threadCount; i++) {
            executorService.submit(() -> {
                try {
                    {{moduleBService}}.create{{ModuleB}}(request);
                    successCount.incrementAndGet();
                } catch (BusinessException e) {
                    failCount.incrementAndGet(); // 중복 방지로 인한 실패는 정상
                } finally {
                    latch.countDown();
                }
            });
        }
        latch.await(10, TimeUnit.SECONDS);
        executorService.shutdown();

        // Then — DB 건수 = 성공 건수 (중복 없음 보장)
        long totalInDB = repository.count();
        assertThat(successCount.get() + failCount.get()).isEqualTo(threadCount);
        assertThat(totalInDB).isEqualTo(successCount.get());
        System.out.println("성공: " + successCount.get() + "건, 실패(중복방지): " + failCount.get() + "건");
    }
}
```

---

### 🟥 W3 — 통합·마감·안정화

**Dev B W3 집중 목표:** "모듈B 완전 통합 + 성능 최적화 + 포트폴리오 어필 포인트 완성"

| 일자 | 오전 | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W3 월** | Stand-up. 통합 이슈 파악. | 타 모듈과 연동 확인. N+1 쿼리 체크 + 최적화. | Fetch Join 또는 EntityGraph 적용. |
| **W3 화** | Stand-up. Feature Freeze 선언 확인. | 버그 수정만. 성능 최적화 마무리. | 남은 테스트 보강. |
| **W3 수** | Stand-up. | 통합 테스트 전체 통과 확인. 커버리지 목표 확인. | 코드 리팩터링 (AI 활용). |
| **W3 목** | Stand-up. | 시연 시나리오 점검. 시드 데이터 준비. | 시연 리허설 준비. |
| **W3 금** | Stand-up. | **W3 게이트 회의 + 시연 1차 리허설**. | W3 회고. 블로그 초안. |

**W3 Dev B 산출물**

| 산출물 | 완료 기준 |
| --- | --- |
| N+1 해결 | 쿼리 개수 확인 (Hibernate SQL 로그 또는 p6spy) |
| Service 커버리지 | 80%+ |
| 통합 테스트 | 핵심 시나리오 E2E 통과 |
| 성능 지표 | JMeter 기본 부하 테스트 결과 기록 (DevOps/QA와 협력) |

---

### ⬜ W4 — 시연 준비

| 일자 | 활동 | 산출물 |
| --- | --- | --- |
| W4 월 | 시연 시나리오 내 모듈B 파트 리허설. Q&A (동시성·트랜잭션 관련). | Q&A 카드 |
| W4 화 | 시연 리허설 1회차 참여. | 피드백 반영 |
| W4 수 | 백업 영상 녹화 참여. 노션 기술 블로그 1편 완성. | 블로그 1편 |
| W4 목 | 시연 리허설 2회차. | 시연 대본 숙지 |
| W4 금 | 시연 D-Day. 회고 참여. | 회고 기여 |

---

## 4. 기술 기준 & 코딩 지침

### 4.1 트랜잭션 기준

```java
// ✅ 모든 Service public 메서드에 명시적 트랜잭션
@Transactional                      // 쓰기
@Transactional(readOnly = true)     // 읽기

// ✅ 트랜잭션 경계 원칙
// - 하나의 비즈니스 작업 = 하나의 @Transactional
// - Controller에서 @Transactional 사용 금지
// - Repository에서 직접 트랜잭션 제어 금지 (Service가 관리)

// ✅ 롤백 전략
@Transactional(rollbackFor = BusinessException.class)
// 기본: RuntimeException만 롤백. Checked Exception은 명시 필요.
```

### 4.2 동시성 처리 기준

```java
// 비관적 락 (충돌이 빈번한 경우)
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT m FROM {{ModuleB}} m WHERE m.id = :id")
Optional<{{ModuleB}}> findByIdForUpdate(@Param("id") Long id);

// 낙관적 락 (충돌이 드문 경우)
@Version
private Long version;
// → OptimisticLockException 발생 시 재시도 로직 추가

// 선택 기준:
// - 충돌 빈도 높음 + 데이터 손실 용납 안 됨 → 비관적 락
// - 충돌 빈도 낮음 + 성능 중요 → 낙관적 락
```

### 4.3 검색·페이징 기준

```java
// ✅ 목록 조회는 항상 Pageable
@Transactional(readOnly = true)
public Page<{{ModuleB}}Response> findAll({{ModuleB}}SearchCondition condition, Pageable pageable) { ... }

// ✅ N+1 방지 — Fetch Join 또는 @EntityGraph
@EntityGraph(attributePaths = {"member", "category"})
@Query("SELECT m FROM {{ModuleB}} m WHERE ...")
Page<{{ModuleB}}> findAllWithFetch(Pageable pageable);

// ✅ 검색 조건 — 동적 쿼리 (QueryDSL 권장 또는 JPQL)
// BooleanBuilder를 이용한 조건부 Where 절
```

### 4.4 상태 전이 검증

```java
// ✅ Enum 상태 + 전이 검증
public enum {{ModuleB}}Status {
    PENDING, CONFIRMED, CANCELLED, COMPLETED;

    public boolean canTransitionTo({{ModuleB}}Status next) {
        return switch (this) {
            case PENDING -> next == CONFIRMED || next == CANCELLED;
            case CONFIRMED -> next == COMPLETED || next == CANCELLED;
            default -> false;
        };
    }
}

// Service에서 사용
if (!current.getStatus().canTransitionTo(newStatus)) {
    throw new BusinessException(ErrorCode.INVALID_STATUS_TRANSITION);
}
```

---

## 5. AI 에이전트 활용 가이드 (Dev B 특화)

| 작업 | 도구 | 프롬프트 예시 |
| --- | --- | --- |
| Entity 작성 | Claude Code | "@docs/02_ERD.md의 {{ModuleB}} 엔티티 작성. 상태 Enum, 비관적 락 @Lock 쿼리 포함." |
| Service 단위 테스트 | Claude Code | "@{{ModuleB}}Service.java의 create 메서드 단위 테스트. Mockito. 동시성 케이스 포함. Given-When-Then 주석 필수. 실패 케이스 1개 이상." |
| 동시성 테스트 | Claude Code | "ExecutorService로 {{ModuleB}} 동시 생성 테스트 작성. CountDownLatch 사용. 예상 결과: 중복 없음." |
| N+1 확인 | Claude 웹 | "이 JPQL이 N+1을 발생시키는지 분석해줘. Fetch Join으로 해결하는 코드도 작성해줘." |
| 상태 전이 검증 | Claude Code | "{{ModuleB}}Status Enum에 canTransitionTo 메서드 추가. PENDING→CONFIRMED→COMPLETED→CANCELLED 흐름 기준." |

---

## 6. 협업 포인트

| 협업 대상 | 시점 | 내용 |
| --- | --- | --- |
| **Lead** | W1 수 | 트랜잭션 전략 + 동시성 선택 (비관적/낙관적 락) 합의 |
| **Dev A** | W1 금 | `getCurrentMember()` API 사용법 확인 |
| **Dev D** | W2 목 | 관리자 화면에서 모듈B 조회 API 연동 확인 |
| **DevOps/QA** | W3 초 | 모듈B 성능 테스트 시나리오 협의 |

---

## 7. 블로커 에스컬레이션

```
막혔을 때 행동 순서:
1. 스스로 30분 이내 해결 시도 (검색, AI 보조)
2. 30분 이상 막히면 → #dev 채널에 "🔴 블로커: [내용] — [예상 영향]" 공지 + Lead DM
3. Lead 60분 내 응답 없으면 → 해당 기능 잠시 보류 + 다음 작업으로 전환
4. 일일 회고에서 반드시 공유

{{MODULE_B}} 관련 타 모듈·Lead 소유 코드 수정이 필요할 때:
→ #dev 채널에 변경 요청 공지 → 해당 소유자 승인 후 진행 (직접 수정 금지)
트랜잭션·동시성 전략 변경 시: Lead와 반드시 사전 합의 후 #dev 공지
```


## 8. 기술 블로그 주제 가이드

| 주제 | 내용 | 어필 포인트 |
| --- | --- | --- |
| "{{핵심기능}}의 동시성 처리" | 비관적/낙관적 락 선택 이유 + 테스트 | 고급 기술 이해 |
| "JPA N+1 문제 해결" | Fetch Join vs EntityGraph 비교 | 성능 최적화 경험 |
| "상태 패턴으로 {{기능}} 상태 관리" | Enum + 전이 검증 구현 | 설계 역량 |
| "트랜잭션 경계를 잘못 잡으면 생기는 일" | 실수 사례 + 해결 | 실전 경험 |

---

## 9. 일일 체크리스트 (Dev B용)

### 매일 Stand-up 전 (09:00)

```
[ ] develop 최신화 (git pull origin develop)
[ ] 어제 작업 CI 상태 확인
[ ] 오늘 집중 작업 1가지 결정
[ ] 트랜잭션 경계 변경 사항 없는지 확인
```

### 코드 작성 중

```
[ ] 모든 Service public 메서드: @Transactional 적용
[ ] 목록 조회: Pageable 적용
[ ] 상태 전이: canTransitionTo() 검증 거쳤는가
[ ] 동시성 시나리오: 락 전략 적용했는가
[ ] 예외: BusinessException + ErrorCode 사용
[ ] Entity를 View/Controller에 직접 노출 안 했는가
```

### PR 올리기 전

```
[ ] ./gradlew test 로컬 통과
[ ] 동시성 테스트 포함했는가
[ ] N+1 쿼리 없음 확인 (Hibernate SQL 로그)
[ ] PR 크기 400줄 이하
[ ] AI 코드 환각 체크 완료
```

---

## 10. 성공 기준 (Dev B 개인)

| 기준 | 목표 | 측정 방법 |
| --- | --- | --- |
| 핵심 기능 완성도 | W2 금까지 CRUD 100% 동작 | 데모 확인 |
| 동시성 처리 | 동시 접근 테스트 통과 | 단위 테스트 |
| Service 커버리지 | 80%+ | JaCoCo |
| N+1 제거 | 목록 조회 쿼리 최대 3개 이내 | SQL 로그 |
| 기술 블로그 | 1편 이상 | 노션 링크 |

---

## 📋 Dev B 체크리스트 총괄

```
W1:
[ ] Entity + Enum + Repository develop 머지
[ ] Service 골격 + 트랜잭션 전략 확정 공지
[ ] @DataJpaTest 기본 동작 확인

W2:
[ ] 핵심 기능 CRUD 전체 API 완성
[ ] 동시성 처리 구현 + 테스트 통과
[ ] 화면 3종 완성
[ ] Service 커버리지 60%+

W3:
[ ] N+1 해결 확인
[ ] Service 커버리지 80%+
[ ] 통합 E2E 테스트 통과

W4:
[ ] 시연 리허설 2회 통과
[ ] 기술 블로그 1편 발행
```
