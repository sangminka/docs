# 16. 운영 매뉴얼

> **목적** 시연 기간 및 발표 후 운영자(팀의 DevOps/QA 담당)가 서비스를 안정적으로 유지하고 장애에 대응할 수 있도록 한다.
> **작성 시점** W3 후반에 초안, 발표 주에 비상 연락망·체크리스트 갱신.
> **{{프로젝트명}} 운영 매뉴얼 / 작성자: {{DevOps 담당자}} / 최종 수정일: {{YYYY-MM-DD}}**

---

## 1. 운영 개요

### 1.1 시스템 한눈에 보기

```
[사용자 브라우저]
        │ HTTPS
        ▼
[CloudFront / nginx]
        │
        ▼
[Spring Boot App x N]
   │       │
   ▼       ▼
[MySQL]  [Redis]
```

### 1.2 환경

| 환경 | URL | 인프라 | 용도 |
|------|------|--------|------|
| Local | localhost:8080 | Docker Compose | 개발 |
| Staging | {{staging.example.com}} | {{AWS EC2 t3.small}} | 검증, 시연 리허설 |
| Production | {{prod.example.com}} | {{AWS EC2 t3.medium}} | 시연 본 |

### 1.3 책임 분담

| 영역 | 1차 담당 | 백업 |
|------|----------|------|
| 인프라 / 배포 | {{DevOps 담당}} | {{Lead}} |
| 데이터베이스 | {{DevA}} | {{DevOps 담당}} |
| 애플리케이션 (auth) | {{DevB}} | {{DevC}} |
| 애플리케이션 ({{기능1}}) | {{DevC}} | {{DevD}} |
| 모니터링 / 알람 | {{DevOps 담당}} | {{Lead}} |

---

## 2. 일상 운영

### 2.1 매일 (시연 주간 09:00)

- [ ] Health check 응답 200 확인 (`/actuator/health`)
- [ ] 어제 발생한 ERROR 로그 0건 또는 분류 완료 확인
- [ ] 모니터링 대시보드 주요 지표 정상 범위 확인
- [ ] 디스크 사용률 80% 미만 확인
- [ ] 백업 작업 정상 종료 여부 확인

### 2.2 매주

- [ ] 의존성 보안 취약점 스캔 (Dependabot 알림 검토)
- [ ] 로그 로테이션 및 30일 초과 로그 아카이브
- [ ] DB 슬로우 쿼리 점검 (응답 1초 초과 쿼리)
- [ ] 백업 복구 리허설 (분기 1회 이상 권장)

### 2.3 발표 주 특별 운영

| 시각 | 액션 | 담당 |
|:----:|------|------|
| 매일 09:00 | 헬스체크 + 회귀 테스트 트리거 | {{DevOps}} |
| 매일 14:00 | 시연 시나리오 수동 리허설 | 발표자 |
| 발표 D-1 18:00 | 데이터 시드 리셋 + 시연 계정 재생성 | {{DevOps}} |
| 발표 D-Day 30분 전 | 최종 헬스체크, 알람 확인, 백업 노트북 준비 | 전원 |

---

## 3. 주요 운영 작업

### 3.1 배포

운영 환경 배포는 `main` 브랜치에 PR이 머지되면 GitHub Actions가 자동 수행한다 (자세한 내용은 **20_CICD_파이프라인** 참조).

#### 수동 배포 (롤백 포함)

```bash
# Staging 배포
gh workflow run stage-deploy.yml --ref main

# Production 배포
gh workflow run prod-deploy.yml --ref main

# 롤백 (직전 버전으로)
gh workflow run prod-deploy.yml --ref {{이전_tag}}
```

> 📌 **시연 주 동안은 prod 배포 동결**. 결함 수정은 hotfix 브랜치 → staging 검증 → 시연 외 시간에만 prod 반영.

### 3.2 데이터베이스

#### 접속

```bash
# Staging
mysql -h {{staging-db}} -u readonly -p

# Production (RDS bastion 경유)
ssh bastion -L 3306:{{rds-endpoint}}:3306
mysql -h 127.0.0.1 -u {{user}} -p
```

> 🚫 **운영 DB에 직접 UPDATE/DELETE 금지**. 반드시 마이그레이션 파일로 변경.

#### 마이그레이션 적용

Flyway가 애플리케이션 부팅 시 자동 적용. 수동 실행이 필요한 경우:

```bash
./gradlew flywayMigrate -Pdb=prod
```

#### 백업

| 종류 | 주기 | 보관 | 위치 |
|------|------|------|------|
| 자동 스냅샷 | 매일 03:00 | 7일 | RDS 자동 백업 |
| 수동 dump | 시연 주 매일 | 14일 | S3 `{{bucket}}/backups/` |
| 시드 데이터 | 시연용 고정 | 영구 | 리포지토리 `db/seed/` |

#### 복구 절차

```bash
# 1. 점검 모드 진입 (애플리케이션 maintenance 페이지로 전환)
gh workflow run maintenance-on.yml --ref main

# 2. 스냅샷에서 RDS 복구
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier {{prod-restored}} \
  --db-snapshot-identifier {{snapshot-id}}

# 3. 엔드포인트 변경 후 재배포
gh workflow run prod-deploy.yml --ref main

# 4. 점검 모드 해제
gh workflow run maintenance-off.yml --ref main
```

### 3.3 캐시 / Redis

```bash
# Redis 접속
redis-cli -h {{redis-host}} -a {{password}}

# 특정 키 삭제 (예: 캐시 무효화)
redis-cli DEL "user:123:profile"

# 전체 flush (운영 환경에서 절대 금지, staging만)
redis-cli FLUSHDB
```

### 3.4 로그

| 로그 | 위치 | 보관 |
|------|------|------|
| 애플리케이션 | CloudWatch Logs / `/var/log/app/*.log` | 30일 |
| Nginx access | CloudWatch / `/var/log/nginx/access.log` | 30일 |
| 보안 (인증 실패 등) | 별도 그룹, 90일 | 90일 |

```bash
# 최근 ERROR 로그 빠르게 보기
aws logs tail /aws/app/{{프로젝트명}} --filter-pattern "ERROR" --since 1h
```

---

## 4. 모니터링 & 알람

### 4.1 대시보드

| 대시보드 | URL | 주요 지표 |
|----------|------|-----------|
| 애플리케이션 | {{Grafana URL}} | 응답시간, 에러율, RPS |
| 인프라 | {{CloudWatch URL}} | CPU, 메모리, 디스크 |
| DB | {{RDS Performance Insights}} | 슬로우 쿼리, 커넥션 |

### 4.2 알람 임계값

| 지표 | Warning | Critical | 통보 채널 |
|------|---------|----------|-----------|
| 5xx 에러율 | > 1% (5분) | > 5% (5분) | Slack `#alerts` |
| p95 응답시간 | > 1s | > 3s | Slack `#alerts` |
| CPU | > 70% | > 90% | Slack `#alerts` |
| 디스크 | > 70% | > 85% | Slack + SMS |
| 헬스체크 실패 | 1회 | 3회 연속 | Slack + SMS |

> 발표 주는 임계값을 한 단계 더 보수적으로 낮춘다.

---

## 5. 장애 대응 (Incident Response)

### 5.1 장애 등급

| 등급 | 정의 | 대응 시간 | 사후 보고 |
|:----:|------|:---------:|:---------:|
| P0 | 시연 완전 불가 | 즉시 | 24시간 내 |
| P1 | 핵심 기능 일부 불가 | 30분 내 | 48시간 내 |
| P2 | 사용성 저해, 우회 가능 | 4시간 내 | 1주 내 |
| P3 | 사소한 문제 | 다음 영업일 | 불필요 |

### 5.2 P0/P1 장애 대응 흐름

```
탐지 → 즉시 Slack #incident 채널 알림 → On-call 담당 응답 (10분 내)
  → 영향 범위 파악 → 임시 조치 (롤백 / 점검 모드) → 사용자 공지
  → 근본 원인 분석 → 영구 수정 → 사후 보고서 작성
```

### 5.3 자주 쓰는 응급 조치

| 증상 | 1차 조치 | 2차 조치 |
|------|----------|----------|
| 5xx 폭주 | 직전 버전 롤백 | 트래픽 차단 + 점검 모드 |
| DB 커넥션 고갈 | 앱 재시작 | DB 리부트 (최후 수단) |
| 디스크 풀 | 로그 정리 | 인스턴스 사이즈 업 |
| CPU 100% | 스레드 덤프 후 재시작 | 스케일 아웃 |
| Redis 장애 | 캐시 우회 모드 토글 | Redis 재시작 |

### 5.4 시연 중 장애 발생 시 (특별 절차)

1. 발표자는 **당황하지 않고** "백업 시연 영상으로 전환하겠습니다"라고 안내
2. **사전 녹화한 데모 영상**으로 즉시 전환 (D-2까지 준비, 19_시연_준비_가이드 참조)
3. 발표 후 별도 시간에 라이브 시연 재시도 가능 여부 안내
4. 사후 보고서 작성

---

## 6. 보안

### 6.1 비밀 관리

- 모든 시크릿은 **GitHub Secrets** 또는 **AWS Secrets Manager**에만 저장
- `.env` 파일은 절대 커밋 금지 (`.gitignore` 등록 + pre-commit hook으로 차단)
- 의심되는 누출 발견 시 즉시 키 회전 + 영향 범위 파악

### 6.2 접근 통제

| 리소스 | 접근 권한자 |
|--------|-------------|
| AWS 콘솔 | {{DevOps}}, {{Lead}} |
| RDS Production | {{DevOps}} (쓰기), 전체 (읽기 전용) |
| GitHub Repo | 전체 (Maintainer는 Lead만) |
| 도메인 / DNS | {{DevOps}}, {{Lead}} |

### 6.3 사용자 데이터 처리

- 비밀번호는 BCrypt strength 12 단방향 해시
- 개인정보(이메일 등)는 DB 암호화 컬럼 또는 KMS 적용
- 로그·에러 메시지에 비밀번호·토큰·이메일이 평문으로 남지 않도록 마스킹

---

## 7. 비상 연락망

| 역할 | 이름 | 연락처 (slack ID / 전화) | On-call 시간 |
|------|------|--------------------------|--------------|
| Lead | {{이름}} | {{@xxx}} / {{010-...}} | 24/7 (시연 주) |
| DevOps | {{이름}} | {{@xxx}} / {{010-...}} | 평일 09-22 |
| Backup | {{이름}} | {{@xxx}} / {{010-...}} | DevOps 부재 시 |

> 시연 D-1 ~ 발표 종료까지는 **전원 24시간 응답 가능 상태** 유지.

---

## 8. 문서 / 도구 링크

| 항목 | 링크 |
|------|------|
| 인프라 다이어그램 | [`docs/03_아키텍처_정의서.md`](./03_아키텍처_정의서.md) |
| CI/CD 파이프라인 | [`docs/20_CICD_파이프라인.md`](./20_CICD_파이프라인.md) |
| Runbook 모음 | [`docs/runbooks/`](./runbooks/) |
| 모니터링 대시보드 | {{Grafana URL}} |
| 알람 채널 | Slack `#alerts`, `#incident` |

---

## 9. 변경 이력

| 버전 | 날짜 | 작성자 | 주요 변경 |
|------|------|--------|----------|
| v1.0 | YYYY-MM-DD | DevOps/QA | 초기 작성 |
| v1.1 | YYYY-MM-DD | DevOps/QA | 발표 주 비상 연락망 갱신 |

---

## 체크리스트

- [ ] 모든 환경의 URL, 인프라 사양, 책임자가 명시되어 있는가
- [ ] 배포·롤백·DB 복구 절차가 명령어 수준까지 구체적인가
- [ ] 알람 임계값이 시연 주에 맞게 보수적으로 조정되어 있는가
- [ ] 시연 중 장애 발생 시 백업 영상 전환 절차가 포함되어 있는가
- [ ] 비상 연락망이 발표 주 시작 전 갱신되었는가
