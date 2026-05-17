<!--
{{PROJECT_NAME}} 포트폴리오 README 템플릿
사용법:
  1. {{...}} 자리표시자 모두 치환
  2. 스크린샷·GIF는 docs/assets/에 보관
  3. 배지(badge)는 실제 CI 설정 후 활성화
  4. "팀원" 섹션은 각자 GitHub 프로필 + 1줄 기여
포트폴리오 README는 채용 담당자가 30초 안에 "더 볼지 말지" 판단하는 페이지다.
스크롤 안 하고도 핵심이 보여야 한다.
-->

<div align="center">

# {{EMOJI}} {{PROJECT_NAME}}

**{{한 줄 가치 제안 — 이 프로젝트가 어떤 문제를 어떻게 해결하는가}}**

[![Build Status](https://github.com/{{org}}/{{repo}}/actions/workflows/ci.yml/badge.svg)](https://github.com/{{org}}/{{repo}}/actions/workflows/ci.yml)
[![Coverage](https://img.shields.io/badge/coverage-{{N}}%25-brightgreen)](https://github.com/{{org}}/{{repo}})
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-6DB33F?logo=spring-boot)
![JDK](https://img.shields.io/badge/JDK-21-orange?logo=openjdk)
![MySQL](https://img.shields.io/badge/MySQL-8-4479A1?logo=mysql)

[🔗 라이브 데모](https://example.com) ·
[🎬 시연 영상](https://youtu.be/...) ·
[📝 기술 블로그](https://www.notion.so/...) ·
[🐛 이슈 제보](https://github.com/{{org}}/{{repo}}/issues)

</div>

---

## 📖 목차

1. [한눈에 보기](#-한눈에-보기)
2. [주요 기능](#-주요-기능)
3. [기술 스택](#%EF%B8%8F-기술-스택)
4. [시스템 아키텍처](#%EF%B8%8F-시스템-아키텍처)
5. [기술적 도전과 해결](#-기술적-도전과-해결)
6. [성능·품질 지표](#-성능품질-지표)
7. [빠른 시작](#-빠른-시작)
8. [프로젝트 구조](#-프로젝트-구조)
9. [팀](#-팀)
10. [회고 & 학습](#-회고--학습)

---

## 🔍 한눈에 보기

| 항목 | 내용 |
| --- | --- |
| **개발 기간** | {{YYYY-MM-DD}} ~ {{YYYY-MM-DD}} ({{N}}주) |
| **팀 구성** | {{N}}명 (책임개발자 1명 + 개발자 {{N-1}}명, AI 에이전트 보조) |
| **개발 방식** | 애자일 — 주차별 마일스톤, 데일리 스탠드업, AI 페어 프로그래밍 |
| **내 역할** | {{본인 담당 영역 한 줄}} |
| **핵심 성과** | {{정량 지표 — 예: API 응답 P95 350ms, 테스트 커버리지 82%, 동시 사용자 1000명}} |

---

## ✨ 주요 기능

> 스크린샷·GIF는 docs/assets에 저장. 화면이 곧 가치 — 글보다 이미지 먼저.

### {{MAIN_FEATURE_1}}

![{{MAIN_FEATURE_1}}](docs/assets/feature-1.gif)

- {{핵심 흐름 한 줄}}
- {{기술 포인트 — 예: LLM API 통합, 비동기 처리}}

### {{MAIN_FEATURE_2}}

![{{MAIN_FEATURE_2}}](docs/assets/feature-2.gif)

- {{핵심 흐름 한 줄}}

### 추가 기능

- ✅ {{역할 기반 인증/인가}}
- ✅ {{관리자 통계 대시보드}}
- ✅ {{외부 API 연동 + 폴백}}
- ✅ {{반응형 UI / 다크모드}}

---

## 🛠️ 기술 스택

<table>
<tr>
<td valign="top" width="50%">

### Backend
- **Java 21** + **Spring Boot 3.x**
- **Spring Security** (세션·CSRF)
- **Spring Data JPA** + **MySQL 8**
- **Redis 7** (세션 저장소·캐시)
- **Mustache** (SSR) / **Jackson** (JSON API)

### Frontend
- **HTML5 + CSS3 + JavaScript**
- **Bootstrap 5** / **Tailwind CSS**
- **Chart.js** (관리자 통계)

</td>
<td valign="top" width="50%">

### DevOps & 협업
- **Docker** + **Docker Compose**
- **Nginx** (리버스 프록시 + SSL)
- **GitHub Actions** (CI: 빌드·테스트·린트)
- **GitHub Projects** (이슈·칸반)

### Quality
- **JUnit 5** + **Mockito**
- **JaCoCo** (커버리지 리포트)
- **Checkstyle** + **SpotBugs**
- **Claude Code** / **Cursor** (AI 페어)

</td>
</tr>
</table>

---

## 🏗️ 시스템 아키텍처

```mermaid
flowchart LR
    User[사용자] --> Nginx
    Nginx --> Spring[Spring Boot]
    Spring --> MySQL[(MySQL)]
    Spring --> Redis[(Redis)]
    Spring --> External[{{EXTERNAL}} API]

    subgraph CI/CD
        GH[GitHub] --> Actions[GH Actions]
        Actions -->|build·test| Build[Docker Image]
    end
```

**설계 핵심**

- **레이어드 아키텍처** — Controller / Service / Repository 분리, 비즈니스 로직은 Service에 응집
- **세션은 Redis로 외부화** → 서버 무중단 재시작 가능
- **외부 API는 폴백 경로** → 장애 시에도 핵심 기능 유지
- **공유 레이어 단일 책임** → 동시 작업 시 충돌 최소화 ([상세](docs/03_아키텍처_정의서.md))

---

## 💡 기술적 도전과 해결

> 이 섹션이 포트폴리오의 핵심이다. **문제 → 시도 → 해결 → 결과(숫자)** 순서로.

### 🔥 도전 1: {{문제 — 예: 동시 예약 시 슬롯 중복 발생}}

- **상황:** {{언제·어떻게 발견}}
- **원인 분석:** {{루트 코즈}}
- **시도한 접근:**
  1. {{첫 번째 시도와 한계}}
  2. {{두 번째 시도와 한계}}
- **최종 해결:** {{선택한 방법}} (예: DB 유니크 제약 + 비관적 락)
- **결과:** {{숫자로 — 예: 동시 100건 테스트 시 중복 0건}}
- **학습:** {{인사이트}}
- **상세 글:** [노션 기술 블로그](https://www.notion.so/...)

### 🔥 도전 2: {{문제}}

(반복)

### 🔥 도전 3: AI 에이전트 협업 워크플로우 정착

- **상황:** 6명 팀 + AI 에이전트로 3주 안에 완성해야 함
- **시도:** AGENTS.md 표준화, 환각 체크리스트, slash command 등록
- **결과:** {{개발 속도 X% 향상 / 코드 리뷰 시간 Y% 감소}}
- **상세 글:** [노션 기술 블로그](https://www.notion.so/...)

---

## 📊 성능·품질 지표

| 지표 | 결과 | 목표 |
| --- | --- | --- |
| API 응답 시간 (P95) | {{X}}ms | 500ms 이하 ✅ |
| 동시 사용자 부하 테스트 | {{N}} TPS | {{목표}} TPS |
| 테스트 커버리지 (Service) | {{X}}% | 80%+ ✅ |
| 테스트 커버리지 (전체) | {{X}}% | 70%+ |
| 빌드 시간 | {{X}}s | 60s 이하 |
| Lighthouse 점수 (성능/접근성/SEO) | {{X}}/{{Y}}/{{Z}} | 90/90/90 |

> 측정 환경: {{사양}} / 측정 도구: JMeter, Lighthouse, JaCoCo

---

## 🚀 빠른 시작

### 사전 요구사항

- JDK 21+
- Docker & Docker Compose
- (선택) Make

### 1분 만에 띄우기

```bash
# 1. 저장소 클론
git clone https://github.com/{{org}}/{{repo}}.git
cd {{repo}}

# 2. 환경변수 준비
cp .env.example .env
# .env 파일을 열어 필요 값 입력 (DB 비밀번호, 외부 API 키 등)

# 3. 도커로 전체 스택 띄우기 (App + DB + Redis + Nginx)
docker compose up -d

# 4. 접속
open http://localhost:8080
```

### 개발 모드 (로컬 IDE에서 실행)

```bash
# DB·Redis만 도커로
docker compose -f docker-compose.dev.yml up -d

# Spring Boot 로컬 실행
./gradlew bootRun --args='--spring.profiles.active=dev'
```

### 테스트 실행

```bash
./gradlew test                              # 전체
./gradlew test --tests "*ReservationServiceTest*"  # 단일
./gradlew jacocoTestReport                  # 커버리지 리포트
```

상세 가이드: [`docs/14_배포_가이드.md`](docs/14_배포_가이드.md)

---

## 📁 프로젝트 구조

```
{{repo}}/
├── src/main/
│   ├── java/com/{{org}}/{{project}}/
│   │   ├── config/           ← Spring 설정 (Security, Web, Cache)
│   │   ├── common/           ← 전역 예외, 공통 DTO, 인터셉터
│   │   ├── domain/           ← 도메인별 패키지 (모듈)
│   │   │   ├── reservation/  ← controller·service·repository·entity
│   │   │   ├── user/
│   │   │   └── ...
│   │   └── external/         ← 외부 API 클라이언트
│   └── resources/
│       ├── templates/        ← Mustache 뷰
│       ├── static/           ← CSS·JS·이미지
│       └── application*.properties
├── src/test/                 ← 테스트 코드 (구조 동일)
├── docs/                     ← 설계 문서 (00~16)
├── docker/
├── .github/workflows/        ← CI/CD
├── AGENTS.md                 ← AI 에이전트 컨텍스트
└── README.md
```

---

## 👥 팀

| 사진 | 이름 | 역할 | 주요 기여 | GitHub |
| --- | --- | --- | --- | --- |
| <img src="https://github.com/{{user1}}.png" width="50"/> | {{이름1}} | 책임개발자 | 아키텍처·Security·공유 레이어·배포 | [@{{user1}}](https://github.com/{{user1}}) |
| <img src="https://github.com/{{user2}}.png" width="50"/> | {{이름2}} | Backend | {{모듈명}} CRUD, 외부 API 연동 | [@{{user2}}](https://github.com/{{user2}}) |
| <img src="https://github.com/{{user3}}.png" width="50"/> | {{이름3}} | Backend | {{모듈명}} CRUD, 통계·관리자 | [@{{user3}}](https://github.com/{{user3}}) |
| <img src="https://github.com/{{user4}}.png" width="50"/> | {{이름4}} | Frontend | {{화면 그룹}} UI/UX | [@{{user4}}](https://github.com/{{user4}}) |
| <img src="https://github.com/{{user5}}.png" width="50"/> | {{이름5}} | Frontend | {{화면 그룹}} UI/UX, 반응형 | [@{{user5}}](https://github.com/{{user5}}) |
| <img src="https://github.com/{{user6}}.png" width="50"/> | {{이름6}} | DevOps/QA | CI/CD, 모니터링, 테스트 자동화 | [@{{user6}}](https://github.com/{{user6}}) |

---

## 📝 회고 & 학습

### 잘한 점 (Keep)
- {{포인트 1}}
- {{포인트 2}}

### 아쉬운 점 (Problem)
- {{포인트 1}}
- {{포인트 2}}

### 다음에 시도할 것 (Try)
- {{포인트 1}}
- {{포인트 2}}

**상세 회고:** [`docs/21_프로젝트_회고.md`](docs/21_프로젝트_회고.md)
**기술 블로그:** [노션 — 우리가 배운 것](https://www.notion.so/...)

---

## 📄 라이선스

MIT License — 자유롭게 사용·수정·배포 가능. 자세한 내용은 [LICENSE](LICENSE) 참조.

---

<div align="center">

**⭐ 도움이 되었다면 Star를, 의견이 있다면 Issue를 남겨주세요!**

Made with 💙 by Team {{TeamName}} (2026)

</div>
