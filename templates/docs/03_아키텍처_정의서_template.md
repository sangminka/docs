# {{EMOJI}} {{PROJECT_NAME}} — 프로젝트 아키텍처 정의서 v1.0

> **문서 버전:** v1.0
> **작성일:** YYYY-MM-DD
> **연관 문서:** 프로젝트 계획서 / ERD / API 명세서

---

## 목차
1. 전체 시스템 레이어 구조
2. 기술 스택
3. Spring Boot 패키지 구조 & 소유자 맵핑
4. 담당자별 소유 영역 전체 지도
5. 공유 레이어 — 책임개발자 단독 소유
6. 기능 모듈별 경계 정의
7. 세션 저장소 설정
8. 서버 사이드 유효성 검증 패턴
9. CSRF 토큰 + AJAX 처리
10. 페이징 구현 패턴

---

## 1. 전체 시스템 레이어 구조

```
Browser
  │ HTTPS
  ▼
Nginx (80/443) ── SSL 종단
  │
  ▼
Spring Boot App (8080)
  ├─ Filter: Security, CSRF
  ├─ Interceptor: LayoutModelInterceptor
  ├─ Controller (SSR / @RestController)
  ├─ Service
  ├─ Repository (Spring Data JPA)
  ├─ Session ─── Redis
  └─ External ── {{외부 API (LLM / 결제 / 메시지)}}
            │
            ▼
         MySQL 8.0
```

---

## 2. 기술 스택

| 영역 | 기술 | 버전 | 비고 |
| --- | --- | --- | --- |
| Runtime | JDK | 21 | LTS |
| Framework | Spring Boot | 3.x | |
| View | Mustache | | SSR |
| DB | MySQL | 8.0 | |
| ORM | Spring Data JPA | | |
| Session | Redis | 7.0 | Spring Session Data Redis |
| HTTP Client | Spring WebClient | | 외부 API 호출 |
| Build | Gradle | | |
| Test | JUnit 5 + MockMvc | | |
| Deploy | Docker Compose + Nginx | | |

---

## 3. Spring Boot 패키지 구조 & 소유자 맵핑

```
com.example.{{project}}
├── {{project}}Application.java       [Lead]
├── config/                           [Lead]
│   ├── SecurityConfig.java
│   ├── RedisSessionConfig.java
│   └── WebMvcConfig.java
├── common/                           [Lead]
│   ├── interceptor/LayoutModelInterceptor.java
│   ├── exception/GlobalExceptionHandler.java
│   └── dto/ApiResponse.java
├── domain/
│   ├── auth/                         [Dev A — Lead와 공동]
│   │   ├── AuthController.java
│   │   ├── AuthService.java
│   │   └── Member.java (Entity)
│   ├── {{moduleA}}/                  [Dev A]
│   │   ├── {{ModuleA}}Controller.java
│   │   ├── {{ModuleA}}Service.java
│   │   └── {{ModuleA}}Repository.java
│   ├── {{moduleB}}/                  [Dev B]
│   │   └── ...
│   ├── {{moduleC}}/                  [Dev C]
│   │   └── ...
│   ├── admin/                        [Dev D]
│   │   └── AdminController.java
│   └── stats/                        [Dev D]
│       └── StatsService.java
└── external/                         [Lead — 공유 레이어]
    └── {{external_api}}/
        ├── {{External}}Client.java
        └── {{External}}Response.java
```

---

## 4. 담당자별 소유 영역 전체 지도

### 4.1 책임개발자 (Tech Lead)
- `config/`, `common/`, `domain/` 전체
- 공유 서비스: {{CoreService}}, LayoutModelInterceptor, {{External}}Client, SecurityFilter
- 배포, CI, 문서 정합 관리

### 4.2 개발자 A — {{MODULE_A}}
- `domain/{{moduleA}}/**` 전체
- 관련 Mustache 템플릿: `templates/{{moduleA}}/**`
- 관련 테스트: `test/**/{{moduleA}}/**`

### 4.3 개발자 B — {{MODULE_B}}
- `domain/{{moduleB}}/**`
- ...

### 4.4 개발자 C — {{MODULE_C}}
- `domain/{{moduleC}}/**`
- ...

---

## 5. 공유 레이어 — 책임개발자 단독 소유

### 5.1 {{CoreService}} — {{핵심 공유 비즈니스 로직}} (해당 시)
### 5.2 LayoutModelInterceptor — 공통 레이아웃 Model 자동 주입

```java
// 모든 요청에서 자동으로 현재 사용자·CSRF·레이아웃 정보 주입
@Component
public class LayoutModelInterceptor implements HandlerInterceptor {
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response,
                           Object handler, ModelAndView mv) {
        if (mv != null) {
            // 현재 인증 사용자 정보
            Authentication auth = SecurityContextHolder.getContext().getAuthentication();
            if (auth != null && auth.isAuthenticated()) {
                mv.addObject("currentUser", auth.getName());
                mv.addObject("isLoggedIn", true);
                mv.addObject("isAdmin", auth.getAuthorities().stream()
                    .anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN")));
            } else {
                mv.addObject("isLoggedIn", false);
            }
        }
    }
}
```

### 5.3 {{External}}Client — 외부 API 클라이언트 (Lead 소유)

```java
// external/{{external_api}}/{{External}}Client.java
@Component
@RequiredArgsConstructor
public class {{External}}Client {

    private final WebClient webClient;

    @Value("${external.api.key}")
    private String apiKey;

    public {{External}}Response call({{External}}Request request) {
        return webClient.post()
            .uri("/api/v1/{{endpoint}}")
            .header("Authorization", "Bearer " + apiKey)
            .bodyValue(request)
            .retrieve()
            .bodyToMono({{External}}Response.class)
            .timeout(Duration.ofSeconds(10))
            .block();
    }
}
```

### 5.4 Entity 클래스 — 변경 금지 (schema 변경은 PR 리뷰 필수)

---

## 6. 기능 모듈별 경계 정의

### 모듈 A — {{MODULE_A_NAME}} (개발자 A)
- **의존:** {{ENTITY_1}}, {{ENTITY_2}}
- **제공:** 외부 사용자 진입 화면, {{기능}}
- **인터페이스(서비스 메서드 시그니처):**
  - `{{MODULE_A}}Service.createX(request): Result`

### 모듈 B — {{MODULE_B_NAME}} (개발자 B)
- ...

---

## 7. Redis 세션 저장소 설정

**의존성 (build.gradle)**
```gradle
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
implementation 'org.springframework.session:spring-session-data-redis'
```

**설정 (application.properties)**
```properties
spring.session.store-type=redis
spring.session.redis.namespace=spring:session:{{project}}
spring.data.redis.host=localhost
spring.data.redis.port=6379
server.servlet.session.timeout=30m
```

---

## 8. 서버 사이드 유효성 검증 패턴

**원칙:** 모든 외부 입력은 `@Valid` 또는 `@Validated`로 검증.

**SSR 폼 처리 (PRG 패턴)**
```java
@PostMapping("/{{resource}}")
public String create(@Valid @ModelAttribute {{Resource}}Form form,
                     BindingResult bindingResult,
                     RedirectAttributes ra) {
    if (bindingResult.hasErrors()) {
        return "{{resource}}/form";  // 같은 페이지 재렌더
    }
    service.create(form);
    ra.addFlashAttribute("message", "등록되었습니다");
    return "redirect:/{{resource}}";
}
```

**REST API**
```java
@PostMapping("/api/{{resource}}")
public ResponseEntity<ApiResponse<{{Resource}}Dto>> create(
        @Valid @RequestBody {{Resource}}Request req) {
    return ResponseEntity.ok(ApiResponse.success(service.create(req)));
}
```

---

## 9. CSRF 토큰 + AJAX 처리

**Mustache 메타 태그 방식 (권장)**
```html
<meta name="_csrf" content="{{_csrf.token}}">
<meta name="_csrf_header" content="{{_csrf.headerName}}">
```

```javascript
const token = document.querySelector('meta[name="_csrf"]').content;
const header = document.querySelector('meta[name="_csrf_header"]').content;
fetch('/api/...', { headers: { [header]: token, 'Content-Type': 'application/json' }, ... });
```

---

## 10. 페이징 구현 패턴

```java
@GetMapping("/{{resource}}")
public String list(@PageableDefault(size = 20) Pageable pageable, Model model) {
    Page<{{Resource}}Dto> page = service.findAll(pageable);
    model.addAttribute("page", page);
    return "{{resource}}/list";
}
```

Mustache에서 페이지 번호/이전·다음 처리 파셜(`partials/pagination.mustache`)을 공유 컴포넌트로 둔다.

---

## 11. ADR (Architecture Decision Record)

> 팀이 내린 기술 결정을 기록한다. Lead가 작성하고 PR에 포함한다.

### ADR-001: 인증 방식 — 세션 기반 선택

| 항목 | 내용 |
| --- | --- |
| **날짜** | YYYY-MM-DD |
| **상태** | 승인 |
| **결정** | Spring Security 세션 + Redis 기반 인증 |
| **대안** | JWT (Stateless) |
| **이유** | SSR(Mustache) 환경에서 세션이 자연스럽고, Redis 세션으로 수평 확장 가능. JWT는 토큰 무효화가 복잡. |
| **결과** | Redis 의존성 추가. 단일 서버 이상 환경 시 Redis Cluster 필요. |

### ADR 템플릿 (복제해서 사용)

```markdown
### ADR-{{번호}}: {{결정 제목}}

| 항목 | 내용 |
| --- | --- |
| **날짜** | YYYY-MM-DD |
| **상태** | 제안 / 승인 / 폐기 |
| **결정** | |
| **대안** | |
| **이유** | |
| **결과** | |
```

---

## 12. 에러 페이지 매핑

```java
// GlobalExceptionHandler.java (Lead 소유)
@ControllerAdvice
public class GlobalExceptionHandler {

    // 도메인 예외 — JSON 또는 리디렉션
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ApiResponse<?>> handleBusiness(BusinessException e) {
        return ResponseEntity.status(e.getErrorCode().getStatus())
            .body(ApiResponse.error(e.getErrorCode()));
    }

    // 인가 실패 — 403 페이지
    @ExceptionHandler(AccessDeniedException.class)
    public String handleForbidden() {
        return "error/403";
    }

    // 존재하지 않는 리소스 — 404 페이지
    @ExceptionHandler(EntityNotFoundException.class)
    public String handleNotFound() {
        return "error/404";
    }

    // 서버 에러 — 500 페이지
    @ExceptionHandler(Exception.class)
    public String handleGeneral() {
        return "error/500";
    }
}
```

```
src/main/resources/templates/error/
├── 403.mustache    ← "접근 권한이 없습니다"
├── 404.mustache    ← "페이지를 찾을 수 없습니다"
└── 500.mustache    ← "일시적인 오류가 발생했습니다"
```

---

## 13. Spring Actuator & 헬스체크 설정

```properties
# application.properties
management.endpoints.web.exposure.include=health,info,metrics
management.endpoint.health.show-details=when-authorized
management.endpoint.health.probes.enabled=true
management.info.env.enabled=true

info.app.name=${spring.application.name}
info.app.version=1.0.0
```

```yaml
# 헬스체크 엔드포인트 (DevOps/QA 활용)
GET /actuator/health        → 전체 상태
GET /actuator/health/db     → DB 연결
GET /actuator/health/redis  → Redis 연결
GET /actuator/info          → 앱 정보
```

---

## 14. 로깅 전략

```xml
<!-- logback-spring.xml -->
<configuration>
    <springProfile name="dev">
        <root level="DEBUG">
            <appender-ref ref="CONSOLE"/>
        </root>
        <!-- SQL 로깅 (N+1 감지용) -->
        <logger name="org.hibernate.SQL" level="DEBUG"/>
        <logger name="org.hibernate.type.descriptor.sql" level="TRACE"/>
    </springProfile>

    <springProfile name="prod">
        <root level="INFO">
            <appender-ref ref="FILE"/>
        </root>
        <!-- 운영에서는 SQL 로깅 OFF -->
    </springProfile>
</configuration>
```

**로그 레벨 기준**

| 레벨 | 사용 기준 |
| --- | --- |
| ERROR | 시스템 오류, 외부 API 실패, DB 연결 실패 |
| WARN | 폴백 처리 진입, 비정상 요청, 성능 임계 초과 |
| INFO | 핵심 비즈니스 이벤트 (예약 생성, 결제 완료) |
| DEBUG | 개발 환경 디버깅용 (운영 비활성화) |

---

## 📌 변경 이력

| 버전 | 날짜 | 작성자 | 주요 변경 |
| --- | --- | --- | --- |
| v1.0 | YYYY-MM-DD | Lead | 초기 작성 |
