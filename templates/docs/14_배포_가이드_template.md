# {{EMOJI}} {{PROJECT_NAME}} — 배포 가이드

> **문서 버전:** v1.0
> **작성일:** YYYY-MM-DD

---

## 목차
1. 배포 환경
2. 사전 준비
3. 환경변수 설정
4. Docker Compose 배포
5. SSL 인증서
6. DB 초기화
7. 서비스 기동 & 상태 확인
8. 로그·백업·모니터링
9. 보안 체크리스트
10. 트러블슈팅

---

## 1. 배포 환경

### 1.1 권장 사양
- OS: Ubuntu 22.04
- CPU: 2 core 이상
- RAM: 4GB 이상 (외부 LLM 연동 시 8GB+ 권장)
- Disk: 20GB 이상

### 1.2 시스템 아키텍처

```
[Client] ─HTTPS─> [Nginx:80/443] ─> [Spring Boot:8080]
                                         │
                                         ├─> [MySQL:3306]
                                         ├─> [Redis:6379]
                                         └─> [External API]
```

### 1.3 컨테이너 구성
- `nginx` — 리버스 프록시 + SSL
- `app` — Spring Boot
- `mysql` — DB
- `redis` — 세션

---

## 2. 사전 준비

### 2.1 Docker & Docker Compose

```bash
# Ubuntu 22.04
sudo apt update && sudo apt install -y docker.io docker-compose-v2
sudo systemctl enable --now docker
sudo usermod -aG docker $USER  # 재로그인 필요
docker --version && docker compose version
```

### 2.2 JDK 21 (네이티브 빌드 시)

```bash
sudo apt install -y openjdk-21-jdk
java -version
```

---

## 3. 환경변수 설정

### 3.1 `.env` 파일 생성
```bash
cp .env.example .env
# 실제 값 기입
```

### 3.2 전체 환경변수 목록

| 변수 | 설명 | 예 |
| --- | --- | --- |
| `SPRING_PROFILES_ACTIVE` | 프로필 | `prod` |
| `DB_HOST` | MySQL 호스트 | `mysql` |
| `DB_PORT` | MySQL 포트 | `3306` |
| `DB_NAME` | DB명 | `{{project}}` |
| `DB_USER` | DB 사용자 | `app` |
| `DB_PASSWORD` | DB 비밀번호 | `********` |
| `REDIS_HOST` | Redis 호스트 | `redis` |
| `REDIS_PORT` | Redis 포트 | `6379` |
| `{{EXTERNAL}}_API_KEY` | 외부 API 키 | `...` |
| `SERVER_PORT` | 앱 포트 | `8080` |

### 3.3 `.env` 예시

```
SPRING_PROFILES_ACTIVE=prod
DB_HOST=mysql
DB_PORT=3306
DB_NAME={{project}}
DB_USER=app
DB_PASSWORD=change-me
REDIS_HOST=redis
REDIS_PORT=6379
{{EXTERNAL}}_API_KEY=...
```

---

## 4. Docker Compose 배포

### 4.1 소스 코드 다운로드
```bash
sudo mkdir -p /opt/{{project}}
sudo chown -R $USER:$USER /opt/{{project}}
cd /opt/{{project}}
git clone https://github.com/{{org}}/{{repo}}.git .
```

### 4.2 SSL 인증서 준비
```bash
# 자체 서명 (개발/테스트)
mkdir -p nginx/ssl
openssl req -x509 -nodes -newkey rsa:2048 \
  -days 365 \
  -keyout nginx/ssl/key.pem \
  -out nginx/ssl/cert.pem
# 운영: Let's Encrypt 또는 기관 인증서를 nginx/ssl/에 배치
```

### 4.3 전체 서비스 기동
```bash
docker compose up -d
docker compose ps
docker compose logs -f app
```

---

## 5. SSL 인증서 (Let's Encrypt 예)

```bash
sudo apt install -y certbot
sudo certbot certonly --standalone -d {{도메인}}
# 생성된 인증서를 nginx/ssl/로 복사 / 심볼릭 링크
```

---

## 6. DB 초기화

```bash
# 최초 기동 시 schema.sql, data.sql 자동 실행
# 수동 복원
docker compose exec mysql \
  mysql -u root -p{{password}} {{project}} < backup/init.sql
```

---

## 7. 서비스 기동 & 상태 확인

```bash
# 헬스체크
curl -k https://localhost/actuator/health

# 로그
docker compose logs -f --tail=200 app

# 재시작
docker compose restart app
```

---

## 8. 로그·백업·모니터링

### 8.1 로그
- 경로: `/opt/{{project}}/logs/app.log` (logback 설정)
- rotate: 일 단위, 30일 보관

### 8.2 DB 백업 (cron)
```bash
0 3 * * * docker compose exec -T mysql \
  mysqldump -u root -p{{password}} {{project}} \
  | gzip > /opt/{{project}}/backup/$(date +\%Y\%m\%d).sql.gz
```

### 8.3 모니터링
- Actuator `/actuator/health`, `/actuator/metrics`
- 외부: Prometheus + Grafana (권장)

---

## 9. 보안 체크리스트
- [ ] `.env` 파일 권한 600
- [ ] DB/Redis 포트 외부 노출 금지 (Docker 네트워크 내부)
- [ ] Actuator `/actuator/**` 관리자만 접근
- [ ] API 키 로그 출력 금지
- [ ] HTTPS 강제 (Nginx `return 301 https://...`)
- [ ] Security Headers (HSTS, CSP, X-Frame-Options)
- [ ] 방화벽: 22/80/443만 개방

---

## 10. 트러블슈팅

| 증상 | 원인 후보 | 조치 |
| --- | --- | --- |
| 502 Bad Gateway | app 컨테이너 기동 전 | `docker compose logs app`, 재시작 |
| DB 연결 실패 | env 변수 / 네트워크 | `.env` 확인, mysql 컨테이너 로그 |
| Redis 연결 실패 | 컨테이너 미기동 | `docker compose up -d redis` |
| 세션 끊김 | Redis 재시작 | 세션 저장소 확인 |
| 외부 API 401 | 키 만료 / 오타 | API 키 갱신 |

---

## 11. 롤백 절차

> 운영 배포 후 문제 발생 시 즉시 실행. **DevOps/QA + Lead 합동.**

### 11.1 Docker 이미지 롤백

```bash
# 1. 이전 이미지 태그 확인
docker images | grep {{project}}

# 2. docker-compose.prod.yml의 image 태그를 이전 버전으로 변경
# image: ghcr.io/{{org}}/{{project}}:v1.0.0-prev  ← 이전 버전으로

# 3. 재배포
docker compose -f docker-compose.prod.yml up -d --force-recreate app

# 4. 헬스체크
sleep 15
curl -f https://{{도메인}}/actuator/health || echo "❌ 롤백 실패"
echo "✅ 롤백 완료"
```

### 11.2 DB 롤백 (스키마 변경 시)

```bash
# DDL 변경사항이 있는 경우에만 실행
# 사전에 백업된 SQL 파일을 복원
docker compose exec -T mysql   mysql -u root -p${MYSQL_ROOT_PASSWORD} {{project}} < backup/pre-deploy-backup.sql
```

> ⚠️ DB 롤백은 **데이터 손실 위험**이 있으므로 Lead의 최종 승인 후 실행.

### 11.3 롤백 결정 기준

| 상황 | 조치 |
| --- | --- |
| 배포 후 헬스체크 실패 | 즉시 자동 롤백 |
| Critical 에러율 1% 초과 | 5분 내 수동 롤백 결정 |
| 시연 시나리오 핵심 기능 불가 | 즉시 롤백 + 백업 영상 |
| 비핵심 기능 문제 | 핫픽스 후 재배포 (롤백 없이) |

---

## 12. 배포 전 체크리스트

### 12.1 스테이징 배포 전 (W1 금, W2 금)

```
[ ] develop 브랜치 CI green
[ ] 전원 로컬 bootRun 성공 확인
[ ] application-staging.properties 환경변수 확인
[ ] DB 마이그레이션 스크립트 검토 (DDL 변경 있으면)
[ ] 스테이징 배포 후 헬스체크: /actuator/health
[ ] 스테이징 URL 팀 전체 공유
```

### 12.2 운영 배포 전 (W3 목)

```
[ ] 스테이징에서 시연 시나리오 전체 통과 확인
[ ] 통합 테스트 + E2E 테스트 CI green
[ ] DB 백업 실행 (배포 전 스냅샷)
[ ] 운영 환경변수 재확인 (민감 정보 점검)
[ ] docker-compose.prod.yml 최신 이미지 태그 확인
[ ] 배포 시간: 팀원 2명 이상 대기 중
[ ] 롤백 절차 숙지 (11번 섹션)
```

### 12.3 운영 배포 후

```
[ ] /actuator/health green 확인
[ ] DB 연결: /actuator/health/db green
[ ] Redis 연결: /actuator/health/redis green
[ ] 시연 시나리오 핵심 화면 1회 수동 확인
[ ] 시드 데이터 삽입: sh scripts/seed-data.sh
[ ] 외부 URL 팀 전체 공유 + Slack 공지
[ ] 배포 완료 태그: git tag v1.0.0 && git push origin v1.0.0
```

---

## 📌 변경 이력

| 버전 | 날짜 | 작성자 | 주요 변경 |
| --- | --- | --- | --- |
| v1.0 | YYYY-MM-DD | DevOps/QA | 초기 작성 |
