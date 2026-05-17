# {{PROJECT_NAME}} — CI/CD 파이프라인 가이드

> **문서 버전:** v1.0
> **작성일:** YYYY-MM-DD
> **연관 문서:** 09 Git 규칙 / 11 테스트 전략서 / 14 배포 가이드

---

## 📌 변경 이력

| 버전 | 날짜 | 작성자 | 주요 변경 |
| --- | --- | --- | --- |
| v1.0 | YYYY-MM-DD | DevOps/QA | 초기 작성 |

---

## 0. 이 문서의 목적

GitHub Actions 기반으로 **빌드 → 테스트 → 품질 게이트 → 빌드 산출물 → 배포** 자동화를 구성한다. 포트폴리오 관점에서 "테스트 + 품질 게이트가 있는 저장소"는 신뢰를 크게 높인다.

---

## 1. 파이프라인 전체 그림

```
PR 생성/업데이트
    │
    ▼
┌──────────────────────────────────────┐
│ ci.yml (필수 게이트)                 │
│  ├─ 1. 빌드 (Gradle)                  │
│  ├─ 2. 단위 테스트 + 커버리지         │
│  ├─ 3. 린트 (Checkstyle/SpotBugs)     │
│  └─ 4. 보안 스캔 (Dependency-Check)  │
└──────────────────────────────────────┘
    │ (모두 통과 시)
    ▼
2명 리뷰 승인
    │
    ▼
develop 머지
    │
    ▼
┌──────────────────────────────────────┐
│ stage-deploy.yml                     │
│  ├─ Docker 이미지 빌드                │
│  └─ 스테이징 환경 배포                │
└──────────────────────────────────────┘
    │
    ▼ (수동 승인)
main 머지 (release/* 브랜치 통해)
    │
    ▼
┌──────────────────────────────────────┐
│ prod-deploy.yml                      │
│  ├─ 태그 푸시                          │
│  └─ 운영 배포                          │
└──────────────────────────────────────┘
```

---

## 2. 필수 워크플로우 파일

### 2.1 `.github/workflows/ci.yml` — PR 게이트

```yaml
name: CI

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [develop]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build-test:
    name: Build & Test
    runs-on: ubuntu-latest
    timeout-minutes: 15

    services:
      mysql:
        image: mysql:8
        env:
          MYSQL_ROOT_PASSWORD: testpw
          MYSQL_DATABASE: {{project}}_test
        ports: ['3306:3306']
        options: >-
          --health-cmd="mysqladmin ping" --health-interval=10s
          --health-timeout=5s --health-retries=10
      redis:
        image: redis:7
        ports: ['6379:6379']
        options: >-
          --health-cmd "redis-cli ping" --health-interval 10s
          --health-timeout 5s --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
          cache: gradle

      - name: Grant execute permission for gradlew
        run: chmod +x ./gradlew

      - name: Build (without tests)
        run: ./gradlew build -x test

      - name: Run tests with coverage
        run: ./gradlew test jacocoTestReport
        env:
          SPRING_PROFILES_ACTIVE: test

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: build/reports/tests/test/

      - name: Upload coverage report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: build/reports/jacoco/test/html/

      - name: Coverage threshold check
        run: ./gradlew jacocoTestCoverageVerification

  lint:
    name: Lint & Static Analysis
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { distribution: temurin, java-version: 21, cache: gradle }
      - run: chmod +x ./gradlew
      - name: Checkstyle
        run: ./gradlew checkstyleMain checkstyleTest
      - name: SpotBugs
        run: ./gradlew spotbugsMain
      - name: Upload SpotBugs report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: spotbugs-report
          path: build/reports/spotbugs/

  security:
    name: Security Scan
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { distribution: temurin, java-version: 21, cache: gradle }
      - run: chmod +x ./gradlew
      - name: OWASP Dependency Check
        run: ./gradlew dependencyCheckAnalyze --info
        continue-on-error: true   # 초기엔 경고만, 안정화 후 fail 처리
      - name: Upload Dependency Check report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: dependency-check-report
          path: build/reports/dependency-check-report.html
```

### 2.2 `.github/workflows/stage-deploy.yml` — develop 머지 시 스테이징 배포

```yaml
name: Stage Deploy

on:
  push:
    branches: [develop]

jobs:
  deploy:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    environment: staging

    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with: { distribution: temurin, java-version: 21, cache: gradle }
      - run: chmod +x ./gradlew

      - name: Build JAR
        run: ./gradlew bootJar -x test

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:staging-${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          script: |
            cd ~/{{repo}}
            docker compose pull
            docker compose up -d --remove-orphans
            docker image prune -f
```

### 2.3 `.github/workflows/prod-deploy.yml` — 태그 푸시 시 운영 배포

```yaml
name: Production Deploy

on:
  push:
    tags: ['v*']

jobs:
  deploy:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    environment:
      name: production
      url: https://{{domain}}

    steps:
      - uses: actions/checkout@v4
      # ... (stage-deploy.yml과 유사. environment: production로 수동 승인 게이트)
```

> `environment: production`을 GitHub Settings에서 **Required reviewers** 로 묶으면 운영 배포 직전에 사람 승인 필수.

---

## 3. 자동화 도구 설정

### 3.1 Gradle 플러그인 (build.gradle.kts)

```kotlin
plugins {
    id("org.springframework.boot") version "3.x.x"
    id("io.spring.dependency-management") version "1.1.x"
    java
    jacoco
    checkstyle
    id("com.github.spotbugs") version "6.x.x"
    id("org.owasp.dependencycheck") version "10.x.x"
}

jacoco {
    toolVersion = "0.8.11"
}

tasks.test {
    useJUnitPlatform()
    finalizedBy(tasks.jacocoTestReport)
}

tasks.jacocoTestReport {
    dependsOn(tasks.test)
    reports {
        html.required = true
        xml.required = true
    }
}

tasks.jacocoTestCoverageVerification {
    violationRules {
        rule {
            element = "BUNDLE"
            limit {
                counter = "INSTRUCTION"
                minimum = "0.70".toBigDecimal()
            }
        }
        rule {
            element = "CLASS"
            includes = listOf("*.service.*")
            limit {
                counter = "INSTRUCTION"
                minimum = "0.80".toBigDecimal()
            }
        }
        rule {
            element = "CLASS"
            excludes = listOf(
                "*.config.*",
                "*.dto.*",
                "*Application*"
            )
        }
    }
}

checkstyle {
    toolVersion = "10.x"
    configFile = file("$rootDir/config/checkstyle/checkstyle.xml")
    isIgnoreFailures = false
}

spotbugs {
    toolVersion.set("4.8.x")
    excludeFilter.set(file("$rootDir/config/spotbugs/exclude.xml"))
}
```

### 3.2 Branch Protection (Settings → Branches)

`main` / `develop` 둘 다:

- [x] Require a pull request before merging
- [x] Require approvals — 2명
- [x] Require status checks to pass — `Build & Test`, `Lint & Static Analysis`
- [x] Require branches to be up to date before merging
- [x] Require conversation resolution before merging
- [x] Include administrators

### 3.3 GitHub 환경(Environments)

- **staging**: secrets — `STAGING_HOST`, `STAGING_USER`, `STAGING_SSH_KEY`
- **production**: secrets 동일 + **Required reviewers**: 책임개발자 1명

---

## 4. 코드 품질 도구

### 4.1 Checkstyle 핵심 규칙 (`config/checkstyle/checkstyle.xml`)

- 인덴트 4 spaces
- 라인 길이 120자
- 미사용 import 금지
- 매직 넘버 금지 (-1, 0, 1, 2 제외)
- public 메서드 javadoc 권장 (필수는 아님 — 너무 빡세면 위반 폭주)

### 4.2 SpotBugs 핵심 카테고리

- BUG: 명백한 버그 패턴
- SECURITY: 보안 취약 패턴
- PERFORMANCE: 성능 안티패턴
- (CORRECTNESS, BAD_PRACTICE는 점진 도입)

### 4.3 (선택) SonarCloud 연동

오픈소스 저장소는 무료. 포트폴리오 가산점.

```yaml
- name: SonarCloud Scan
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
  run: ./gradlew sonar
```

---

## 5. 자동화된 PR 라벨링 & 알림

### 5.1 `.github/labeler.yml` (자동 라벨)

```yaml
backend:
  - changed-files:
      - any-glob-to-any-file: 'src/main/java/**'
frontend:
  - changed-files:
      - any-glob-to-any-file: 'src/main/resources/templates/**'
docs:
  - changed-files:
      - any-glob-to-any-file: 'docs/**'
ci:
  - changed-files:
      - any-glob-to-any-file: '.github/workflows/**'
```

### 5.2 `.github/workflows/labeler.yml`

```yaml
name: PR Labeler
on: [pull_request_target]
jobs:
  label:
    runs-on: ubuntu-latest
    permissions: { contents: read, pull-requests: write }
    steps:
      - uses: actions/labeler@v5
```

### 5.3 Slack 알림 (선택)

`#ci-alerts` 채널에 빌드 실패만 알림.

```yaml
- name: Notify Slack on failure
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    payload: '{"text": "❌ CI failed on ${{ github.ref }}"}'
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

---

## 6. PR 자동 검증 추가 (선택)

### 6.1 PR 제목 포맷 강제 (Conventional Commits)

```yaml
name: PR Title Lint
on: [pull_request]
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: amannn/action-semantic-pull-request@v5
        env: { GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} }
```

### 6.2 PR 사이즈 경고

```yaml
- name: PR size labeler
  uses: codelytv/pr-size-labeler@v1
  with:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    s_label: 'size/S'   # ~100 LOC
    m_label: 'size/M'   # ~400 LOC
    l_label: 'size/L'   # ~1000 LOC
    xl_label: 'size/XL' # 1000 LOC+
    fail_if_xl: false
```

---

## 7. 캐싱 전략

CI 시간 단축을 위해 반드시:

- **Gradle 캐시:** `actions/setup-java` 의 `cache: gradle`
- **Docker 빌드 캐시:** `cache-from: type=gha, cache-to: type=gha,mode=max`
- **Service container** 헬스체크 사용 — 무한 대기 방지

---

## 8. 비용 & 사용량 관리

GitHub Actions는 public 저장소면 무료. **단, 사용량 폭주 방지:**

- `concurrency` 그룹으로 동일 PR의 이전 실행 자동 취소
- `timeout-minutes` 모든 job에 명시
- 무거운 job(보안 스캔)은 `develop` 머지 시점에만 (`schedule:` cron 야간 실행도 OK)
- Secret 노출 방지 — 절대 `echo $SECRET` 금지

---

## 9. 실패 시 트러블슈팅 가이드

| 증상 | 원인 후보 | 1차 조치 |
| --- | --- | --- |
| Gradle build OOM | 힙 부족 | `org.gradle.jvmargs=-Xmx2g` |
| Test에서 DB 연결 실패 | services 헬스체크 부족 | `--health-retries` 늘리기 |
| Coverage 미달 | 테스트 부족 | 미달 클래스 PR에 명시 |
| Checkstyle 갑자기 폭주 | 룰 추가됨 | `// CHECKSTYLE:OFF` 임시, 후속 이슈 |
| 도커 이미지 캐시 미스 | 의존성 변경 | gradle dependencies 변경분만 우선 빌드 |
| GHCR push 권한 오류 | PAT 만료 | `permissions: packages: write` 확인 |

---

## 10. 도입 단계 — 3주 압축 일정에 맞는 점진 도입

압축 일정에서 한 번에 다 켜면 일정 망가짐. 단계별로:

| 단계 | 시점 | 내용 |
| --- | --- | --- |
| 1단계 | W1 월~화 | `ci.yml` (build-test만) — 가장 먼저 안전망 확보 |
| 2단계 | W1 수~목 | Branch Protection, PR template |
| 3단계 | W1 금 | Lint job 추가 (Checkstyle 룰은 최소부터) |
| 4단계 | W2 월 | Coverage 검증 추가 (목표: Service 60%부터 시작, W3에 80%) |
| 5단계 | W2 중반 | stage-deploy.yml |
| 6단계 | W3 후반 | 보안 스캔, prod-deploy.yml |
| 7단계 | W4 | Slack 알림, 라벨러 등 부가 기능 |

> **원칙:** "안 켜진 게이트 < 망가진 게이트". 작동하지 않는 CI를 끌고 가지 말고 단계적으로 올린다.

---

## 11. 운영 후 점검

- [ ] 평균 CI 시간 5분 이내?
- [ ] 빌드 실패율 10% 이하?
- [ ] False positive (오탐) 비율?
- [ ] 가장 자주 실패하는 step? → 보강
- [ ] 캐시 hit ratio?

매주 금 통합 미팅에서 1분으로 공유.

---

## 📋 CI/CD 도입 체크리스트

```
[ ] ci.yml 통과
[ ] PR template / Issue template
[ ] Branch protection (main / develop)
[ ] Coverage threshold (Service 80%+)
[ ] Checkstyle / SpotBugs
[ ] OWASP Dependency Check (월 1회 이상 결과 확인)
[ ] stage-deploy 자동
[ ] prod-deploy는 수동 승인
[ ] Secrets 분리 (staging / production)
[ ] CI 평균 시간 5분 이하
[ ] 빌드 배지(badge)를 README에 노출
```
