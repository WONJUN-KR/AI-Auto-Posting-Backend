# AI-Auto-Posting-Backend

AI 기반 블로그 콘텐츠 생성과 자동 업로드를 하나의 파이프라인으로 묶은 통합 프로젝트의 백엔드 서버입니다.
쇼핑몰 상품 홍보 자동화에 특화된 서비스이며, Spring Boot 기반으로 사용자/워크플로우/작업 상태를 관리합니다.

## 프로젝트 개요

- 프로젝트명: OCP (Online Commerce Promotion)
- 개발 기간: 2025.11 ~ 2025.12
- 팀 구성: 6명 (Backend 3, Frontend 2, AI 1)
- 담당 역할: Backend

## 시연 영상

https://github.com/user-attachments/assets/002db9a0-98e8-41f5-a7a1-5d2380e73112

## 담당 업무

### 통계 API
- 운영 현황 파악을 위해 관리자 사용자 통계 조회 API 구현
- 운영 현황 파악을 위해 관리자 포스팅 통계 조회 API 구현
- 플랫폼별 성과 분석을 위해 블로그 플랫폼별 발행 통계 조회 API 구현
- 통계 데이터 누락 및 날짜 범위 표시 오류 수정

### 자동 집계
- 수동 집계 부담 해소를 위해 일별 통계 자동 집계 스케줄러 구현
- 일별 포스팅 통계 자동 집계 추가

### 공통 기능
- 코드 일관성 유지를 위해 공통코드 관리 기능 구현
- CommonCode groupId 문자열 타입으로 통일 및 Session 충돌 해결
- 관리자 사용자 관리 기능 구현

### 리팩토링 및 환경 설정
- 라이브러리 충돌 해결을 위해 Spring Boot 버전 및 의존성 호환성 수정 (Swagger 충돌 해결)
- ApiResponse를 ApiResult로 변경하여 Swagger 충돌 해결
- Admin 컨트롤러를 admin 패키지로 이동
- api 패키지를 controller로 통일
- monitoring domain entities 추가
- 프로젝트 기본 세팅

## 기술 스택

- Language: Java 21
- Framework: Spring Boot 3.3, Spring Security OAuth2
- ORM: JPA
- Scheduler: Quartz
- Messaging: RabbitMQ
- DB: MySQL, PostgreSQL (Airflow 메타 DB)
- Cache/Broker: Redis

## 시스템 구성도

1. 워크플로우 기반 Quartz job 등록
2. 등록된 반복 규칙 1시간 전 Backend → RabbitMQ에 `ContentGenerateRequest` 발행 및 Airflow 트리거
3. Airflow → RabbitMQ에서 `ContentGenerateRequest` 수신
4. 트렌드 키워드 크롤링 → GPT 기반 키워드 선택
5. 상품 선택 (크롤된 상품 활용 또는 URL 기반 검색 분기)
6. AI 콘텐츠 생성
7. 단계별 웹훅 전송 + Airflow 로그 수집 웹훅
8. 등록된 반복 규칙 정시 Backend → RabbitMQ에 `BlogUploadRequest` 발행
9. Blog-Upload-Module → RabbitMQ에서 `BlogUploadRequest` 수신
10. 블로그 업로드 파이프라인 실행
11. 결과 웹훅 전송

## 실행 방법

### 1) 인프라 실행
```bash
docker compose up -d mysql rabbitmq redis postgres
```

### 2) 백엔드 실행
```bash
./gradlew bootRun
```

### 3) 환경 변수
`src/main/resources/application.yml` 기반으로 로컬 기본값이 존재하며, `.env` 파일로 오버라이드할 수 있습니다.

## 관련 레포지토리

- Frontend: [AI-Auto-Posting-Frontend](https://github.com/WONJUN-KR/AI-Auto-Posting-Frontend)
- 통합 레포: [Kernel-Final-Project/integration](https://github.com/Kernel-Final-Project/integration)
