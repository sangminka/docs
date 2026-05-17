# QT-AI ERD 변경 검토서 v1.0

> 기존 `02_ERD_문서.md`는 유지하고, 변경된 요구사항을 기준으로 ERD 보강 필요 여부를 검토한 별도 문서이다.

---

## 1. 검토 목적

현재 프로젝트에는 기존 ERD 문서가 있으며, 성경 본문 데이터(`성경/KJV.json`, `성경/88.json`)와 주석 데이터(`주석/refer.jsonl`)를 바탕으로 성경 본문, 주석 원문, 절 매핑, AI 생성 콘텐츠, 노트, 공유, 운영 로그 등의 구조가 설계되어 있다.

이 문서는 변경된 요구사항 명세서(`요구사항명세서_현재완성v1.md`)를 기준으로 기존 ERD가 그대로 사용 가능한지, 또는 어떤 부분을 보강해야 하는지 데이터 모델 관점에서 판단하기 위해 작성한다.

---

## 2. 종합 판단

기존 ERD는 폐기할 필요가 없다.

다만 변경된 요구사항 v3.1 기준으로는 일부 핵심 요구사항을 안정적으로 충족하기 위해 ERD 보강이 필요하다. 특히 다음 영역은 현재 ERD만으로는 요구사항을 완전히 만족하기 어렵다.

- 공유본 스냅샷 보존
- 검증 체크리스트 버전 추적
- AI 평가 셋 및 실패 사례 환류
- AI Q&A 신고 및 운영 검토
- 관리자, 자체 제작자, 시스템 배치 권한 분리
- 감사 로그의 수행 주체 확장
- 시뮬레이터 컴포넌트 라이브러리 버전 추적

반면 성경 본문 및 주석 원천 데이터 저장 구조는 현재 설계 방향이 적절하다.

---

## 3. 기존 ERD에서 유지해도 되는 영역

### 3.1 성경 본문 구조

기존 ERD의 `bible_books`, `bible_verses` 구조는 유지 가능하다.

현재 데이터 구조상 `KJV.json`과 `88.json`은 원본 JSON 구조가 다르지만, 성경 권, 장, 절 좌표를 기준으로 병합할 수 있다. 기존 ERD는 이 데이터를 `bible_verses`에 절 단위로 정규화해 저장하도록 설계되어 있으므로 요구사항의 다음 내용을 만족한다.

- 한글/영어 성경 본문을 절 좌표 기준으로 정렬
- QT 본문 범위는 실제 본문 텍스트가 아니라 절 좌표로 참조
- 일반 성경 조회와 QT 조회가 동일한 절 좌표 체계를 사용

단, `88.json`은 현재 파일 표시상 한글이 깨져 보이므로 DB 설계 문제가 아니라 데이터 적재 전 인코딩 검증이 필요하다.

### 3.2 주석 원문 및 절 매핑 구조

기존 ERD의 다음 구조는 유지 가능하다.

- `commentary_sources`
- `commentary_materials`
- `commentary_material_verses`

`refer.jsonl`은 단일 절 주석과 범위 주석을 모두 포함한다. 기존 ERD는 주석 원문을 `commentary_materials`에 한 번 저장하고, 포함 절 목록을 `commentary_material_verses`로 연결하는 방식이다.

이는 범위 주석을 절마다 중복 저장하지 않으면서 특정 절에 연결된 주석을 조회할 수 있으므로 적절하다.

### 3.3 QT 본문 구조

기존 ERD의 `qt_passages`, `qt_passage_verses` 구조도 유지 가능하다.

요구사항은 오늘의 QT 본문이 장절 중간에서 끊길 수 있고, 실제 본문 텍스트가 아니라 날짜별 범위 정보만 관리해야 한다고 되어 있다. 기존 구조는 시작 절, 종료 절, 표시 순서 기반 절 매핑을 모두 지원하므로 요구사항에 맞다.

### 3.4 노트 기본 구조

기존 ERD의 `notes`, `note_verses` 구조는 큰 방향에서 유지 가능하다.

`notes.category`로 QT 노트, 설교 노트, 기도제목, 회개, 감사일기를 구분하고, QT 노트의 4개 섹션을 컬럼으로 표현하는 방식은 MVP 범위에서는 실용적이다.

---

## 4. 변경이 필요한 영역

## 4.1 공유본 스냅샷 보존

### 현재 ERD 상태

현재 `sharing_posts`는 다음 핵심 컬럼을 가진다.

- `note_id`
- `member_id`
- `nickname_snapshot`
- `comments_enabled`
- `status`
- `published_at`

### 요구사항 변화

변경된 요구사항에서는 공유본 스냅샷을 다음과 같이 정의한다.

- 사용자가 공유를 선택한 시점의 본문, 노트 내용, 닉네임, 공개 설정을 별도 보존
- 원본 노트가 이후 수정되어도 공유본은 변경되지 않음
- 원본 노트가 삭제되거나 비공개로 전환되어도 공유본 표시 정책을 별도로 관리

### 문제점

현재 `sharing_posts`는 원본 노트의 `note_id`를 참조할 뿐, 공유 시점의 노트 본문을 별도로 저장하지 않는다.

따라서 원본 `notes`가 수정되면 공유 화면에서 과거 내용이 아니라 최신 노트 내용이 보일 가능성이 있다. 이는 “변경 불가 스냅샷” 요구사항과 충돌한다.

### 변경 필요 판단

변경 필요: 높음

### 권장 변경안

`sharing_posts`에 스냅샷 컬럼을 추가하는 방식을 권장한다. MVP에서는 별도 테이블보다 단순하고 조회가 쉽다.

추가 후보 컬럼:

| 컬럼 | 타입 | 설명 |
|---|---|---|
| note_category_snapshot | VARCHAR(30) | 공유 시점 노트 카테고리 |
| title_snapshot | VARCHAR(100) | 공유 시점 제목 |
| feeling_snapshot | TEXT | QT 노트 느낀 점 스냅샷 |
| memory_verse_snapshot | TEXT | QT 노트 기억할 구절 스냅샷 |
| application_snapshot | TEXT | QT 노트 적용할 점 스냅샷 |
| prayer_snapshot | TEXT | QT 노트 기도 스냅샷 |
| body_snapshot | TEXT | 자유 노트 본문 스냅샷 |
| verse_snapshot_json | JSON | 공유 시점 선택 구절 목록 |
| source_note_deleted_at | DATETIME(6) | 원본 노트 삭제 감지 시각 |

대안으로 `sharing_post_snapshots` 별도 테이블을 둘 수도 있다. 다만 공유글 1건당 스냅샷 1건이라면 `sharing_posts` 내 컬럼 추가가 단순하다.

---

## 4.2 검증 체크리스트 버전 추적

### 현재 ERD 상태

현재 ERD에는 `ai_prompt_versions`가 존재한다. AI 생성 지시 버전은 추적할 수 있다.

그러나 검증 체크리스트 버전을 별도로 관리하는 테이블은 없다. `ai_validation_logs.checklist_json`에 검증 결과는 저장되지만, 적용된 체크리스트의 버전 자체를 안정적으로 추적하기에는 부족하다.

### 요구사항 변화

변경된 요구사항은 모든 AI 산출물에 대해 다음 정보를 추적하도록 요구한다.

- 생성 지시 자산 버전
- 검증 체크리스트 버전
- 산출물 생성 시각
- 검증 상태

### 문제점

체크리스트 내용이 바뀌었을 때 과거 산출물이 어떤 기준으로 검증되었는지 확인하기 어렵다.

### 변경 필요 판단

변경 필요: 높음

### 권장 변경안

신규 테이블 `ai_validation_checklist_versions`를 추가한다.

| 컬럼 | 타입 | 설명 |
|---|---|---|
| id | BIGINT | 체크리스트 버전 ID |
| checklist_type | VARCHAR(30) | EXPLANATION, SIMULATOR, QA |
| version | VARCHAR(30) | 예: 2026.05.17-1 |
| content_hash | VARCHAR(100) | 체크리스트 원문 해시 |
| status | VARCHAR(20) | ACTIVE, RETIRED |
| created_by_admin_id | BIGINT | 생성 관리자 |
| created_at | DATETIME(6) | 생성 시각 |

그리고 `ai_validation_logs`에 다음 컬럼을 추가한다.

| 컬럼 | 타입 | 설명 |
|---|---|---|
| checklist_version_id | BIGINT | ai_validation_checklist_versions.id |

---

## 4.3 AI 평가 셋 및 실패 사례 환류

### 현재 ERD 상태

현재 ERD에는 AI 생성 작업, 산출물, 검증 로그는 존재한다.

- `ai_generation_jobs`
- `ai_generated_assets`
- `ai_validation_logs`

그러나 평가 셋 또는 평가 케이스를 관리하는 구조는 없다.

### 요구사항 변화

변경된 요구사항에는 다음 내용이 포함된다.

- 평가 셋 최소 기준을 10개 본문으로 상향
- 사용자 신고로 접수된 AI 응답을 평가 셋 개선에 활용
- 검증 실패 사례를 평가 셋 후보로 환류
- 운영자가 평가 셋 반영 여부를 판단

### 문제점

현재 구조로는 검증 실패나 신고 사례를 단순 로그로 남길 수는 있지만, 이를 평가 셋 후보로 등록하고 확정/반려하는 흐름을 데이터로 관리하기 어렵다.

### 변경 필요 판단

변경 필요: 중간 이상

### 권장 변경안

신규 테이블을 추가한다.

#### ai_evaluation_sets

| 컬럼 | 타입 | 설명 |
|---|---|---|
| id | BIGINT | 평가 셋 ID |
| name | VARCHAR(100) | 평가 셋 이름 |
| eval_type | VARCHAR(30) | EXPLANATION, SIMULATOR, QA |
| version | VARCHAR(30) | 평가 셋 버전 |
| status | VARCHAR(20) | DRAFT, ACTIVE, RETIRED |
| created_at | DATETIME(6) | 생성 시각 |

#### ai_evaluation_cases

| 컬럼 | 타입 | 설명 |
|---|---|---|
| id | BIGINT | 평가 케이스 ID |
| evaluation_set_id | BIGINT | ai_evaluation_sets.id |
| target_type | VARCHAR(30) | BIBLE_VERSE, QT_PASSAGE, QA_REQUEST |
| target_id | BIGINT | 대상 ID |
| input_json | JSON | 평가 입력 |
| expected_policy_json | JSON | 기대 검증 기준 |
| status | VARCHAR(20) | CANDIDATE, APPROVED, REJECTED |
| source_type | VARCHAR(30) | VALIDATION_FAILURE, USER_REPORT, ADMIN_CREATED |
| source_id | BIGINT | 원천 로그 또는 신고 ID |
| reviewed_by_admin_id | BIGINT | 검토 관리자 |
| created_at | DATETIME(6) | 생성 시각 |

---

## 4.4 AI Q&A 신고 및 운영 검토

### 현재 ERD 상태

현재 `reports`는 `target_type`, `target_id` 구조를 가진다. 문서상 대상은 `POST`, `COMMENT` 중심으로 설명되어 있다.

### 요구사항 변화

변경된 요구사항에서는 사용자 신고로 접수된 AI 응답을 운영자가 확인할 수 있어야 한다.

### 문제점

현재 `reports.target_type`에 AI Q&A 요청이나 AI 응답 산출물을 신고 대상으로 포함하는 정책이 명확하지 않다.

### 변경 필요 판단

변경 필요: 중간

### 권장 변경안

`reports.target_type` 허용 값을 확장한다.

기존:

- POST
- COMMENT

추가:

- AI_QA_REQUEST
- AI_ASSET

또한 `reports.reason`에 AI 응답 전용 신고 사유 코드를 추가한다.

예:

- FACT_ERROR
- UNSAFE_ADVICE
- UNSOURCED_ANSWER
- POLICY_VIOLATION

---

## 4.5 역할 및 권한 분리

### 현재 ERD 상태

현재 `admin_users`는 다음 역할을 가진다.

- SUPER_ADMIN
- OPERATOR
- REVIEWER

### 요구사항 변화

변경된 요구사항은 다음 주체를 분리한다.

- 일반 사용자
- 관리자
- 자체 제작자
- 시스템 배치

특히 검증용 한국어 주석 자료는 일반 관리자에게도 노출되면 안 되고, 자체 제작자와 필요한 시스템 배치 경로에서만 접근 가능해야 한다.

### 문제점

현재 `admin_users.admin_role`만으로는 자체 제작자와 시스템 배치를 명확히 표현하기 어렵다. 시스템 배치는 회원 계정과 직접 연결되지 않는 비인간 주체일 가능성이 높다.

### 변경 필요 판단

변경 필요: 높음

### 권장 변경안

MVP에서는 `admin_role` 값을 확장하는 방식으로 시작할 수 있다.

추가 후보:

- CONTENT_CREATOR
- SYSTEM_BATCH

다만 장기적으로는 별도 구조를 권장한다.

#### service_accounts

| 컬럼 | 타입 | 설명 |
|---|---|---|
| id | BIGINT | 시스템 계정 ID |
| account_name | VARCHAR(100) | 배치 또는 시스템 주체명 |
| account_type | VARCHAR(30) | BATCH, VALIDATION_AGENT |
| status | VARCHAR(20) | ACTIVE, DISABLED |
| created_at | DATETIME(6) | 생성 시각 |

그리고 감사 로그에서 관리자뿐 아니라 시스템 계정도 수행 주체로 기록할 수 있게 해야 한다.

---

## 4.6 감사 로그 수행 주체 확장

### 현재 ERD 상태

현재 `audit_logs`는 `admin_user_id` 중심이다.

### 요구사항 변화

감사 로그에는 다음 이벤트가 포함되어야 한다.

- AI Q&A 차단 사례
- 자동 검증 실패 사례
- 평가 셋 반영
- 검증 체크리스트 버전 변경
- 컴포넌트 라이브러리 버전 변경

이 중 일부는 관리자가 아니라 시스템 배치 또는 검증 agent가 수행한다.

### 문제점

`admin_user_id`만으로는 시스템 배치가 수행한 작업을 표현하기 어렵다.

### 변경 필요 판단

변경 필요: 높음

### 권장 변경안

`audit_logs`에 수행 주체 컬럼을 추가한다.

| 컬럼 | 타입 | 설명 |
|---|---|---|
| actor_type | VARCHAR(30) | ADMIN, SYSTEM_BATCH, CONTENT_CREATOR |
| actor_id | BIGINT | admin_users.id 또는 service_accounts.id |
| actor_label | VARCHAR(100) | 표시용 수행 주체명 |

기존 `admin_user_id`는 유지하되, 점진적으로 `actor_type + actor_id` 기반으로 확장할 수 있다.

---

## 4.7 시뮬레이터 컴포넌트 라이브러리 버전 추적

### 현재 ERD 상태

현재 `simulator_clips`는 `scene_script_json`, `status`, `ai_asset_id`를 가진다.

### 요구사항 변화

시뮬레이터 요구사항에는 다음 내용이 포함된다.

- AI가 자유 형식 코드, 외부 자산, 검증되지 않은 이미지를 직접 생성해 노출하면 안 됨
- 시뮬레이터는 정해진 컴포넌트 라이브러리를 기반으로 구성
- 컴포넌트 라이브러리는 버전으로 관리

### 문제점

현재 구조만으로는 특정 시뮬레이터 클립이 어떤 컴포넌트 라이브러리 버전으로 만들어졌는지 추적하기 어렵다.

### 변경 필요 판단

변경 필요: 중간

### 권장 변경안

신규 테이블 `simulator_component_library_versions`를 추가한다.

| 컬럼 | 타입 | 설명 |
|---|---|---|
| id | BIGINT | 라이브러리 버전 ID |
| version | VARCHAR(30) | 컴포넌트 라이브러리 버전 |
| content_hash | VARCHAR(100) | 라이브러리 정의 해시 |
| status | VARCHAR(20) | ACTIVE, RETIRED |
| created_at | DATETIME(6) | 생성 시각 |

그리고 `simulator_clips` 또는 `ai_generated_assets`에 다음 컬럼을 추가한다.

| 컬럼 | 타입 | 설명 |
|---|---|---|
| component_library_version_id | BIGINT | simulator_component_library_versions.id |

---

## 4.8 AI Q&A 사용량 제한

### 현재 ERD 상태

`ai_qa_requests`는 사용자별 질문 이력을 저장한다.

### 요구사항 변화

요구사항에는 사용자별 사용량 제한이 적용되어야 한다고 되어 있다.

### 문제점

단순한 일별 제한은 `ai_qa_requests.member_id + created_at` 집계로 처리 가능하다. 따라서 MVP에서는 별도 테이블이 없어도 된다.

다만 요금제, 정책 변경, 운영자 수동 조정 등이 생기면 별도 제한 정책 테이블이 필요하다.

### 변경 필요 판단

MVP 기준 변경 불필요. 확장 시 검토.

### 확장 후보

- `usage_limit_policies`
- `member_usage_daily_counters`

---

## 5. 우선순위별 변경 권장안

### P0: 반드시 반영 권장

| 항목 | 이유 |
|---|---|
| 공유본 스냅샷 컬럼 추가 | 요구사항의 변경 불가 공유본을 직접 만족해야 함 |
| 검증 체크리스트 버전 테이블 추가 | AI 산출물 추적성 핵심 요구사항 |
| 감사 로그 actor 구조 확장 | 시스템 배치/자체 제작자 작업 기록 필요 |
| 역할 구조 보강 | 검증용 자료 접근 제한 요구사항 충족 필요 |

### P1: 구현 전 설계 반영 권장

| 항목 | 이유 |
|---|---|
| 평가 셋/평가 케이스 테이블 추가 | 검증 실패 및 신고 사례 환류를 데이터로 관리 |
| AI Q&A 신고 대상 확장 | AI 응답 운영 검토 요구사항 충족 |
| 시뮬레이터 컴포넌트 라이브러리 버전 테이블 추가 | 시뮬레이터 검증 가능성 확보 |

### P2: 후속 확장 검토

| 항목 | 이유 |
|---|---|
| 사용량 제한 정책 테이블 | MVP는 요청 로그 집계로 가능 |
| 세분화된 RBAC 테이블 | MVP는 enum 확장으로 시작 가능 |
| 검색 엔진/전문 검색 인덱스 | 현재 요구사항은 좌표 기반 조회 중심 |

---

## 6. 기존 ERD 대비 변경 후보 요약

| 구분 | 기존 테이블 | 변경 유형 | 변경 후보 |
|---|---|---|---|
| 공유 | sharing_posts | 컬럼 추가 | 노트 본문, 제목, 구절, 카테고리 스냅샷 |
| AI 검증 | 신규 | 테이블 추가 | ai_validation_checklist_versions |
| AI 검증 로그 | ai_validation_logs | 컬럼 추가 | checklist_version_id |
| AI 평가 | 신규 | 테이블 추가 | ai_evaluation_sets, ai_evaluation_cases |
| 신고 | reports | 코드 확장 | target_type에 AI_QA_REQUEST, AI_ASSET 추가 |
| 권한 | admin_users | 역할 확장 | CONTENT_CREATOR, SYSTEM_BATCH 등 |
| 시스템 주체 | 신규 | 테이블 추가 | service_accounts |
| 감사 로그 | audit_logs | 컬럼 추가 | actor_type, actor_id, actor_label |
| 시뮬레이터 | 신규 | 테이블 추가 | simulator_component_library_versions |
| 시뮬레이터 | simulator_clips 또는 ai_generated_assets | 컬럼 추가 | component_library_version_id |

---

## 7. 최종 의견

기존 ERD는 성경 본문, QT 본문, 주석 원문, 절 매핑, 노트, AI 생성 작업의 기본 골격을 잘 잡고 있다. 특히 현재 보유한 `성경` 폴더와 `주석` 폴더의 데이터를 적재하기 위한 구조는 유지해도 된다.

다만 요구사항 v3.1에서 운영, 검증, 감사, 공유 스냅샷 요구가 강화되었기 때문에 기존 ERD를 그대로 사용하면 일부 요구사항은 애플리케이션 로직이나 JSON 컬럼에 과도하게 의존하게 된다.

따라서 기존 ERD를 유지하되, 다음 4가지는 우선 반영하는 것을 권장한다.

1. `sharing_posts`에 공유 시점 스냅샷 저장 구조 추가
2. `ai_validation_checklist_versions` 추가 및 `ai_validation_logs`와 연결
3. `audit_logs`의 수행 주체를 관리자 외 시스템 배치까지 확장
4. 자체 제작자와 시스템 배치 권한을 표현할 수 있도록 역할 구조 보강

이 보강을 적용하면 기존 ERD의 방향성을 유지하면서도 변경된 요구사항을 훨씬 안정적으로 수용할 수 있다.
