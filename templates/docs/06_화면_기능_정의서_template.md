# {{EMOJI}} {{PROJECT_NAME}} — 전체 화면 목록 및 화면별 기능 정의서 v1.0

> **문서 버전:** v1.0
> **작성일:** YYYY-MM-DD
> **연관 문서:** API 명세서 v1.0 / 시퀀스 다이어그램 v1.0 / 스토리보드 v1.0

---

## 목차
1. 화면 목록 전체 요약
2. 공통 레이아웃 컴포넌트
3. 외부 — 비로그인 사용자 화면
4. 공통 — 인증 화면
5. ROLE_{{A}} 화면
6. ROLE_{{B}} 화면
7. ROLE_{{C}} 화면
8. ROLE_ADMIN 화면
9. 공통 — 내 정보관리 화면

---

## 1. 화면 목록 전체 요약

| # | 화면명 | Role | Layout | URL | 주요 API |
| --- | --- | --- | --- | --- | --- |
| 00 | 메인 | - | external | `/` | - |
| 01 | 로그인 | - | auth | `/login` | `POST /login` |
| 02 | {{A}} 대시보드 | ROLE_{{A}} | internal | `/{{role_a}}/dashboard` | `GET /{{role_a}}/dashboard` |
| ... | ... | ... | ... | ... | ... |

---

## 2. 공통 레이아웃 컴포넌트

### 2.1 레이아웃 유형 정의

| 유형 | 대상 | 구성 |
| --- | --- | --- |
| `external` | 비로그인 | 헤더(로고) + 메인 + 푸터 |
| `auth` | 로그인/회원가입 | 중앙 정렬 카드 |
| `internal` | 로그인 | 헤더 + 사이드바(Role별) + 메인 + 푸터 |

### 2.2 헤더 컴포넌트
- 좌: 로고 (클릭 시 role별 대시보드로)
- 우: 사용자명 / 알림 / 로그아웃

### 2.3 사이드 메뉴 컴포넌트 (Role별 노출)

| Role | 메뉴 |
| --- | --- |
| ROLE_{{A}} | 대시보드 / {{리소스1}} / 내 정보 |
| ROLE_{{B}} | 대시보드 / {{리소스2}} / 내 정보 |
| ROLE_ADMIN | 대시보드 / 사용자 / 설정 / 통계 |

### 2.4 푸터 컴포넌트

### 2.5 전체 레이아웃 조합 예시
```
┌──────────────────────────────┐
│ 헤더                         │
├──────┬───────────────────────┤
│      │                       │
│ 사이 │  메인 콘텐츠          │
│ 드바 │                       │
│      │                       │
├──────┴───────────────────────┤
│ 푸터                         │
└──────────────────────────────┘
```

### 2.6 Mustache 템플릿 구조
```
templates/
├── layout.mustache          ← 기본 골격
├── partials/
│   ├── header.mustache
│   ├── sidebar-{{role_a}}.mustache
│   ├── sidebar-{{role_b}}.mustache
│   ├── sidebar-admin.mustache
│   ├── footer.mustache
│   └── pagination.mustache
├── {{role_a}}/
│   ├── dashboard.mustache
│   └── {{resource}}/list.mustache
└── ...
```

### 2.7 Spring Model 필수 전달 데이터
- `user` — 현재 로그인 사용자 (username, role, displayName)
- `csrf` — CSRF 토큰
- `flash.message` — 플래시 메시지
- `layoutType` — external / auth / internal
- `activeMenu` — 활성 메뉴 키

---

## 3. 외부 — 비로그인 사용자 화면

### 📄 화면 00 — 메인
- **레이아웃:** external
- **URL:** `GET /`
- **Role:** 비로그인
- **Model 데이터:** {{공지사항 리스트}}
- **주요 컴포넌트:** 히어로 섹션 / CTA 버튼 / 공지 / 푸터
- **상태 전이:** -
- **연관 API:** -

### 📄 화면 01 — {{진입점}}
- ...

---

## 4. 공통 — 인증 화면

### 📄 화면 — 로그인
- **레이아웃:** auth
- **URL:** `GET /login`, `POST /login`
- **Role:** 비로그인
- **Model 데이터:** `error?`
- **주요 컴포넌트:** username, password, 로그인 버튼
- **연관 API:** `POST /login`

---

## 5. ROLE_{{A}} 화면

### 📄 화면 NN — {{A}} 대시보드
- **레이아웃:** internal (사이드바 = sidebar-{{role_a}})
- **URL:** `GET /{{role_a}}/dashboard`
- **Role:** ROLE_{{A}}
- **Model 데이터:** `stats`, `todayList`
- **주요 컴포넌트:** 통계 카드 / 오늘 할 일 목록
- **상태 전이:** -
- **연관 API:** `GET /{{role_a}}/dashboard`

### 📄 화면 NN — {{리소스}} 목록
- **레이아웃:** internal
- **URL:** `GET /{{role_a}}/{{resource}}`
- **Model 데이터:** `page` (Page<Dto>), `q`, `status`
- **주요 컴포넌트:** 검색 / 필터 / 페이징 (20건)
- **연관 API:** `GET /{{role_a}}/{{resource}}`

### 📄 화면 NN — {{리소스}} 상세/수정
- ...

---

<!-- ROLE_B, ROLE_C, ROLE_ADMIN 섹션을 같은 패턴으로 복제 -->

---

## 9. 공통 — 내 정보관리 화면

### 📄 화면 NN — 내 정보 조회
- **URL:** `GET /me`
- **Model:** 현재 사용자 정보

### 📄 화면 NN — 내 정보 수정
- **URL:** `GET /me/edit`, `POST /me`
- **Model:** 현재 사용자 정보 + form binding

### 📄 화면 NN — 비밀번호 변경
- **URL:** `GET /me/password`, `POST /me/password`

---

## 📌 변경 이력

| 버전 | 날짜 | 작성자 | 주요 변경 |
| --- | --- | --- | --- |
| v1.0 | YYYY-MM-DD | Lead | 초기 작성 |
