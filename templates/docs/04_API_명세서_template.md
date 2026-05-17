# {{EMOJI}} {{PROJECT_NAME}} — API 명세서 v1.0

> **문서 버전:** v1.0
> **작성일:** YYYY-MM-DD
> **연관 문서:** ERD v1.0 / 아키텍처 정의서 v1.0

---

## 목차
1. 공통 규칙
2. 인증 API
3. 외부 API
4. {{기능 1}} API
5. ROLE_{{A}} API
6. ROLE_{{B}} API
7. ROLE_{{C}} API
8. ROLE_ADMIN API
9. 공통 — 내 정보관리 API

---

## 1. 공통 규칙

### 1.1 Base URL
- 개발: `http://localhost:8080`
- 운영: `https://{{도메인}}`

### 1.2 URL 설계 원칙
- SSR 경로: RPC 스타일 계층형 — `/{{role}}/{{resource}}/{{action}}`
- REST API: `/api/{{resource}}` — PUT/PATCH/DELETE 사용

### 1.3 URL 접두어별 접근 권한

| 접두어 | 접근 권한 |
| --- | --- |
| `/` (비로그인 가능) | ALL |
| `/{{role1}}/**` | ROLE_{{A}} |
| `/{{role2}}/**` | ROLE_{{B}} |
| `/admin/**` | ROLE_ADMIN |
| `/api/**` | 인증 필요 (세션) |

### 1.4 HTTP 메서드 규칙

| 동작 | SSR | REST API |
| --- | --- | --- |
| 조회 | GET | GET |
| 생성 | POST (폼 제출) | POST |
| 수정 | POST | PUT / PATCH |
| 삭제 | POST | DELETE |

### 1.5 컨트롤러 반환 규칙

| 패턴 | 반환 타입 | 예 |
| --- | --- | --- |
| SSR 화면 | `String` (view name) | `"user/list"` |
| SSR 폼 처리 후 | `String` (redirect) | `"redirect:/user"` |
| REST API | `ResponseEntity<ApiResponse<T>>` | `ResponseEntity.ok(...)` |

### 1.6 JSON API 레이어 (`/api/**`)
- 공통 응답 포맷:
  ```json
  {
    "success": true,
    "data": { ... },
    "error": null,
    "timestamp": "2026-04-23T10:00:00Z"
  }
  ```
- 오류 응답:
  ```json
  {
    "success": false,
    "data": null,
    "error": { "code": "VALIDATION_ERROR", "message": "...", "fields": {"field": "msg"} }
  }
  ```

### 1.7 서버 사이드 유효성 검증 패턴
- 모든 입력은 `@Valid` 적용
- 에러 필드는 `BindingResult` → Mustache에서 `errors.fieldName` 표시

### 1.8 {{특수 검증 규칙}} (예: 의사 진료 요일 검증)
- ...

### 1.9 인증 오류 공통 처리
- 비인증 접근: `302 /login`
- 권한 부족: `403` + `error/403.mustache`
- API 비인증: `401` + JSON
### 1.10 HTTP 상태 코드 사용 기준

| 상태 코드 | 의미 | 사용 시점 |
| --- | --- | --- |
| 200 OK | 성공 | 조회·수정 성공 |
| 201 Created | 생성 성공 | POST 생성 성공 (REST API) |
| 302 Found | 리디렉션 | SSR 폼 처리 후 PRG 패턴 |
| 400 Bad Request | 잘못된 요청 | 유효성 검증 실패 |
| 401 Unauthorized | 비인증 | 로그인 필요 (API 요청) |
| 403 Forbidden | 권한 없음 | 로그인됐지만 권한 부족 |
| 404 Not Found | 리소스 없음 | 존재하지 않는 ID 조회 |
| 409 Conflict | 충돌 | 중복 데이터 (이메일 중복 등) |
| 500 Internal Server Error | 서버 오류 | 예상치 못한 서버 에러 |

### 1.11 공통 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
| --- | --- | --- |
| `VALIDATION_ERROR` | 400 | @Valid 유효성 검증 실패 |
| `INVALID_INPUT` | 400 | 비즈니스 규칙 위반 입력 |
| `UNAUTHORIZED` | 401 | 비인증 접근 |
| `FORBIDDEN` | 403 | 권한 없는 접근 |
| `NOT_FOUND` | 404 | 리소스 없음 |
| `DUPLICATE_EMAIL` | 409 | 이메일 중복 |
| `INVALID_STATUS_TRANSITION` | 409 | 유효하지 않은 상태 전이 |
| `EXTERNAL_API_ERROR` | 503 | 외부 API 오류 (폴백 처리 중) |
| `INTERNAL_ERROR` | 500 | 서버 내부 오류 |

```java
// ErrorCode enum (Lead 소유 — common/exception/ErrorCode.java)
@Getter
@RequiredArgsConstructor
public enum ErrorCode {
    VALIDATION_ERROR(HttpStatus.BAD_REQUEST, "VALIDATION_ERROR", "입력값이 올바르지 않습니다"),
    NOT_FOUND(HttpStatus.NOT_FOUND, "NOT_FOUND", "리소스를 찾을 수 없습니다"),
    DUPLICATE_EMAIL(HttpStatus.CONFLICT, "DUPLICATE_EMAIL", "이미 사용 중인 이메일입니다"),
    // ...
    ;
    private final HttpStatus status;
    private final String code;
    private final String message;
}
```

### 1.12 페이징 응답 형식

```json
{
  "success": true,
  "data": {
    "content": [ { "id": 1, "..." } ],
    "pageNumber": 0,
    "pageSize": 20,
    "totalElements": 150,
    "totalPages": 8,
    "first": true,
    "last": false
  }
}
```

```java
// 페이징 요청 파라미터 (모든 목록 API 공통)
// ?page=0&size=20&sort=createdAt,desc
@GetMapping("/{{resource}}")
public String list(@PageableDefault(size = 20, sort = "createdAt",
                   direction = Sort.Direction.DESC) Pageable pageable,
                   Model model) { ... }
```

---

## 2. 인증 API

### 2.1 로그인 화면
- **Method + URL:** `GET /login`
- **인증:** 비로그인
- **Response:** `login.mustache`
- **Model:** -

### 2.2 로그인 처리
- **Method + URL:** `POST /login`
- **인증:** 비로그인
- **Request:** form `username`, `password`, `_csrf`
- **Response:**
  - 성공: role 기반 redirect (예: STAFF → `/staff/dashboard`)
  - 실패: `redirect:/login?error`

### 2.3 로그아웃
- **Method + URL:** `POST /logout`
- **Response:** `redirect:/`

---

## 3. 외부 API (비로그인 사용자)

### 3.1 메인 화면
- **Method + URL:** `GET /`
- **인증:** 비로그인
- **Response:** `external/main.mustache`
- **Model:** `{{주요 공지, 안내 데이터}}`

### 3.2 ...

---

## 4. {{기능 1 이름}} API

### 4.1 {{세부 기능}}
- **Method + URL:** `POST /api/{{resource}}/{{action}}`
- **인증:** ...
- **Request:**
  ```json
  { "field1": "...", "field2": 123 }
  ```
- **Response:**
  ```json
  { "success": true, "data": {...} }
  ```
- **비고:** 타임아웃 5초, 실패 시 폴백 경로

---

## 5. ROLE_{{A}} API

### 5.1 {{A}} 대시보드 화면
- **Method + URL:** `GET /{{role_a}}/dashboard`
- **인증:** ROLE_{{A}}
- **Response:** `{{role_a}}/dashboard.mustache`
- **Model:** `{{통계, 오늘 할 일 목록}}`

### 5.2 {{리소스}} 목록 화면
- **Method + URL:** `GET /{{role_a}}/{{resource}}?page=0&size=20&q=검색어&status=...`
- **Response:** `{{role_a}}/{{resource}}/list.mustache`
- **Model:** `page`, `q`, `status`

### 5.3 {{리소스}} 상세 화면
- **Method + URL:** `GET /{{role_a}}/{{resource}}/{id}`

### 5.4 {{리소스}} 처리
- **Method + URL:** `POST /{{role_a}}/{{resource}}/{id}/{{action}}`
- **Request:** form fields
- **Response:** `redirect:/{{role_a}}/{{resource}}`

### 5.5 내 정보관리 화면
- **Method + URL:** `GET /{{role_a}}/me`
- **Response:** `{{role_a}}/me.mustache`

### 5.6 내 정보 수정 처리
- **Method + URL:** `POST /{{role_a}}/me`
- **Request:** form `phone`, `email`, `password?`
- **Response:** PRG — `redirect:/{{role_a}}/me`

---

<!-- ROLE_B, ROLE_C, ROLE_ADMIN 섹션을 같은 패턴으로 복제 -->

---

## 9. 공통 — 내 정보관리 API (모든 로그인 사용자)

### 9.1 내 정보 조회
- **Method + URL:** `GET /me`

### 9.2 비밀번호 변경
- **Method + URL:** `POST /me/password`
- **Request:** `oldPassword`, `newPassword`, `newPasswordConfirm`

---

## 📋 전체 API 요약

| # | Method | URL | 인증 | 설명 |
| --- | --- | --- | --- | --- |
| 1 | GET | `/` | - | 메인 |
| 2 | GET | `/login` | - | 로그인 화면 |
| ... | ... | ... | ... | ... |

---

## 📌 변경 이력

| 버전 | 날짜 | 작성자 | 주요 변경 |
| --- | --- | --- | --- |
| v1.0 | YYYY-MM-DD | Lead | 초기 작성 |
| v1-FINAL | YYYY-MM-DD | Lead | W2 금 API Lock — 이후 변경 시 팀 합의 필수 |
