# 📋 {{PROJECT_NAME}} — DevOps/QA 상세 일정표

> **문서 버전:** v1.0
> **작성일:** YYYY-MM-DD
> **담당자:** {{DEVOPS_NAME}}
> **역할:** DevOps / QA 엔지니어
> **담당 영역:** CI/CD · 인프라 · 테스트 자동화 · 모니터링 · 문서 정합성
> **개발 기간:** W0 ~ W4 (전 기간 집중)
> **연관 문서:** 00_개발_일정_총괄표 / 13_테스트_보고서 / 16_운영_메뉴얼 / 20_CICD_파이프라인 / AGENTS.md

---

## 📌 변경 이력

| 버전 | 날짜 | 작성자 | 주요 변경 |
| --- | --- | --- |
| v1.0 | YYYY-MM-DD | DevOps/QA | 초기 작성 |

---

## 0. DevOps/QA 역할 핵심 선언

> **"팀이 자신 있게 코드를 올릴 수 있는 환경을 만든다."**
> DevOps/QA는 기능 개발에서 빠지는 대신, **팀 전체의 속도·품질·배포 안정성**의 단독 책임자가 된다.
> CI/CD가 W1에 완성되지 않으면 팀 전체의 PR 게이트가 없어진다. **W1은 DevOps의 최고 우선순위.**

### DevOps/QA의 3대 책임

| 책임 | 내용 | 실패 시 영향 |
| --- | --- | --- |
| **CI 게이트** | 모든 PR에 빌드·테스트·린트 자동 실행 | 코드 품질 무법지대 |
| **배포 안정성** | 스테이징·운영 배포 자동화 + 롤백 전략 | 시연 당일 장애 |
| **테스트 자동화** | E2E·통합·부하 테스트 파이프라인 | 품질 보증 불가 |

---

## 1. 소유권 선언 (Code Ownership)

```
.github/
└── workflows/                      ← DevOps/QA 전담 소유 (Lead 공동)
    ├── ci.yml                      (PR 빌드·테스트·린트)
    ├── cd-staging.yml              (develop → 스테이징 자동 배포)
    ├── cd-prod.yml                 (main → 운영 수동 트리거)
    └── scheduled-test.yml         (야간 회귀 테스트, 선택)

docker/                             ← DevOps/QA 전담
├── Dockerfile
├── docker-compose.yml              (로컬 개발)
├── docker-compose.staging.yml
└── docker-compose.prod.yml

scripts/                            ← DevOps/QA 전담
├── deploy.sh
├── rollback.sh
├── health-check.sh
└── seed-data.sh

src/test/java/{{BASE_PACKAGE}}/
└── integration/                    ← DevOps/QA 주도 (각 Dev 협업)
    ├── {{ModuleA}}IntegrationTest.java
    ├── {{ModuleB}}IntegrationTest.java
    └── E2EScenarioTest.java

docs/
├── 13_테스트_보고서.md             ← DevOps/QA 작성
├── 16_운영_매뉴얼.md              ← DevOps/QA 작성
└── 20_CICD_파이프라인.md          ← DevOps/QA 작성
```

> **단독 결정권:** CI/CD 파이프라인 구조, 배포 전략, 테스트 환경 구성.
> Lead와 협의 후 결정하되, 일상적인 파이프라인 변경은 DevOps/QA 단독.

---

## 2. 일별 상세 일정

### 🟦 W0 — 킥오프 + 인프라 선행 준비

| 일자 | 활동 | 산출물 |
| --- | --- | --- |
| D-4 (수) | 킥오프 회의 참석. DevOps/QA 역할·도구 확정. 서버 환경 확인 (운영 서버 스펙·접근 권한). | 역할 확정 |
| D-3 (목) | 03_아키텍처 v1 작성 Lead와 페어. 인프라 구조 확정 (Nginx + Spring Boot + MySQL + Redis + Docker). | 인프라 아키텍처 초안 |
| D-2 (금) | **GitHub Actions CI 1단계 가동** (PR 빌드+테스트). 09_Git 규칙 + 브랜치 보호 설정 Lead와 함께. | ci.yml commit + CI 동작 확인 |
| D-1 (토) | Docker 기본 설정 (docker-compose.yml, Dockerfile 초안). | Dockerfile + compose 초안 |

---

### 🟩 W1 — CI/CD 완전 구축 (DevOps 최우선 주간)

**DevOps/QA W1 집중 목표:** "W1 금요일까지 CI/CD 전체 파이프라인 완성. 팀이 PR 올리면 자동으로 게이트 통과"

| 일자 | 오전 | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W1 월** | Stand-up. develop pull. | **CI 파이프라인 완성 (ci.yml)**. 빌드·테스트·린트·커버리지 리포트 자동화. | CI 동작 전체 팀 Slack 공지. |
| **W1 화** | Stand-up. | **Docker 완성** — Dockerfile 멀티스테이지, docker-compose 3종 (dev/staging/prod). | 로컬 docker-compose 팀 테스트. |
| **W1 수** | Stand-up. | **스테이징 배포 파이프라인 (cd-staging.yml)**. develop push → 스테이징 자동 배포. | 스테이징 URL 팀 공지. |
| **W1 목** | Stand-up. | **MySQL + Redis 운영 환경 구성**. 헬스체크 엔드포인트 확인 (`/actuator/health`). | 환경변수 관리 방식 문서화. |
| **W1 금** | Stand-up. | **테스트 자동화 기반 설정** — JaCoCo 커버리지 리포트, 임계값 설정. **W1 게이트 회의 (16:00)**. | W1 회고. 20_CICD 문서 초안. |

**W1 DevOps/QA 산출물**

| 산출물 | 파일 | 완료 기준 |
| --- | --- | --- |
| CI 파이프라인 | `.github/workflows/ci.yml` | PR마다 빌드·테스트·린트 자동 실행 green |
| CD 스테이징 | `.github/workflows/cd-staging.yml` | develop push → 스테이징 자동 배포 |
| Docker 3종 | `docker/docker-compose*.yml` | 전 팀원 로컬 `docker-compose up` 성공 |
| 커버리지 게이트 | JaCoCo 설정 | Service 60% 미달 시 CI 실패 |
| 스테이징 URL | 공지 | 전 팀원 접속 가능 |

**W1 금 게이트 체크리스트 (DevOps/QA)**

```
[ ] CI: 임의 PR에서 빌드·테스트·린트 자동 동작 확인
[ ] CD: develop push 후 스테이징 배포 자동 확인
[ ] Docker: docker-compose up으로 전체 서비스 로컬 기동 확인
[ ] JaCoCo: 커버리지 리포트 CI에서 생성 확인
[ ] GitHub: main/develop 브랜치 보호 + PR 1명 리뷰 + CI green 필수 규칙 동작
[ ] 전 팀원: 스테이징 URL 접속 성공 확인
```

---

### 🟨 W2 — 테스트 자동화 + 모니터링 구축

**DevOps/QA W2 집중 목표:** "단위 테스트 리뷰 지원 + 통합 테스트 기반 완성 + 모니터링 구축"

| 일자 | 오전 | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W2 월** | Stand-up. | **모니터링 구축** — Spring Actuator 엔드포인트 활성화, Logback 로그 설정, 서버 헬스체크 스크립트. | 모니터링 URL 팀 공지. |
| **W2 화** | Stand-up. | **통합 테스트 기반 구조 작성** — `@SpringBootTest`, TestContainers (선택) 또는 H2 테스트 DB 설정. | 통합 테스트 1개 이상 동작. |
| **W2 수** | Stand-up. | **각 모듈 통합 테스트 작성 지원** (Dev A/B/C/D와 협업). 테스트 데이터 픽스처 설계. | 모듈별 통합 테스트 초안 완성. |
| **W2 목** | Stand-up. | **E2E 테스트 시나리오 설계** (주요 사용자 흐름 4가지). Selenium 또는 REST Assured 기반. | E2E 시나리오 문서화. |
| **W2 금** | Stand-up. | 중간 데모 인프라 지원. **W2 게이트 회의 (16:00)**. | W2 회고. 테스트 전략 문서 갱신. |

**W2 DevOps/QA 산출물**

| 산출물 | 완료 기준 |
| --- | --- |
| 모니터링 | Actuator health/info 엔드포인트 동작 + 서버 로그 수집 |
| 통합 테스트 | `@SpringBootTest` 기반 각 모듈 1건 이상 통과 |
| E2E 시나리오 | 4가지 핵심 흐름 문서화 + 1가지 자동화 |
| 11_테스트_전략 | W2 현황 반영 버전 갱신 |

---

### 🟥 W3 — 통합 테스트 + 부하 테스트 + 운영 배포

**DevOps/QA W3 집중 목표:** "운영 배포 완성 + 부하 테스트 수행 + 시연 환경 안전 보장"

| 일자 | 오전 | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W3 월** | Stand-up. | **통합 E2E 테스트 전체 완성**. 핵심 시나리오 4가지 자동화 완성. | CI에 통합 테스트 파이프라인 추가. |
| **W3 화** | Stand-up. Feature Freeze 확인. | **부하 테스트 수행** (JMeter 또는 k6). TPS, 응답시간(P95), 오류율 측정. | 부하 테스트 결과 #dev 공유. |
| **W3 수** | Stand-up. | **운영 배포 준비** — cd-prod.yml 완성, 운영 환경변수 설정, DB 마이그레이션 확인. | Lead와 운영 배포 리허설. |
| **W3 목** | Stand-up. | **운영 배포 실행** (Lead와 합동). 시드 데이터 삽입 실행. 헬스체크 + 시연 시나리오 운영에서 확인. | 운영 URL 전 팀원 접속 확인. |
| **W3 금** | Stand-up. | **W3 게이트 회의 + 시연 1차 리허설**. 16_운영_매뉴얼 초안. | W3 회고. |

**W3 DevOps/QA 산출물**

| 산출물 | 완료 기준 |
| --- | --- |
| E2E 테스트 | 4가지 시나리오 CI에서 자동 실행 + 통과 |
| 부하 테스트 결과 | TPS/P95/오류율 측정값 기록 (목표: P95 500ms 이하) |
| 운영 배포 | 외부 URL 접속 + 핵심 기능 동작 확인 |
| 롤백 전략 | rollback.sh 동작 확인 (이전 버전 복구 가능) |
| 16_운영_매뉴얼 초안 | 서버 접근·배포·롤백·헬스체크 방법 문서화 |

---

### ⬜ W4 — 시연 환경 안전 보장 + 문서 완성

| 일자 | 활동 | 산출물 |
| --- | --- | --- |
| W4 월 | **13_테스트_보고서 작성** (커버리지, E2E 결과, 부하 테스트 결과). | 테스트 보고서 완성 |
| W4 화 | 시연 리허설 1회차 — 인프라 담당. 시연 중 장애 대응 매뉴얼 완성. | 장애 대응 매뉴얼 |
| W4 수 | **백업 영상 녹화** (시연 장애 대비 필수). 운영 서버 최종 헬스체크. **기술 블로그 1편 완성**. | 백업 영상 + 블로그 1편 |
| W4 목 | 시연 리허설 2회차 — 시연 전 30분 헬스체크 시나리오 리허설. | 헬스체크 루틴 확정 |
| W4 금(시연일) | **D-30분 헬스체크 수행**. 시연 중 인프라 모니터링 담당. 장애 발생 시 즉각 대응. | 시연 성공 |

**W4 DevOps/QA 특별 임무: 시연 당일 헬스체크 루틴**

```bash
# 시연 D-30분 헬스체크 스크립트 (순서대로 실행)
#!/bin/bash

echo "=== 시연 D-30분 헬스체크 시작 ==="

# 1. 운영 서버 기동 확인
curl -f https://{{PROD_URL}}/actuator/health || { echo "❌ 서버 다운!"; exit 1; }
echo "✅ 서버 기동 확인"

# 2. DB 연결 확인
curl -f https://{{PROD_URL}}/actuator/health/db || { echo "❌ DB 연결 실패!"; exit 1; }
echo "✅ DB 연결 확인"

# 3. 핵심 API 응답 확인
curl -f https://{{PROD_URL}}/api/health-check || { echo "❌ API 응답 실패!"; exit 1; }
echo "✅ API 응답 확인"

# 4. 시연 시나리오 계정 로그인 확인
# (직접 브라우저로 확인)

# 5. 시드 데이터 존재 확인
# (시연 시나리오 첫 화면 확인)

echo "=== ✅ 헬스체크 완료. 시연 준비 OK ==="
```

---

## 3. CI/CD 파이프라인 상세 설계

### 3.1 ci.yml (PR 게이트)

```yaml
name: CI

on:
  pull_request:
    branches: [ main, develop ]

jobs:
  build-test:
    runs-on: ubuntu-latest
    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_DATABASE: {{DB_NAME}}_test
          MYSQL_ROOT_PASSWORD: test
        options: --health-cmd="mysqladmin ping" --health-interval=10s
      redis:
        image: redis:7-alpine
        options: --health-cmd="redis-cli ping"

    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'

      - name: Cache Gradle
        uses: actions/cache@v4
        with:
          path: ~/.gradle/caches
          key: gradle-${{ hashFiles('**/*.gradle*') }}

      - name: Build & Test
        run: ./gradlew build test jacocoTestReport
        env:
          SPRING_PROFILES_ACTIVE: test

      - name: Check Coverage
        run: ./gradlew jacocoTestCoverageVerification  # 임계값 미달 시 실패

      - name: Upload Coverage Report
        uses: actions/upload-artifact@v4
        with:
          name: jacoco-report
          path: build/reports/jacoco/

      - name: Lint (Checkstyle)
        run: ./gradlew checkstyleMain checkstyleTest
```

### 3.2 cd-staging.yml (스테이징 자동 배포)

```yaml
name: CD-Staging

on:
  push:
    branches: [ develop ]

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build Docker Image
        run: docker build -t {{IMAGE_NAME}}:staging .

      - name: Deploy to Staging
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          script: |
            cd /app/staging
            docker-compose -f docker-compose.staging.yml pull
            docker-compose -f docker-compose.staging.yml up -d --remove-orphans
            sleep 10
            curl -f http://localhost:8080/actuator/health || exit 1
            echo "✅ 스테이징 배포 성공"
```

### 3.3 JaCoCo 커버리지 게이트 설정

```kotlin
// build.gradle.kts
jacoco {
    toolVersion = "0.8.11"
}

tasks.jacocoTestCoverageVerification {
    violationRules {
        rule {
            limit {
                counter = "INSTRUCTION"
                value = "COVEREDRATIO"
                minimum = "0.60".toBigDecimal()  // W2: 60%, W3: 70%+로 단계적 상향
            }
            // Service 레이어만 적용
            includes = listOf("*.service.*")
        }
    }
}

tasks.jacocoTestReport {
    reports {
        xml.required = true
        html.required = true
    }
}
```

---

## 4. 테스트 전략 상세

### 4.1 테스트 계층별 전략

| 테스트 종류 | 작성 주체 | 도구 | 완료 목표 | 커버리지 |
| --- | --- | --- | --- | --- |
| 단위 테스트 (Service) | 각 Dev | JUnit5 + Mockito | W2 목 | 80%+ |
| 단위 테스트 (Repository) | 각 Dev | @DataJpaTest | W1 | 기본 CRUD |
| Controller 테스트 | 각 Dev | MockMvc | W2 | 주요 엔드포인트 |
| 통합 테스트 | DevOps/QA + 각 Dev | @SpringBootTest | W3 초 | 핵심 시나리오 100% |
| E2E 테스트 | DevOps/QA | REST Assured | W3 중 | 4가지 시나리오 |
| 부하 테스트 | DevOps/QA | JMeter / k6 | W3 화 | TPS / P95 측정 |
| 회귀 테스트 | DevOps/QA | 전체 자동화 | W4 | 유지 |

### 4.2 통합 테스트 기본 구조

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Transactional
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class {{Scenario}}IntegrationTest {

    @Autowired private TestRestTemplate restTemplate;
    @Autowired private MemberRepository memberRepository;

    @BeforeEach
    void setUp() {
        // 공통 테스트 데이터 설정
        // 각 테스트 독립성 보장
    }

    @Test
    @Order(1)
    @DisplayName("회원가입 → 로그인 → 핵심 기능 사용 전체 흐름")
    void 전체_사용자_여정() {
        // Given: 사용자 정보
        // When: 회원가입 → 로그인 → API 호출
        // Then: 기대 결과 검증
    }
}
```

### 4.3 부하 테스트 시나리오 (JMeter)

```
목표 지표:
- 동시 사용자: {{N}}명 (예: 50명)
- 목표 TPS: {{T}} (예: 30 TPS)
- P95 응답시간: 500ms 이하
- 오류율: 1% 이하
- 지속 시간: 5분

테스트 시나리오:
1. 로그인 → {{핵심 기능}} 조회 (읽기 위주, 높은 TPS)
2. 로그인 → {{핵심 기능}} 생성 (쓰기 위주, 동시성 확인)

결과 기록 (13_테스트_보고서에 포함):
- Thread Count / Ramp-up / Duration
- Average / P95 / P99 / Max 응답시간
- Error Rate
- Throughput (TPS)
```

---

## 5. 환경 관리 기준

### 5.1 환경변수 관리 원칙

```
# ✅ 모든 민감 정보: GitHub Secrets 또는 서버 환경변수
DB_PASSWORD=xxx          → GitHub Secrets: PROD_DB_PASSWORD
JWT_SECRET=xxx           → GitHub Secrets: JWT_SECRET
EXTERNAL_API_KEY=xxx     → GitHub Secrets: EXTERNAL_API_KEY

# ✅ application.properties에는 환경변수 참조만
spring.datasource.password=${DB_PASSWORD}
app.jwt.secret=${JWT_SECRET}

# ✅ .gitignore에 반드시 추가
.env
application-prod.properties  (또는 Lead 관리)
```

### 5.2 Docker 멀티스테이지 빌드 기준

```dockerfile
# ✅ 멀티스테이지: 최종 이미지에 소스코드 포함 금지
# Build Stage
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY gradlew .
COPY gradle gradle
COPY build.gradle.kts .
COPY settings.gradle.kts .
COPY src src
RUN ./gradlew bootJar --no-daemon

# Run Stage
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring
COPY --from=builder /app/build/libs/*.jar app.jar

HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -qO- http://localhost:8080/actuator/health || exit 1

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## 6. AI 에이전트 활용 가이드 (DevOps/QA 특화)

| 작업 | 도구 | 프롬프트 예시 |
| --- | --- | --- |
| GitHub Actions 작성 | Claude Code | "Spring Boot 21 + Gradle + MySQL 8 + Redis 7 환경의 GitHub Actions CI 파이프라인을 작성해줘. JaCoCo 커버리지 60% 미달 시 실패, Checkstyle 포함." |
| Dockerfile | Claude Code | "Spring Boot JAR 멀티스테이지 Dockerfile. JRE 21 알파인 기반. 비루트 사용자 실행. HEALTHCHECK 포함." |
| 통합 테스트 | Claude Code | "@{{Scenario}}.java 비즈니스 시나리오를 보고, @SpringBootTest 통합 테스트를 작성해줘. TestRestTemplate, 트랜잭션 롤백 기반." |
| 부하 테스트 스크립트 | Claude 웹 | "k6 스크립트로 {{핵심 API}}에 50 VU, 5분간 부하 테스트. P95 500ms, 오류율 1% 이하를 threshold로 설정." |
| 운영 매뉴얼 | Claude 웹 | "다음 배포 구조를 보고 16_운영_매뉴얼을 작성해줘: {{인프라 구조}}. 장애 대응, 롤백, 로그 확인 방법 포함." |

---

## 7. 협업 포인트

| 협업 대상 | 시점 | 내용 |
| --- | --- | --- |
| **Lead** | W0 금 | CI 첫 가동 함께 확인 + main/develop 보호 규칙 |
| **Lead** | W3 목 | 운영 배포 합동 실행 |
| **Dev A** | W2 수 | 인증 통합 테스트 시나리오 협의 |
| **Dev B** | W2 수 | 동시성 통합 테스트 시나리오 협의 |
| **Dev C** | W2 수 | 외부 연동 E2E 테스트 협의 (Mock 또는 실제) |
| **Dev D** | W3 수 | 시연 시나리오 전체 E2E 테스트 실행 |
| **전원** | W4 화 | 시연 리허설 — 인프라 담당 지원 |

---

## 8. 블로커 에스컬레이션

```
막혔을 때 행동 순서:
1. 스스로 30분 이내 해결 시도 (GitHub Actions 공식 문서, Docker Hub)
2. 30분 이상 막히면 → #dev 채널에 "🔴 인프라 블로커: [내용] — [팀 영향]" 공지 + Lead DM
3. 블로커 내용에 따른 임시 조치:
   - CI 실패 → 임시로 해당 step skip 처리 후 이슈 등록
   - 배포 실패 → 이전 이미지로 롤백, Lead에게 즉시 보고
   - DB/Redis 연결 실패 → docker-compose restart + 팀원 작업 중단 공지
4. 1시간 내 해결 불가 → Lead와 함께 영향 범위 산정 + 대안 결정

타 팀원 코드 수정 필요 시:
→ 직접 수정 금지. 해당 모듈 소유자에게 Slack DM 요청
→ CI/CD 파이프라인 변경은 Lead 최종 확인 후 머지

DevOps/QA 특수 상황:
→ 운영 서버 완전 다운: Lead 즉시 연락 + rollback.sh 실행 준비
→ GitHub Secrets 만료: 즉시 교체 후 전 팀원 공지 (API 키 노출 의심 시 기존 키 즉시 폐기)
```


## 9. 기술 블로그 주제 가이드

| 주제 | 내용 | 어필 포인트 |
| --- | --- | --- |
| "GitHub Actions로 CI/CD 파이프라인 구축" | ci.yml + cd-staging.yml 구조 설명 | DevOps 역량 |
| "JaCoCo 커버리지 게이트로 품질 강제하기" | 임계값 설정 + 팀 운영 경험 | 품질 관리 |
| "6명 팀 부하 테스트 — k6/JMeter 실전" | TPS·P95 측정 결과 + 병목 분석 | 성능 엔지니어링 |
| "Spring Boot 운영 배포 — Docker + GitHub Actions" | 멀티스테이지 Dockerfile + 롤백 전략 | 인프라 실전 |

---

## 10. 일일 체크리스트 (DevOps/QA용)

### 매일 Stand-up 전 (09:00)

```
[ ] CI 현황 확인 — 어젯밤 failing PR 없는가
[ ] 스테이징 서버 헬스체크 (curl https://staging-url/actuator/health)
[ ] GitHub Actions 실패 Alert 있으면 즉시 확인
[ ] 오늘 리뷰 지원할 팀원 PR 확인
```

### 매일 저녁 회고 전 (18:00)

```
[ ] CI 빌드 실패 누적 없는가 (develop 브랜치)
[ ] 통합 테스트 추가 필요 시나리오 있는가
[ ] 커버리지 목표 달성 중인가 (JaCoCo 리포트 확인)
[ ] 내일 배포 필요 사항 있는가
```

### 주간 통합 미팅 준비 (매주 금)

```
[ ] 이번 주 CI 실패 건수·원인 요약
[ ] 현재 커버리지 수치 (Service/전체)
[ ] 스테이징 배포 성공률
[ ] 다음 주 테스트 계획 (통합/E2E/부하)
```

---

## 11. 장애 대응 매뉴얼 (시연 당일)

```
장애 유형별 대응:

1. 서버 다운
   → docker-compose restart app
   → 1분 내 미복구: rollback.sh 실행 (이전 버전 이미지)
   → 백업 영상으로 전환 신호 Lead에게 전달

2. DB 연결 실패
   → docker-compose restart db
   → 연결 확인: curl /actuator/health/db
   → 복구 불가: 로컬 H2 모드 전환 (사전 준비 필요)

3. 외부 API 다운
   → 폴백 응답 동작 여부 확인 (Dev C/Lead)
   → Mock 데이터 모드 시연으로 전환

4. 스크린 공유 안 됨
   → 시연 PC 전환 → 백업 영상 재생

5. 데이터 오염
   → seed-data.sh 재실행
   → 소요 시간: 약 30초

긴급 연락:
- Lead: {{Lead 연락처}}
- 운영 서버 접근: {{접근 방법}}
- 백업 영상 위치: {{Google Drive / YouTube 비공개 링크}}
```

---

## 12. 성공 기준 (DevOps/QA 개인)

| 기준 | 목표 | 측정 방법 |
| --- | --- | --- |
| CI 가동 시점 | W1 금요일까지 | GitHub Actions 첫 green |
| 팀 CI 성공률 | develop 브랜치 90%+ green 유지 | Actions 대시보드 |
| 스테이징 배포 | W1 금요일까지 자동화 | 자동 배포 로그 |
| 운영 배포 | W3 목까지 완료 | 외부 URL 접속 확인 |
| 부하 테스트 | P95 500ms 이하 달성 | JMeter 리포트 |
| E2E 테스트 | 핵심 시나리오 4가지 CI 통과 | Actions 로그 |
| 기술 블로그 | 1편 이상 | 노션 링크 |

---

## 📋 DevOps/QA 체크리스트 총괄

```
W0:
[ ] CI 1단계 가동 (빌드+테스트 자동화)
[ ] 브랜치 보호 규칙 설정

W1:
[ ] CI 완전 가동 (빌드+테스트+린트+커버리지)
[ ] CD 스테이징 자동 배포 완성
[ ] Docker 3종 완성 + 팀 전원 로컬 테스트
[ ] JaCoCo 60% 게이트 설정

W2:
[ ] 모니터링 구축 (Actuator + 로그)
[ ] 통합 테스트 기반 완성
[ ] E2E 시나리오 설계 완료

W3:
[ ] E2E 테스트 4가지 CI 자동화
[ ] 부하 테스트 수행 + 결과 기록
[ ] 운영 배포 완료
[ ] 16_운영_매뉴얼 초안 완성

W4:
[ ] 13_테스트_보고서 완성
[ ] 백업 영상 녹화
[ ] 시연 D-30분 헬스체크 수행
[ ] 기술 블로그 1편 발행
```
