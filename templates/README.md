# 📦 프로젝트 문서 템플릿 팩 v2

> [Tae0072/team-project-templates](https://github.com/Tae0072/team-project-templates) — [Tae0072/hms](https://github.com/Tae0072/hms)(1차 프로젝트, HMS) 문서 체계를 **도메인 중립 템플릿**으로 재구성한 세트입니다.
>
> **v2 보강 포인트** — 2차 팀 프로젝트(6명, AI 에이전트 활용, 3주 압축 개발, 포트폴리오·시연 목적)에 맞춰 다음을 추가/갱신했습니다.
> - **AI 에이전트 협업 가이드** + **AGENTS.md 예시** (Claude Code, Cursor, Codex 등 보편 표준)
> - **포트폴리오용 README** / **시연 준비 가이드** / **CI/CD 파이프라인** / **회고**
> - 누락되어 있던 **08 스토리보드 / 13 테스트 보고서 / 15 사용자 매뉴얼 / 16 운영 매뉴얼** 템플릿 신규 작성
> - **00 일정 / 01 계획서**를 6명·3주·AI 활용·포트폴리오 가정에 맞게 갱신
> - **2차 프로젝트 부트스트랩 문서** (`BOOTSTRAP_2차프로젝트.md`) 신규 — 5/6 주제 선정 ~ 5/11 개발 착수 사이 운영 매뉴얼

---

## 빠른 시작 — 2차 프로젝트 팀이라면 여기서부터

> ⚡ **5월 6일 주제 선정 / 5월 11일 개발 착수**라면 이 순서대로:
>
> 1. [`BOOTSTRAP_2차프로젝트.md`](./BOOTSTRAP_2차프로젝트.md)를 통독 (15분)
> 2. [`프로젝트_문서_가이드.md`](./프로젝트_문서_가이드.md)로 전체 문서 체계 파악 (20분)
> 3. [`templates/17_AI_에이전트_협업_가이드_template.md`](./templates/17_AI_에이전트_협업_가이드_template.md) 합의 + [`templates/AGENTS.md.example`](./templates/AGENTS.md.example)을 프로젝트 루트의 `AGENTS.md`로 복사 (Day 0)
> 4. 01·02·07 작성 → 03·09·10·12 픽스 → 개발 시작

---

## 구성

### 메타 문서

| 파일 | 용도 |
| --- | --- |
| [`BOOTSTRAP_2차프로젝트.md`](./BOOTSTRAP_2차프로젝트.md) | **2차 프로젝트 전용** — 6명·3주·AI 에이전트·포트폴리오 가정에서의 W0~W4 운영 매뉴얼 |
| [`프로젝트_문서_가이드.md`](./프로젝트_문서_가이드.md) | 전체 문서 체계 / 작성 순서 / 컨벤션 / 체크리스트 |

### 단계별 템플릿

| 번호 | 파일 | 용도 | 작성 시점 |
| :---: | --- | --- | :---: |
| 00 | [`00_개발_일정_총괄표_template.md`](./templates/00_개발_일정_총괄표_template.md) | Gantt + 마일스톤 + Critical Path (**v2: 6명·3주 압축**) | W0 |
| 01 | [`01_프로젝트_계획서_template.md`](./templates/01_프로젝트_계획서_template.md) | SMART 목표·WBS·리스크 (**v2: 포트폴리오 차별화 / AI 활용 계획**) | W0 |
| 02 | [`02_ERD_문서_template.md`](./templates/02_ERD_문서_template.md) | 테이블·관계·상태코드 | W0~W1 |
| 03 | [`03_아키텍처_정의서_template.md`](./templates/03_아키텍처_정의서_template.md) | 패키지·소유자·공유 레이어 | W0~W1 |
| 04 | [`04_API_명세서_template.md`](./templates/04_API_명세서_template.md) | 공통 규칙·엔드포인트 템플릿 | W1 |
| 05 | [`05_시퀀스_다이어그램_template.md`](./templates/05_시퀀스_다이어그램_template.md) | Mermaid 시퀀스 | W1 |
| 06 | [`06_화면_기능_정의서_template.md`](./templates/06_화면_기능_정의서_template.md) | Layout·Model·화면 상세 | W1 |
| 07 | [`07_요구사항_정의서_template.md`](./templates/07_요구사항_정의서_template.md) | 교과/과제 필수 요건 체크리스트 | W0 |
| **08** | [**`08_스토리보드_template.md`**](./templates/08_스토리보드_template.md) | **🆕 사용자 시나리오 흐름·와이어프레임·시연 강조 화면** | W1 |
| 09 | [`09_Git_규칙_template.md`](./templates/09_Git_규칙_template.md) | 브랜치·커밋·PR | W0 |
| 10 | [`10_환경_설정_템플릿/`](./templates/10_환경_설정_템플릿/) | application 공통/dev/prod 3종 | W0 |
| 11 | [`11_테스트_전략서_template.md`](./templates/11_테스트_전략서_template.md) | 커버리지·담당자·CI | W1 |
| 12 | [`12_코드_리뷰_규칙_template.md`](./templates/12_코드_리뷰_규칙_template.md) | PR·리뷰 체크리스트 | W0 |
| **13** | [**`13_테스트_보고서_template.md`**](./templates/13_테스트_보고서_template.md) | **🆕 시연 직전 품질 스냅샷 / 결함 목록 / 시연 가능 여부 판정** | W4 초 |
| 14 | [`14_배포_가이드_template.md`](./templates/14_배포_가이드_template.md) | Docker Compose·Nginx·SSL | W2 |
| **15** | [**`15_사용자_메뉴얼_template.md`**](./templates/15_사용자_메뉴얼_template.md) | **🆕 5분 빠른 시작 + 기능별 사용법 + FAQ** | W3~W4 |
| **16** | [**`16_운영_메뉴얼_template.md`**](./templates/16_운영_메뉴얼_template.md) | **🆕 일상 운영·장애 대응·발표 주 특별 운영 / 비상 연락망** | W3~W4 |

### v2 신규 템플릿 (AI·포트폴리오·운영)

| 번호 | 파일 | 용도 | 작성 시점 |
| :---: | --- | --- | :---: |
| **17** | [`17_AI_에이전트_협업_가이드_template.md`](./templates/17_AI_에이전트_협업_가이드_template.md) | **AI 에이전트 사용 규칙 / CTPV 프롬프트 / 6단계 검증 / 환각 체크리스트** | W0 |
| - | [`AGENTS.md.example`](./templates/AGENTS.md.example) | **프로젝트 루트에 둘 `AGENTS.md` 실제 예시 (Claude Code·Cursor·Codex 보편 표준)** | W0 |
| **18** | [`18_포트폴리오_README_template.md`](./templates/18_포트폴리오_README_template.md) | **현직 개발자 평가용 README — 30초 가치 전달 / 기술적 도전 / 성능 지표** | W3~W4 |
| **19** | [`19_시연_준비_가이드_template.md`](./templates/19_시연_준비_가이드_template.md) | **15분 시연 대본 / Q&A 30선 / 라이브 데모 안전장치 / 발표자 역할 분담** | W4 |
| **20** | [`20_CICD_파이프라인_template.md`](./templates/20_CICD_파이프라인_template.md) | **GitHub Actions ci/stage/prod 전체 코드 / 단계적 도입 일정** | W0~W2 |
| **21** | [`21_프로젝트_회고_template.md`](./templates/21_프로젝트_회고_template.md) | **KPT / 5 Whys / AI 협업 정량 회고 / ADR / 기술 블로그 후속 계획** | W4 |

---

## 치환 가이드 (Find & Replace)

템플릿에 적힌 `{{...}}` 자리표시자를 일괄 치환하세요.

| 토큰 | 치환 예 |
| --- | --- |
| `{{프로젝트명}}` | 우리팀_서비스명 |
| `{{EMOJI}}` | 🚀 |
| `{{DOMAIN}}` | my-domain |
| `{{DURATION}}` | 4주 (개발 3주 + 발표 1주) |
| `{{N}}` | 6 (팀원 수) |
| `{{PROJECT_ENV}}` | MYAPP (환경변수 prefix) |
| `{{project}}` | myapp (패키지·DB명 등 소문자) |
| `{{ENTITY_1}}`, `{{ENTITY_2}}` | USER, ORDER ... |
| `{{ROLE_A}}`, `{{ROLE_B}}`, `{{ROLE_C}}` | USER, ADMIN, MANAGER ... |
| `{{MAIN_FEATURE_1}}` | 핵심 기능 1 |
| `{{EXTERNAL}}` | CLAUDE / KAKAO / TOSS 등 |
| `{{Lead}}`, `{{DevA}}~{{DevD}}`, `{{DevOps}}` | 6명 팀의 역할 (v2 신규) |

**빠른 치환 스크립트 (bash)**

```bash
find templates -type f -name "*.md" -print0 | xargs -0 sed -i \
  -e 's/{{프로젝트명}}/내_프로젝트/g' \
  -e 's/{{EMOJI}}/🚀/g' \
  -e 's/{{project}}/myproject/g' \
  -e 's/{{DURATION}}/4주/g' \
  -e 's/{{N}}/6/g'
```

---

## 사용 순서 (권장)

### 2차 프로젝트 (6명·3주·AI·포트폴리오) 기준

1. **W0 킥오프 (5/6~5/10)** — `BOOTSTRAP_2차프로젝트.md` 통독, 가이드 훑기
2. **핵심 문서 4종** — 01 계획서 / 02 ERD / 07 요구사항 / **17 AI 가이드 + AGENTS.md**
3. **기반 규칙 픽스** — 03 아키텍처 / 09 Git / 10 환경 / 12 코드리뷰 / **20 CI/CD 최소 파이프라인**
4. **개발 직전** — 04 API / 05 시퀀스 / 06 화면 / **08 스토리보드** / 11 테스트
5. **개발 진행 (W1~W3, 5/11~5/29)** — 일일 stand-up + 격일 회고 + AI 에이전트 활용 로그
6. **발표 주 (W4)** — **13 테스트 보고서** / 14 배포 / **15 사용자 / 16 운영 / 18 README / 19 시연 / 21 회고**

> 일정 가정의 자세한 내용은 [`BOOTSTRAP_2차프로젝트.md`](./BOOTSTRAP_2차프로젝트.md) 참조.

---

## v1 → v2 변경 요약

| 영역 | v1 (1차, HMS) | v2 (2차, 본 저장소 갱신) |
|------|--------------|---------------------------|
| 팀 규모 | 3명 | 6명 (Lead·DevA~D·DevOps/QA) |
| 기간 | 4주 풀개발 | 4주 (3주 개발 + 1주 발표) |
| AI 활용 | 부분 | 정식 워크플로우 (17 + AGENTS.md) |
| 산출물 목적 | 수업 평가 | 포트폴리오 + 현직 개발자 시연 |
| CI/CD | 수동 배포 | GitHub Actions 3-tier (ci/stage/prod) |
| 신규 템플릿 | - | 08, 13, 15, 16, 17, 18, 19, 20, 21 + AGENTS.md.example |

---

## 원본 / 참고

- 1차 프로젝트: https://github.com/Tae0072/hms
- 본 저장소: https://github.com/Tae0072/team-project-templates
- 외부 참고: [othneildrew/Best-README-Template](https://github.com/othneildrew/Best-README-Template), AGENTS.md 표준 (agentsmd.net), Anthropic Claude Code Best Practices
