# 📋 {{PROJECT_NAME}} — Dev D 상세 일정표

> **문서 버전:** v1.0
> **작성일:** YYYY-MM-DD
> **담당자:** {{DEVD_NAME}}
> **역할:** 개발자 D
> **담당 모듈:** 관리자(Admin) + 통계(Stats) + 프론트 통합
> **개발 기간:** W1 ~ W4
> **연관 문서:** 00_개발_일정_총괄표 / 02_ERD / 04_API_명세서 / 08_스토리보드 / AGENTS.md

---

## 📌 변경 이력

| 버전 | 날짜 | 작성자 | 주요 변경 |
| --- | --- | --- |
| v1.0 | YYYY-MM-DD | Dev D | 초기 작성 |

---

## 0. Dev D 역할 핵심 선언

> **"시스템을 내려다보는 눈이 있어야 운영이 된다."**
> Dev D는 **모든 모듈의 데이터를 종합해서 관리자와 통계 기능으로 보여주는** 역할이다.
> 다른 팀원들의 모듈이 완성되어야 통합할 수 있으므로, **W1~W2에 자체 골격을 빠르게 완성**하고,
> **W2 후반~W3에 통합·UI 마감**에 집중한다. 또한 시연 시나리오의 UI/UX를 총괄한다.

---

## 1. 소유권 선언 (Code Ownership)

```
src/main/java/{{BASE_PACKAGE}}/
└── domain/
    ├── admin/                      ← Dev D 전담 소유
    │   ├── controller/
    │   │   └── AdminController.java
    │   ├── service/
    │   │   └── AdminService.java
    │   ├── dto/
    │   │   ├── AdminDashboardResponse.java
    │   │   └── AdminSearchCondition.java
    │   └── (Entity: 다른 모듈 Entity 조합 조회)
    │
    └── stats/                      ← Dev D 전담 소유
        ├── controller/
        │   └── StatsController.java
        ├── service/
        │   └── StatsService.java
        └── dto/
            ├── StatsResponse.java
            └── ChartDataResponse.java

src/main/resources/templates/
├── admin/                          ← Dev D 전담 (Mustache)
│   ├── admin-dashboard.mustache
│   ├── admin-{{moduleA}}-list.mustache
│   ├── admin-{{moduleB}}-list.mustache
│   ├── admin-{{moduleC}}-list.mustache
│   └── admin-member-list.mustache
├── stats/                          ← Dev D 전담
│   └── stats-dashboard.mustache
└── layout/                         ← Dev D 총괄 (공통 레이아웃, Lead 협의)
    ├── header.mustache
    ├── footer.mustache
    └── sidebar.mustache

src/test/java/{{BASE_PACKAGE}}/
└── domain/
    ├── admin/                      ← Dev D 작성
    └── stats/                      ← Dev D 작성
```

> **Dev D의 특수한 의존성:**
> Admin/Stats 서비스는 **다른 모든 모듈의 Repository/Service를 조회**한다.
> → 모든 팀원의 public 메서드를 사용하되, **다른 모듈의 코드를 직접 수정하지 않는다.**
> → 필요한 조회 메서드가 없으면 해당 모듈 담당자에게 요청한다.

---

## 2. Dev D의 특수 전략 (병렬 진행)

```
W1 — 자체 관리자 골격 빠르게 완성
     └─> 다른 모듈 완성을 기다리지 않고 Mock 데이터로 화면 완성

W2 전반 — 다른 모듈과 실제 연동 시작
     └─> Dev A/B/C의 Service를 import하여 실제 데이터로 교체

W2 후반~W3 — 전체 UI 통합
     └─> 공통 레이아웃, 내비게이션, 시연 시나리오 흐름 완성

W4 — 시연 UI/UX 총괄
     └─> 모든 팀원 화면이 일관되게 보이도록 조율
```

---

## 3. 일별 상세 일정

### 🟦 W0 — 킥오프 참여

| 일자 | 활동 | 산출물 |
| --- | --- | --- |
| D-4 (수) | 킥오프 회의 참석. 관리자·통계 모듈 담당 확정. | 역할 확정 |
| D-3 (목) | ERD — 통계용 집계 쿼리 설계 (Join 범위 논의). | ERD 통계 파트 |
| D-2 (금) | 08_스토리보드 시안 + 화면 정의서 관리자 파트 초안 작성. | 스토리보드 초안 |
| D-1 (토) | 개발 환경 세팅. 공통 레이아웃(header/footer) 구조 설계. | 레이아웃 초안 |

---

### 🟩 W1 — 관리자·통계 골격 + 공통 레이아웃

**Dev D W1 집중 목표:** "공통 레이아웃 완성 + 관리자/통계 골격 Mock 데이터로 화면까지 완성"

| 일자 | 오전 | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W1 월** | Stand-up. develop pull + 공통 레이어 확인. | **공통 레이아웃 작성** (header.mustache, footer.mustache, sidebar.mustache). Lead와 레이아웃 구조 협의. | 레이아웃 로컬 렌더링 확인. |
| **W1 화** | Stand-up. | **AdminController + AdminService 골격** 작성. Mock 데이터로 대시보드 화면 작성 시작. | admin-dashboard.mustache 기본 완성. |
| **W1 수** | Stand-up. | **StatsController + StatsService 골격**. Mock 데이터로 통계 화면 기본 완성. | 차트 라이브러리 결정 (Chart.js 권장). |
| **W1 목** | Stand-up. | **관리자 목록 화면** (member-list, 각 모듈 list) — Mock 데이터로 페이징 UI까지 완성. | 08_스토리보드 화면 정의서 업데이트. |
| **W1 금** | Stand-up. | **W1 게이트 회의 (16:00)**. 레이아웃 PR 제출. | W1 회고. |

**W1 Dev D 산출물 (Mock 데이터 기반이어도 OK)**

| 산출물 | 파일 | 완료 기준 |
| --- | --- | --- |
| 공통 레이아웃 | `layout/*.mustache` | 전체 페이지에서 공통 헤더/푸터 렌더링 |
| 관리자 대시보드 | `admin-dashboard.mustache` | Mock 데이터로 숫자 카드 표시 |
| 관리자 목록 화면 | `admin-*-list.mustache` | Mock 페이징 UI 동작 |
| 통계 화면 기본 | `stats-dashboard.mustache` | Chart.js 기본 차트 렌더링 |
| 화면 정의서 | 08_스토리보드 관리자 파트 | 전체 화면 목록·흐름 명시 |

---

### 🟨 W2 — 실제 데이터 연동 + 통계 완성

**Dev D W2 집중 목표:** "Mock → 실제 DB 데이터로 교체 + 통계 집계 완성"

| 일자 | 오전 | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W2 월** | Stand-up. Dev A/B의 Service 공개 API 확인. | **AdminService에 실제 데이터 연동 시작** — `MemberService`, `{{ModuleA}}Service`, `{{ModuleB}}Service` import. | 회원 관리 실데이터 확인. |
| **W2 화** | Stand-up. API 정합 미팅 참석. | **{{ModuleC}}Service** 연동. 관리자 CRUD (상태 변경, 강제 취소 등). | 관리자 권한 확인 (@PreAuthorize 또는 SecurityConfig). |
| **W2 수** | Stand-up. | **StatsService 집계 쿼리** 작성 — 날짜별·상태별·{{기준}}별 집계. | 통계 데이터 검증. |
| **W2 목** | Stand-up. | **차트 데이터 API** 완성 (JSON 응답 → Chart.js 연동). 대시보드 숫자 실데이터 교체. | 통합 테스트 1개. |
| **W2 금** | Stand-up. | 중간 데모. **W2 게이트 회의 (16:00)**. | W2 회고. |

**W2 Dev D 산출물**

| 산출물 | 완료 기준 |
| --- | --- |
| 관리자 대시보드 | 실제 DB 기반 숫자 카드 (회원 수, {{모듈}}별 집계) |
| 관리자 목록·상세 | 전체 모듈 데이터 조회 + 상태 변경 가능 |
| 통계 화면 | 날짜별·상태별 집계 차트 (Chart.js) |
| Service 커버리지 | 60%+ |

**W2 특별 규칙: 타 모듈 코드 직접 수정 금지**

```
AdminService가 다른 모듈 데이터 필요 시:
1. 해당 모듈 Service의 공개 메서드 확인
2. 필요한 메서드가 없으면 → 해당 담당자에게 Slack DM으로 요청
3. 직접 Repository를 import하여 쿼리 작성 금지 (서비스 레이어 경유 필수)
4. 예외: 집계 쿼리는 Stats 모듈 내에서 직접 JOIN 쿼리 작성 가능 (성능 고려)
```

---

### 🟥 W3 — 통합·UI 마감·시연 준비

**Dev D W3 집중 목표:** "전체 화면 UI 통합 + 시연 시나리오 흐름 완성 + 시연 준비 총괄"

| 일자 | 오전 | 오후 | 저녁 |
| --- | --- | --- | --- |
| **W3 월** | Stand-up. 전체 화면 통합 상태 점검. | **공통 레이아웃 최종 완성** — 내비게이션 연결, 권한별 메뉴 분기. | 모든 화면 내비게이션 동작 확인. |
| **W3 화** | Stand-up. Feature Freeze 확인. | 버그 수정 + UI 일관성 정리 (색상·폰트·여백). | 접근성 기본 점검 (alt 태그, form label). |
| **W3 수** | Stand-up. | **시연 시나리오 UI 흐름 완성**. 시연용 데이터 시드 설계. | 시연 시나리오 1차 리뷰 (Lead와 함께). |
| **W3 목** | Stand-up. | **15_사용자_메뉴얼 초안 작성** (화면 스크린샷 포함). 시드 데이터 삽입 지원. | 사용자 매뉴얼 초안 완성. |
| **W3 금** | Stand-up. | **W3 게이트 + 시연 1차 리허설 (16:00)**. | W3 회고. 블로그 초안. |

**W3 Dev D 산출물**

| 산출물 | 완료 기준 |
| --- | --- |
| 전체 UI 통합 | 모든 화면에서 공통 레이아웃 동작 + 내비게이션 정상 |
| 시연 시나리오 흐름 | 시연 4가지 시나리오 전체 UI에서 이동 가능 |
| 15_사용자_매뉴얼 초안 | 주요 기능 화면 스크린샷 + 설명 작성 |
| Admin/Stats Service 커버리지 | 80%+ (JaCoCo 확인) |
| 통합 테스트 | 관리자 권한 + 일반 사용자 권한 분기 E2E 통과 |

---

### ⬜ W4 — 시연 준비 + 발표

| 일자 | 활동 | 산출물 |
| --- | --- | --- |
| W4 월 | **19_시연_준비_가이드 작성** (시연 체크리스트, 시나리오, Q&A 30선). 발표 슬라이드 초안. | 시연 가이드 초안 |
| W4 화 | 시연 리허설 1회차. 발표자 역할 담당 시 내용 숙지. | 피드백 반영 |
| W4 수 | 백업 영상 녹화 참여. **기술 블로그 1편 완성**. | 블로그 1편 |
| W4 목 | 시연 리허설 2회차. 18_포트폴리오_README 최종 검토. | README 최종본 |
| W4 금 | 시연 D-Day. 시연 시나리오 화면 담당. 회고 참여. | 회고 기여 |

---

## 4. 기술 기준 & 코딩 지침

### 4.1 관리자 권한 기준

```java
// ✅ 관리자 엔드포인트: 반드시 권한 체크
@PreAuthorize("hasRole('ADMIN')")
@GetMapping("/admin/members")
public String memberList(Model model) { ... }

// 또는 SecurityConfig에서 경로 기반 권한 설정 (Lead 소유 — 변경 요청 필요)
.requestMatchers("/admin/**").hasRole("ADMIN")

// ✅ 관리자 서비스: 권한 체크 이중화 (Controller + Service 양쪽)
@PreAuthorize("hasRole('ADMIN')")
public AdminDashboardResponse getDashboard() { ... }
```

### 4.2 통계 집계 쿼리 기준

```java
// ✅ 집계 쿼리 — JPQL 또는 Native Query (성능 고려)
@Query("SELECT new com.example.stats.dto.StatsResponse(" +
       "DATE(m.createdAt), COUNT(m)) " +
       "FROM {{ModuleB}} m " +
       "WHERE m.createdAt BETWEEN :start AND :end " +
       "GROUP BY DATE(m.createdAt) " +
       "ORDER BY DATE(m.createdAt)")
List<StatsResponse> findDailyStats(@Param("start") LocalDateTime start,
                                   @Param("end") LocalDateTime end);

// ✅ 대용량 집계: 인덱스 설계 Lead와 협의 필수
// ✅ 캐싱 고려: 자주 조회되는 통계는 Redis 캐시 (Lead 협의)

// ✅ 집계 결과 → Chart.js JSON 형식
public ChartDataResponse toChartData(List<StatsResponse> stats) {
    return ChartDataResponse.builder()
        .labels(stats.stream().map(s -> s.getDate().toString()).toList())
        .data(stats.stream().map(StatsResponse::getCount).toList())
        .build();
}
```

### 4.3 공통 레이아웃 기준

```html
{{! header.mustache — 모든 페이지에서 include }}
<header class="navbar navbar-expand-lg navbar-dark bg-dark">
  <div class="container">
    <a class="navbar-brand" href="/">{{projectName}}</a>
    <div class="navbar-nav ms-auto">
      {{#isLoggedIn}}
        {{#isAdmin}}
          <a class="nav-link" href="/admin">관리자</a>
        {{/isAdmin}}
        <a class="nav-link" href="/mypage">{{currentUser}}</a>
        <a class="nav-link" href="/auth/logout">로그아웃</a>
      {{/isLoggedIn}}
      {{^isLoggedIn}}
        <a class="nav-link" href="/auth/login">로그인</a>
      {{/isLoggedIn}}
    </div>
  </div>
</header>

{{! 사용 예시 — 각 mustache 파일에서 }}
{{> layout/header}}
<main class="container mt-4">
  {{! 본문 }}
</main>
{{> layout/footer}}
```

### 4.4 Chart.js 연동 기준

```html
{{! stats-dashboard.mustache }}
<canvas id="dailyChart"></canvas>
<script>
  const labels = {{{statsLabels}}};  // Mustache Triple Brace (HTML escape 방지)
  const data = {{{statsData}}};

  new Chart(document.getElementById('dailyChart'), {
    type: 'bar',
    data: { labels, datasets: [{ label: '일별 {{단위}}', data }] },
    options: { responsive: true }
  });
</script>
```

---

## 5. AI 에이전트 활용 가이드 (Dev D 특화)

| 작업 | 도구 | 프롬프트 예시 |
| --- | --- | --- |
| 레이아웃 | Claude Code | "Bootstrap 5 기반 Spring Mustache 레이아웃. header/footer/sidebar 포함. 권한에 따라 관리자 메뉴 분기." |
| 집계 쿼리 | Claude Code | "JPQL로 {{ModuleB}} 테이블에서 날짜별 COUNT 집계 쿼리 작성. 파라미터: startDate, endDate. GROUP BY DATE." |
| Chart.js 차트 | Claude Code | "Chart.js 바 차트 작성. 데이터: {{labels}}, {{data}}. Responsive, 타이틀 포함. Spring Mustache에서 동작." |
| 사용자 매뉴얼 | Claude 웹 | "이 스크린샷들을 보고 {{PROJECT_NAME}} 사용자 매뉴얼을 작성해줘. 회원 가입부터 {{핵심기능}} 사용까지 단계별." |
| 관리자 서비스 | Claude Code | "@{{ModuleA}}Service.java, @{{ModuleB}}Service.java의 public 메서드를 확인하고, AdminService의 getDashboard() 메서드를 작성해줘. 집계 요약 포함." |

---

## 6. 협업 포인트

| 협업 대상 | 시점 | 내용 |
| --- | --- | --- |
| **Lead** | W1 월 | 공통 레이아웃 구조 + Mustache include 방식 확인 |
| **Dev A** | W2 월 | 회원 목록 조회 API 확인 (`MemberService.findAll()`) |
| **Dev B** | W2 화 | 관리자에서 모듈B 조회·상태 변경 API 확인 |
| **Dev C** | W2 화 | 관리자에서 모듈C 조회 API 확인 |
| **DevOps/QA** | W3 수 | 시연 시나리오 화면 흐름 E2E 테스트 협의 |
| **전원** | W4 화 | 시연 리허설 — 화면 전환 흐름 최종 점검 |

---

## 7. 블로커 에스컬레이션

```
막혔을 때 행동 순서:
1. 스스로 30분 이내 해결 시도 (검색, AI 보조)
2. 30분 이상 막히면 → #dev 채널에 "🔴 블로커: [내용] — [예상 영향]" 공지 + Lead DM
3. Lead 60분 내 응답 없으면 → 해당 기능 잠시 보류 + 다음 작업으로 전환
4. 일일 회고에서 반드시 공유

Admin·Stats 관련 타 모듈·Lead 소유 코드 수정이 필요할 때:
→ #dev 채널에 변경 요청 공지 → 해당 소유자 승인 후 진행 (직접 수정 금지)
타 모듈 Service 메서드 추가 요청 시: 해당 모듈 소유자(Dev A/B/C)에게 Slack DM → 구현 요청
집계 쿼리 성능 이슈: Lead와 DB 인덱스 설계 협의 후 진행
```


## 8. 기술 블로그 주제 가이드

| 주제 | 내용 | 어필 포인트 |
| --- | --- | --- |
| "Chart.js + Spring Mustache 통계 대시보드" | 집계 쿼리 + JSON → 차트 | 풀스택 구현 경험 |
| "관리자 페이지 설계 — RBAC 적용" | 역할 기반 접근 제어 구현 | 보안 이해 |
| "Spring Mustache 레이아웃 시스템" | 공통 레이아웃 + 컴포넌트화 | SSR 아키텍처 |
| "6명 시연을 위한 UI 통합 경험" | 협업 + 통합 어려움 + 해결 | 협업 역량 |

---

## 9. 일일 체크리스트 (Dev D용)

### 매일 Stand-up 전 (09:00)

```
[ ] develop 최신화 (git pull origin develop)
[ ] 어제 작업한 브랜치 CI 상태 확인
[ ] 오늘 집중할 작업 1가지 결정 (너무 많이 잡지 않기)
[ ] 리뷰 요청 대기 중인 PR 없는지 확인
```

### 코드 작성 중

```
[ ] 관리자 엔드포인트: @PreAuthorize("hasRole('ADMIN')") 또는 SecurityConfig 체크
[ ] 타 모듈 Service 사용 시: 직접 Repository 접근 안 했는가
[ ] 집계 쿼리: 인덱스·성능 Lead와 협의했는가
[ ] 공통 레이아웃 변경 시: 전체 페이지 영향도 확인
[ ] Chart.js: Mustache Triple Brace {{{data}}} 사용 (XSS 방지)
```

### PR 올리기 전

```
[ ] ./gradlew test 로컬 통과
[ ] 관리자 권한 없는 사용자가 /admin 접근 시 403 동작 확인
[ ] PR 크기 400줄 이하
[ ] AI 코드 환각 체크 (존재하지 않는 메서드 없음)
[ ] API Key / 비밀번호 평문 하드코딩 없음
[ ] 화면 변경 PR은 스크린샷 첨부 권장
```

---

## 10. 성공 기준 (Dev D 개인)

| 기준 | 목표 | 측정 방법 |
| --- | --- | --- |
| 전체 UI 통합 | 모든 팀원 화면이 공통 레이아웃으로 일관됨 | 시연 리허설 |
| 관리자 기능 | CRUD + 통계 대시보드 완전 동작 | 데모 |
| Service 커버리지 | 80%+ (W3 기준) | JaCoCo |
| 시연 준비 | 19_시연 가이드 완성 + 리허설 2회 | 리허설 기록 |
| 기술 블로그 | 1편 이상 | 노션 링크 |

---

## 📋 Dev D 체크리스트 총괄

```
W1:
[ ] 공통 레이아웃 (header/footer/sidebar) develop 머지
[ ] 관리자 대시보드 Mock 화면 완성
[ ] 통계 화면 기본 Chart.js 동작
[ ] 화면 정의서 관리자 파트 완성

W2:
[ ] 관리자 실데이터 연동 완성 (전체 모듈)
[ ] 통계 집계 쿼리 + 차트 완성
[ ] Service 커버리지 60%+ (W2) → 80%+ (W3)

W3:
[ ] 전체 UI 내비게이션 통합
[ ] 시연 시나리오 화면 흐름 완성
[ ] 15_사용자_매뉴얼 초안 완성
[ ] 통합 테스트 통과

W4:
[ ] 19_시연_준비_가이드 완성
[ ] 시연 리허설 2회 통과
[ ] 기술 블로그 1편 발행
```
