# OCP(Online Commerce Promotion) 프로젝트 통합(Backend + Airflow + AI 모듈)

AI 기반 블로그 콘텐츠 생성과 자동 업로드를 하나의 파이프라인으로 묶은 통합 프로젝트이며, 쇼핑몰 상품 홍보 자동화에 특화된 서비스입니다.  
Spring Boot 백엔드가 사용자/워크플로우/작업 상태를 관리하고, Airflow + AI 모듈이 트렌드 키워드·상품 선정·콘텐츠 생성을 수행하며, 별도 Python 업로더가 블로그에 자동 게시합니다.

## 시연 영상
https://github.com/user-attachments/assets/002db9a0-98e8-41f5-a7a1-5d2380e73112


## 주요 기능

- OAuth2 소셜 로그인 (Google/Naver/Kakao)
- 트렌드 기반 콘텐츠 생성 워크플로우
- 키워드/상품 선택 결과와 생성 콘텐츠를 단계별 웹훅으로 전달
- 블로그 자동 업로드 및 결과 콜백 처리
- 작업/로그/통계/관리자 기능 및 감사(Audit) 기록
- 사이트/콘텐츠 요청 관리, 스케줄링 및 반복 규칙 지원

## 구성 요소

- Backend: `Spring Boot` 기반 API 서버 (사용자/작업/워크플로우/관리 기능)
- Frontend: 별도 레포지토리 [Kernel-Final-Project/Frontend (dev 브랜치 사용)](https://github.com/Kernel-Final-Project/Frontend)
- Airflow: 트렌드 파이프라인 실행 및 단계별 웹훅 전송
- AI Module: 키워드 크롤링, 키워드/상품 선택, 콘텐츠 생성
- Blog Upload Module: 네이버/티스토리 자동 업로드 CLI/워커
- Infra: MySQL, RabbitMQ, Redis, PostgreSQL(Airflow 메타DB)

## 기술 스택

- Backend: Java 21, Spring Boot 3.3, Spring Security OAuth2, JPA, Quartz
- AI/ETL: Python, Airflow (CeleryExecutor)
- Messaging: RabbitMQ
- DB: MySQL (서비스), PostgreSQL (Airflow 메타 DB)
- Cache/Broker: Redis (Airflow Celery broker)

## ERD
![](./docs/OCP_ERD.png)

## 시스템 구성도
![](./docs/OCP_구성도.png)

1. 워크플로우 기반 Quartz job 등록
2. 등록된 반복 규칙 1시간 전 Back->RabbitMQ에  `ContentGenerateRequest` 발행 및 Airflow트리거
3. Airflow->RabbitMQ에서 `ContentGenerateRequest` 수신
4. 트렌드 키워드 크롤링 → GPT 기반 키워드 선택
5. 상품 선택 (크롤된 상품 활용 또는 URL 기반 검색 분기)
6. AI 콘텐츠 생성
7. 단계별 웹훅 전송 + Airflow 로그 수집 웹훅
8. 등록된 반복 규칙 정시 Back->RabbitMQ에 `BlogUploadRequest` 발행
9. Blog-Upload-Module->RabbitMQ에서 `BlogUploadRequest` 수신
10. 블로그 업로드 파이프라인 실행
11. 결과 웹훅 전송


## 디렉터리 구조

```
.
├── src/                       # Spring Boot 백엔드
├── airflow/                   # Airflow Dockerfile/requirements/dags
├── ai_module/                 # 키워드/상품/콘텐츠 생성 모듈
├── blog_upload_module/        # 블로그 업로드 CLI/워커
├── docs/                      # ERD 등 문서 리소스
└── docker-compose.yml          # 로컬 개발용 인프라/에어플로우 스택
```




## 환경 변수/설정

`src/main/resources/application.yml` 기반으로 로컬 기본값이 존재하며, `.env` 파일로 오버라이드할 수 있습니다.

- DB: `MYSQL_HOST`, `MYSQL_PORT`, `MYSQL_USER`, `MYSQL_PASSWORD`
- OAuth2: `GOOGLE_CLIENT_ID/SECRET`, `NAVER_CLIENT_ID/SECRET`, `KAKAO_CLIENT_ID/SECRET`
- Airflow API: `AIRFLOW_BASE_URL`, `AIRFLOW_USERNAME`, `AIRFLOW_PASSWORD`
- Webhook: `BLOG_UPLOAD_WEBHOOK_*`, `KEYWORD_SELECT_WEBHOOK_*`, `PRODUCT_SELECT_WEBHOOK_*`, `CONTENT_GENERATE_WEBHOOK_*`, `AIRFLOW_LOG_WEBHOOK_*`

Airflow Variable(필수):

- `rabbitmq_url` (예: `amqp://guest:guest@rabbitmq:5672/`)
- `rabbitmq_trend_queue` (기본: `content-generate-queue`)
- `openai_api_key`


## 실행 방법

### 1) 인프라 + Airflow 실행

```bash
docker compose up -d mysql rabbitmq redis postgres
docker compose up airflow-init
docker compose up -d airflow-webserver airflow-scheduler airflow-worker airflow-triggerer airflow-flower
```

- 서비스 포트
  - Backend API: http://localhost:8080
  - MySQL: 3306
  - RabbitMQ AMQP: 5672
  - RabbitMQ 관리 UI: http://localhost:15672
  - Redis: 6379
  - Airflow Web UI: http://localhost:8081
  - Airflow Flower: http://localhost:5555
- Airflow 기본 계정: `admin` / `admin`

### 2) 백엔드 실행

```bash
./gradlew bootRun
```

### 3) AI/업로드 모듈 사용

- AI Module은 Airflow 컨테이너에서 직접 참조됩니다 (`PYTHONPATH`로 마운트).
- 블로그 업로드 기능은 `blog_upload_module/README.md` 참고.
