# {{PROJECT_NAME}} — 테스트 전략서

> **문서 버전:** v1.0
> **작성일:** YYYY-MM-DD
> **연관 문서:** 07_요구사항_정의서 / 04_API_명세서 / 03_아키텍처_정의서 / 20_CICD_파이프라인

---

## 📌 변경 이력

| 버전 | 날짜 | 작성자 | 주요 변경 |
| --- | --- | --- | --- |
| v1.0 | YYYY-MM-DD | DevOps/QA | 초기 작성 |

---

## 목차
1. 테스트 목표 및 범위
2. 테스트 도구
3. 테스트 유형별 전략
4. 동시성 테스트 전략
5. 성능(부하) 테스트 전략
6. 테스트 커버리지 목표 & JaCoCo 설정
7. 담당자별 테스트 범위
8. 외부 연동 테스트 전략
9. 주차별 테스트 일정
10. CI 연동
11. 테스트 작성 규칙

---

## 1. 테스트 목표 및 범위

### 1.1 테스트 목표
- 핵심 비즈니스 규칙 보장
- 회귀 방지 (CI에서 자동 감지)
- 동시성·성능 이슈 시연 전 선제 검출
- 코드 리뷰 시 변경 영향도 즉시 확인

### 1.2 테스트 범위

| 범위 | 포함 | 제외 |
| --- | --- | --- |
| Unit | Service 계층 전체 | SSR 템플릿 렌더링 |
| Repository | JPA 쿼리 정합, 상태 조회 | 외부 운영 DB |
| Controller | MockMvc 기반 (인증·인가 포함) | 실제 세션/CSRF 전 과정 |
| 동시성 | 핵심 자원 경쟁 시나리오 | 복합 트랜잭션 시뮬레이션 |
| 통합 | 핵심 E2E 시나리오 | 외부 API 실제 호출 |
| 성능 | 핵심 API P95 측정, 스파이크 | 전체 화면 부하 |

---

## 2. 테스트 도구

### 2.1 Gradle 의존성 (build.gradle)

```kotlin
// build.gradle.kts
dependencies {
    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testImplementation("org.springframework.security:spring-security-test")
    testImplementation("com.h2database:h2")
    testImplementation("org.mockito:mockito-inline")
    // 필요 시: Testcontainers
    // testImplementation("org.testcontainers:mysql:1.19.3")
    // testImplementation("org.testcontainers:junit-jupiter:1.19.3")
}

tasks.withType<Test> {
    useJUnitPlatform()
    // 병렬 테스트 실행 (선택)
    systemProperty("junit.jupiter.execution.parallel.enabled", "true")
    systemProperty("junit.jupiter.execution.parallel.mode.default", "concurrent")
}
```

### 2.2 도구별 역할

| 도구 | 역할 | 담당 |
| --- | --- | --- |
| JUnit 5 + Mockito | Service 단위 테스트 | 각 Dev |
| @DataJpaTest | Repository 단위 테스트 | 각 Dev |
| @WebMvcTest + MockMvc | Controller 단위 테스트 | 각 Dev |
| @SpringBootTest | 통합 테스트, E2E | DevOps/QA + 각 Dev |
| ExecutorService + CountDownLatch | 동시성 테스트 | 각 Dev (핵심 모듈) |
| JMeter / k6 | 성능(부하) 테스트 | DevOps/QA |
| JaCoCo | 커버리지 측정 + 게이트 | DevOps/QA |

---

## 3. 테스트 유형별 전략

### 3.1 단위 테스트 — Service

```java
@ExtendWith(MockitoExtension.class)
class {{Resource}}ServiceTest {

    @Mock {{Resource}}Repository repository;
    @Mock {{External}}Client externalClient;  // 외부 연동 있으면
    @InjectMocks {{Resource}}Service service;

    @Test
    @DisplayName("정상 생성 - 유효한 입력이면 저장 후 응답 반환")
    void create_withValidInput_success() {
        // Given
        var request = {{Resource}}Request.of(...);
        given(repository.save(any())).willReturn({{Resource}}.of(...));

        // When
        var result = service.create(request);

        // Then
        assertThat(result.getId()).isNotNull();
        verify(repository, times(1)).save(any());
    }

    @Test
    @DisplayName("중복 생성 - 이미 존재하면 BusinessException 발생")
    void create_withDuplicateKey_throwsBusinessException() {
        // Given
        given(repository.existsByKey(any())).willReturn(true);

        // When & Then
        assertThatThrownBy(() -> service.create(request))
            .isInstanceOf(BusinessException.class)
            .hasMessageContaining("중복");
    }
}
```

### 3.2 Repository 테스트 (`@DataJpaTest`)

```java
@DataJpaTest
@ActiveProfiles("test")
class {{Resource}}RepositoryTest {

    @Autowired {{Resource}}Repository repository;
    @Autowired TestEntityManager em;

    @Test
    @DisplayName("상태별 조회 - PENDING만 반환")
    void findByStatus_returnsPendingOnly() {
        // Given - 테스트 데이터 삽입
        em.persist({{Resource}}.of(Status.PENDING));
        em.persist({{Resource}}.of(Status.CONFIRMED));
        em.flush();

        // When
        var result = repository.findByStatus(Status.PENDING);

        // Then
        assertThat(result).hasSize(1);
        assertThat(result.get(0).getStatus()).isEqualTo(Status.PENDING);
    }

    @Test
    @DisplayName("페이징 - 최신순 20건 조회")
    void findAll_withPageable_returnsPagedResult() {
        // Given
        IntStream.range(0, 25).forEach(i -> em.persist({{Resource}}.of()));
        em.flush();

        // When
        var page = repository.findAll(PageRequest.of(0, 20, Sort.by("createdAt").descending()));

        // Then
        assertThat(page.getContent()).hasSize(20);
        assertThat(page.getTotalElements()).isEqualTo(25);
    }
}
```

### 3.3 Controller 테스트 (MockMvc)

```java
@WebMvcTest({{Resource}}Controller.class)
@Import(SecurityConfig.class)
class {{Resource}}ControllerTest {

    @Autowired MockMvc mockMvc;
    @MockBean {{Resource}}Service service;
    @Autowired ObjectMapper objectMapper;

    @Test
    @WithMockUser(roles = "{{ROLE_A}}")
    @DisplayName("목록 조회 - 인증된 ROLE_A면 200 반환")
    void list_withAuthorizedUser_returns200() throws Exception {
        // Given
        given(service.findAll(any())).willReturn(Page.empty());

        // When & Then
        mockMvc.perform(get("/{{role_a}}/{{resource}}"))
            .andExpect(status().isOk())
            .andExpect(view().name("{{role_a}}/{{resource}}/list"));
    }

    @Test
    @WithMockUser(roles = "{{ROLE_B}}")
    @DisplayName("타 권한 접근 - ROLE_B면 403 반환")
    void list_withUnauthorizedRole_returns403() throws Exception {
        mockMvc.perform(get("/{{role_a}}/{{resource}}"))
            .andExpect(status().isForbidden());
    }

    @Test
    @WithMockUser
    @DisplayName("생성 - 유효하지 않은 입력이면 다시 폼 반환")
    void create_withInvalidInput_returnsFormView() throws Exception {
        mockMvc.perform(post("/{{role_a}}/{{resource}}")
                .param("field", "")  // 빈 값
                .with(csrf()))
            .andExpect(status().isOk())
            .andExpect(view().name("{{role_a}}/{{resource}}/form"))
            .andExpect(model().attributeHasFieldErrors("{{resource}}Form", "field"));
    }
}
```

### 3.4 통합 테스트 (`@SpringBootTest`)

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
@Transactional
class {{Scenario}}IntegrationTest {

    @Autowired TestRestTemplate restTemplate;

    @Test
    @Order(1)
    @DisplayName("전체 여정 - 회원가입 → 로그인 → 핵심 기능 사용")
    void fullUserJourney() {
        // Given: 사용자 정보
        // When: 회원가입
        // When: 로그인
        // When: 핵심 기능 호출
        // Then: 기대 결과 검증
    }
}
```

---

## 4. 동시성 테스트 전략

> **이 섹션은 Dev B(핵심기능1) 담당이 주도. DevOps/QA가 CI 연동.**
> 동시성 이슈는 단위 테스트로 잡히지 않는다. 반드시 별도 동시성 테스트를 작성한다.

### 4.1 동시성 테스트 패턴

```java
@SpringBootTest
class {{Resource}}ConcurrencyTest {

    @Autowired {{Resource}}Service service;
    @Autowired {{Resource}}Repository repository;

    @Test
    @DisplayName("동시 N건 생성 - 중복 발생 없이 정합성 보장")
    void concurrentCreate_noDataRace() throws InterruptedException {
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
                    service.create(sharedResource);  // 동일 자원에 동시 접근
                    successCount.incrementAndGet();
                } catch (BusinessException e) {
                    failCount.incrementAndGet();  // 중복 방지로 인한 실패는 OK
                } finally {
                    latch.countDown();
                }
            });
        }
        latch.await(10, TimeUnit.SECONDS);
        executorService.shutdown();

        // Then
        long totalInDB = repository.count();
        assertThat(totalInDB).isEqualTo(successCount.get());  // DB 건수 = 성공 건수
        assertThat(successCount.get() + failCount.get()).isEqualTo(threadCount);
    }
}
```

### 4.2 동시성 테스트 필수 시나리오

| 시나리오 | 담당 모듈 | 테스트 클래스명 |
| --- | --- | --- |
| 같은 자원에 동시 N건 생성 → 중복 방지 | {{MODULE_B}} | `{{Resource}}ConcurrencyTest` |
| 상태 전이 동시 요청 → 이중 처리 방지 | {{MODULE_B}}, {{MODULE_C}} | `{{Resource}}StatusConcurrencyTest` |
| 관리자 + 일반 사용자 동시 수정 | Admin | `AdminConcurrencyTest` |

---

## 5. 성능(부하) 테스트 전략

> **담당: DevOps/QA. 실행 시점: W3 화요일.**

### 5.1 목표 지표

| 지표 | 목표 | 측정 방법 |
| --- | --- | --- |
| 응답시간 P95 | ≤ 500ms | k6 / JMeter |
| 응답시간 P99 | ≤ 1000ms | k6 / JMeter |
| 에러율 | ≤ 1% | k6 / JMeter |
| TPS (핵심 API) | {{N}} TPS | k6 |
| 동시 사용자 | {{N}}명 | k6 |

### 5.2 k6 스크립트 기본 구조

```javascript
// k6-script.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate } from 'k6/metrics';

const errorRate = new Rate('errors');

export const options = {
  stages: [
    { duration: '1m', target: 10 },   // 워밍업
    { duration: '3m', target: 50 },   // 목표 부하
    { duration: '1m', target: 100 },  // 스파이크
    { duration: '1m', target: 0 },    // 쿨다운
  ],
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1000'],
    errors: ['rate<0.01'],
  },
};

export default function () {
  // 로그인
  const loginRes = http.post(`${__ENV.BASE_URL}/login`, {
    username: 'test@example.com',
    password: 'password',
  });
  check(loginRes, { 'login 200': (r) => r.status === 200 });

  // 핵심 API 호출
  const res = http.get(`${__ENV.BASE_URL}/api/{{resource}}`, {
    headers: { Cookie: loginRes.cookies.SESSION },
  });
  errorRate.add(res.status !== 200);
  check(res, { '200': (r) => r.status === 200 });

  sleep(1);
}
```

```bash
# 실행
k6 run --env BASE_URL=https://staging.example.com k6-script.js

# 결과 HTML 리포트
k6 run --out json=result.json k6-script.js
k6 report result.json
```

### 5.3 JMeter 사용 시 (GUI 대신 CLI)

```bash
jmeter -n -t {{project}}-load-test.jmx \
  -l result.jtl \
  -e -o report-dir/ \
  -JBASE_URL=https://staging.example.com \
  -JTHREADS=50 \
  -JDURATION=180
```

---

## 6. 테스트 커버리지 목표 & JaCoCo 설정

### 6.1 커버리지 목표

| 레이어 | W1 목표 | W2 목표 | W3 목표 |
| --- | --- | --- | --- |
| Repository | 30%+ | 50%+ | 70%+ |
| Service | 30%+ | 60%+ | 80%+ |
| Controller | - | 40%+ | 60%+ |
| 전체 | - | - | 70%+ |

### 6.2 JaCoCo 게이트 설정 (build.gradle.kts)

```kotlin
// build.gradle.kts
plugins {
    jacoco
}

jacoco {
    toolVersion = "0.8.11"
}

tasks.jacocoTestReport {
    dependsOn(tasks.test)
    reports {
        xml.required = true    // CI 연동용
        html.required = true   // 로컬 확인용
        csv.required = false
    }
    // Service 레이어만 측정
    classDirectories.setFrom(
        files(classDirectories.files.map {
            fileTree(it) { exclude("**/config/**", "**/dto/**", "**/entity/**") }
        })
    )
}

tasks.jacocoTestCoverageVerification {
    dependsOn(tasks.jacocoTestReport)
    violationRules {
        rule {
            limit {
                counter = "INSTRUCTION"
                value = "COVEREDRATIO"
                minimum = "0.80".toBigDecimal()  // 80% — W3 목표
            }
            includes = listOf("**/service/**")
        }
    }
}

// CI에서 test 실행 시 커버리지도 함께 측정
tasks.test {
    finalizedBy(tasks.jacocoTestReport)
}
```

### 6.3 핵심 클래스 커버리지 목표

| 클래스 | 목표 | 이유 |
| --- | --- | --- |
| `SecurityConfig` | 90%+ | 인증·인가 핵심 |
| `{{Resource}}Service` (핵심 모듈) | 90%+ | 포트폴리오 어필 포인트 |
| `{{External}}Client` | 80%+ (Mock 기반) | 폴백 경로 검증 |
| `GlobalExceptionHandler` | 80%+ | 에러 처리 전 경로 |

---

## 7. 담당자별 테스트 범위

| 담당자 | 역할 | 테스트 범위 | 특이사항 |
| --- | --- | --- | --- |
| **Lead** | Security·공유 레이어 | SecurityConfig, GlobalExceptionHandler, 외부 API Client | 인증 통합 테스트 주도 |
| **Dev A** | Auth + {{MODULE_A}} | AuthService, {{ModuleA}}Service, 연관 Repository·Controller | 인증 Mock 제공 |
| **Dev B** | {{MODULE_B}} | {{ModuleB}}Service, **동시성 테스트** | 비관적/낙관적 락 검증 필수 |
| **Dev C** | {{MODULE_C}} | {{ModuleC}}Service (외부 API Mock), 폴백 테스트 | 실제 외부 API 호출 금지 |
| **Dev D** | Admin + Stats | AdminService, StatsService, 관리자 권한 테스트 | 타 모듈 Mock 처리 |
| **DevOps/QA** | 통합·성능 | E2E 통합 테스트, k6/JMeter 부하 테스트, CI 커버리지 게이트 | W3 집중 |

---

## 8. 외부 연동 테스트 전략

### 원칙
- **테스트에서 실제 외부 API 호출 금지**
- 단위 테스트: `@MockBean` + Mockito
- 통합 테스트: WireMock (선택) 또는 `@MockBean`

### Mock 처리 패턴

```java
@MockBean {{External}}Client client;

@Test
@DisplayName("외부 API 정상 응답 - 결과 저장 후 응답")
void process_withExternalSuccess_savesResult() {
    // Given
    given(client.call(any())).willReturn(ExternalResponse.success(...));

    // When
    var result = service.process(request);

    // Then
    assertThat(result.getStatus()).isEqualTo(Status.CONFIRMED);
    verify(repository, times(1)).save(any());
}

@Test
@DisplayName("외부 API 타임아웃 - 폴백 경로 진입")
void process_withExternalTimeout_usesFallback() {
    // Given
    given(client.call(any())).willThrow(new TimeoutException("3s timeout"));

    // When
    var result = service.processWithFallback(request);

    // Then
    assertThat(result.isFallback()).isTrue();
    assertThat(result.getStatus()).isEqualTo(Status.PENDING);  // 폴백: 대기 저장
}
```

### 시나리오 커버리지

| 시나리오 | 테스트 여부 |
| --- | --- |
| 정상 응답 | ✅ 필수 |
| 타임아웃 | ✅ 필수 |
| 4xx (잘못된 요청) | ✅ 필수 |
| 5xx (서버 오류) | ✅ 필수 |
| 잘못된 JSON 응답 | 권장 |
| Rate Limit | 권장 |

---

## 9. 주차별 테스트 일정

| 주차 | 활동 | 목표 커버리지 | 담당 |
| --- | --- | --- | --- |
| W0 | JaCoCo 설정, CI 커버리지 게이트 구성, application-test.properties 작성 | - | DevOps/QA |
| W1 | Repository @DataJpaTest 착수, 기본 CRUD 검증 | Repository 30%+ | 각 Dev |
| W2 | Service 단위 테스트 집중, MockMvc Controller 테스트, 외부 API Mock 테스트 | Service 60%+ | 각 Dev |
| W3 초 | Controller 테스트 보강, 동시성 테스트 완성, 통합 테스트 작성 | Service 80%+, 전체 70%+ | DevOps/QA + 각 Dev |
| W3 중 | E2E 통합 테스트 전체 통과, k6 부하 테스트 실행 | 핵심 시나리오 100% | DevOps/QA |
| W3 말 | 시연 시나리오 회귀 테스트, P95 측정 기록 | P95 ≤ 500ms | DevOps/QA |
| W4 | 매일 09시 회귀 테스트 실행, 결함 발생 즉시 수정 | (유지) | DevOps/QA |

---

## 10. CI 연동

### `.github/workflows/ci.yml` 핵심 설정

```yaml
- name: Run tests with coverage
  run: ./gradlew test jacocoTestReport jacocoTestCoverageVerification
  env:
    SPRING_PROFILES_ACTIVE: test

- name: Upload coverage report
  uses: actions/upload-artifact@v4
  if: always()
  with:
    name: jacoco-report
    path: build/reports/jacoco/test/html/

# PR에 커버리지 코멘트 (선택)
- name: Coverage comment
  uses: madrapps/jacoco-report@v1.6.1
  with:
    paths: ${{ github.workspace }}/build/reports/jacoco/test/jacocoTestReport.xml
    token: ${{ secrets.GITHUB_TOKEN }}
    min-coverage-overall: 70
    min-coverage-changed-files: 80
```

### PR 머지 조건

- CI green (빌드 + 테스트 + 린트 + 커버리지 게이트)
- Review 2인 approve (그 중 Lead 1명 권장)
- Service 커버리지 미달 시 보완 요청 (게이트 실패)

---

## 11. 테스트 작성 규칙

### 코드 컨벤션

```java
// 클래스명: {Subject}Test
// 메서드명: 한글 + 언더스코어 (행동_조건_기대결과)
void create_withDuplicateEmail_throwsBusinessException() { }

// 또는 영문 + DisplayName 조합
@Test
@DisplayName("중복 이메일로 생성 시 BusinessException 발생")
void create_withDuplicateEmail_throwsException() { }
```

### `application-test.properties`

```properties
# 테스트 전용 H2 인메모리 DB
spring.datasource.url=jdbc:h2:mem:testdb;MODE=MySQL
spring.datasource.driver-class-name=org.h2.Driver
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect

# Redis 비활성화 (단위 테스트)
spring.autoconfigure.exclude=\
  org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
  org.springframework.boot.autoconfigure.session.SessionAutoConfiguration

# 외부 API 키 (Mock 처리, 실제 호출하지 않음)
{{external}}.api.key=test-key-do-not-call
```

### 파일 구조

```
src/test/java/{{BASE_PACKAGE}}/
├── domain/
│   ├── auth/
│   │   ├── AuthServiceTest.java         ← 단위 테스트
│   │   └── AuthRepositoryTest.java
│   ├── {{moduleA}}/
│   │   ├── {{ModuleA}}ServiceTest.java
│   │   ├── {{ModuleA}}ControllerTest.java
│   │   └── {{ModuleA}}ConcurrencyTest.java  ← 동시성 테스트
│   └── ...
├── integration/
│   └── {{Scenario}}IntegrationTest.java  ← E2E 통합 테스트
└── performance/
    └── k6-script.js                       ← k6 부하 테스트
```

### 필수 체크리스트 (PR 전)

```
[ ] 서비스 메서드 추가 시 단위 테스트 동반
[ ] 테스트마다 실패 케이스 1개 이상 포함
[ ] 외부 API: @MockBean으로 Mock 처리 (실제 호출 없음)
[ ] 동시성 위험 있는 기능: ConcurrencyTest 작성
[ ] Given-When-Then 주석 또는 @DisplayName 명시
[ ] 테스트 데이터: 하드코딩 최소화, 팩토리 메서드 사용
```
