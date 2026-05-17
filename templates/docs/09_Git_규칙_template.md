# 팀 프로젝트 Git & GitHub 사용 규칙

> **문서 버전:** v1.0
> **작성일:** YYYY-MM-DD

---

## 1. 기본 원칙 (반드시 지켜주세요)
- `main`은 **배포 전용** — 직접 push 금지, PR merge만
- `develop`이 **개발 중심** — 모든 feature/bugfix는 여기서 분기·병합
- 기능 하나 = 브랜치 하나 = PR 하나
- Merge 전략: **Squash merge** 기본
- Force push는 본인 feature 브랜치에서만 허용

---

## 2. 저장소 처음 가져오기 (최초 Clone)

```bash
git clone https://github.com/{{org}}/{{repo}}.git
cd {{repo}}

# 기본 브랜치를 develop으로 설정해두면 편함
git switch develop
git branch --set-upstream-to=origin/develop
```

---

## 3. 일상적인 개발 워크플로우 (매일 반복)

### Step 0. 작업 시작 전 — develop 최신화 (필수!)
```bash
git switch develop
git pull --rebase origin develop
```

### Step 1. 새 작업 브랜치 생성
```bash
git switch -c feature/123-{{short-desc}}
# 또는 bugfix/128-{{short-desc}} / hotfix/...
```

### Step 2. 작업 → 커밋 → 푸시
```bash
git add .
git commit -m "feat({{scope}}): {{요약}}"
git push -u origin feature/123-{{short-desc}}
```

### Step 3. Pull Request 생성 & 리뷰
- 템플릿에 맞춰 본문 작성
- 리뷰어 2인 지정 (책임개발자 포함 권장)
- CI 통과 확인

### Step 4. develop에 다른 변경이 merge된 후 — 본인 브랜치 업데이트
```bash
git switch feature/123-{{short-desc}}
git fetch origin
git rebase origin/develop
# 충돌 발생 시 → 해결 → git add → git rebase --continue
git push --force-with-lease
```

### Step 5. PR 업데이트 & 재리뷰 → 다시 Squash merge

---

## 4. 브랜치 종류 & 수명 관리

| 브랜치 | 목적 | 수명 | 배포 |
| --- | --- | --- | --- |
| `main` | 배포 스냅샷 | 영구 | ✅ |
| `develop` | 통합 개발 | 영구 | ❌ |
| `feature/{번호}-{설명}` | 신규 기능 | 짧게 (3~5일) | ❌ |
| `bugfix/{번호}-{설명}` | 버그 수정 | 짧게 | ❌ |
| `hotfix/{설명}` | 운영 긴급 수정 | 즉시 | ✅ (main 직접 merge 허용) |
| `release/{버전}` | 배포 준비 | 단기 | ❌ |

---

## 5. Commit Message 규칙 (Conventional Commits 기반)

```
{type}({scope}): {subject}

{body — 무엇을 왜 바꿨는지}

{footer — 관련 이슈, BREAKING CHANGE}
```

**Type**

| type | 설명 |
| --- | --- |
| feat | 신규 기능 |
| fix | 버그 수정 |
| refactor | 기능 변경 없는 구조 개선 |
| test | 테스트 추가/수정 |
| docs | 문서 변경 |
| chore | 빌드/패키지 등 잡일 |
| ci | CI 설정 |
| style | 포맷/세미콜론 등 코드 스타일 |

**팀원/팀장 구분 prefix**
- `[lead] feat(security): ...` — 책임개발자
- `[devA] feat({{moduleA}}): ...` — 개발자 A

**예시**
```
feat(reservation): 전화 예약 등록 흐름 추가

- 전화 예약 전용 Controller/Service
- 중복 슬롯 검증 로직 추가
- 접수 직원 화면에서 진입

refs #123
```

---

## 6. Pull Request 규칙 & 템플릿

**PR 템플릿 (`.github/pull_request_template.md`)**

```markdown
## 관련 이슈
- closes #123

## 변경 사항 요약
- ...

## 스크린샷 / GIF (UI/UX 변경 시 필수)

## 리뷰 포인트
- 특별히 봐줬으면 하는 부분

## 체크리스트
- [ ] 로컬에서 동작 확인
- [ ] 신규 테스트 추가
- [ ] 문서 업데이트
- [ ] API 명세서 동기화
- [ ] CSRF / 인가 체크
```

**PR 크기:** 400줄 이하 권장, 1000줄 초과 시 분할 요청.

---

## 7. 추가 안전장치 & 팀 문화

- **Branch Protection (main / develop)**
  - Require PR + 2 approving reviews
  - Require status checks (CI)
  - Include administrators
- **GitHub Issues 필수 라벨:** `bug`, `feature`, `docs`, `discussion`, `priority/high`
- **PR 제목 prefix** — 커밋 타입과 동일 (예: `feat: ...`)
- **.gitignore 필수 포함**
  - `application-*.properties` (예제 `.example`만 커밋)
  - `.env`, `*.pem`, `*.key`
  - `build/`, `out/`, `.gradle/`

---


---

## 8. AI 에이전트 보조 커밋 & PR 가이드라인

### 8.1 AI 보조 커밋 표시 원칙

- **커밋 메시지에 AI 도구 명시 금지** — 작성자가 코드의 품질·정확성에 책임진다.
- AI로 생성한 코드도 사람이 검토·수정했으면 일반 커밋으로 작성.
- 예외: 1000줄 이상 대규모 AI 생성 PR은 PR 본문에 1줄 명시.

### 8.2 AI 보조 PR 라벨

GitHub Labels에 아래 라벨을 사전 등록한다:

| 라벨 | 조건 | 추가 리뷰 |
| --- | --- | --- |
| `ai-generated` | 코드의 50%+ AI 생성 | 환각 체크리스트(17번 문서) 적용 |
| `ai-assisted` | 일부 AI 보조 (자동완성 수준) | 일반 리뷰 |

### 8.3 AI 생성 코드 PR 크기 관리

AI가 한 번에 1500줄을 만들더라도 **PR은 400줄 이하로 분할**:

```
분할 기준:
- Entity + Repository → 1 PR
- Service + 단위 테스트 → 1 PR  
- Controller + Mustache → 1 PR
```

### 8.4 환각(Hallucination) 의심 커밋 처리

```
커밋 전 체크:
[ ] ./gradlew build 통과
[ ] ./gradlew test 통과
[ ] import된 클래스가 실제 존재 (IDE 빨간 줄 없음)
[ ] 호출한 메서드 시그니처가 라이브러리 공식 문서와 일치
[ ] 환경변수 하드코딩 없음
```

---

## 9. 태그 & 릴리스 전략

```bash
# 스테이징 릴리스 태그 (W3 배포 시)
git tag -a v1.0.0-staging -m "스테이징 배포 - W3 통합 테스트 완료"
git push origin v1.0.0-staging

# 운영 릴리스 태그 (W3 목 운영 배포 시)
git tag -a v1.0.0 -m "v1.0.0 운영 배포 - 시연 버전"
git push origin v1.0.0

# 태그 목록 확인
git tag -l
```

**GitHub Release** 생성 (운영 배포 시):
- Tag: `v1.0.0`
- Title: `v1.0.0 - {{PROJECT_NAME}} 시연 버전`
- Body: 주요 기능 목록 + 알려진 이슈

## 📌 변경 이력

| 버전 | 날짜 | 작성자 | 주요 변경 |
| --- | --- | --- | --- |
| v1.0 | YYYY-MM-DD | Lead | 초기 작성 |
