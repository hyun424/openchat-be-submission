# OpenChat Backend

OpenChat Backend는 실시간 채팅의 메시지 저장, WebSocket fan-out, 채팅방/멤버 관리, 인증, 그리고 부하 테스트 기반 확장성 검증을 포함한 Spring Boot 백엔드 프로젝트입니다.

## 주요 기능

- JWT 기반 인증과 Google OAuth 온보딩 흐름
- 채팅방/멤버/메시지 REST API
- WebSocket 기반 실시간 메시지 송수신
- Redis/Kafka 기반 fan-out 및 장애 대응 구조
- hot room fan-out 최적화와 active/passive 세션 처리
- room partition ownership, reconnect/resync, node drain 설계
- k6와 Terraform 기반 부하 테스트 시나리오

## 기술 스택

- Java 17
- Spring Boot
- Spring Security / OAuth2 Client / JWT
- Spring Data JPA
- MySQL
- Redis
- Kafka
- WebSocket
- Docker Compose
- k6
- Terraform / GCP load-test templates

## 로컬 실행

```bash
cp .env.example .env
# .env 값을 로컬 환경에 맞게 수정

docker compose up -d mysql redis kafka
./gradlew bootRun
```

## 테스트

```bash
./gradlew test
```

## 부하 테스트

k6 시나리오는 `k6/scenarios/`에 있습니다. 로컬 또는 별도 테스트 환경에서 실행할 수 있습니다.

```bash
cd k6
make help
```

## 설계 요약

- 실시간 메시지는 DB 저장과 client ACK/fan-out 경계를 분리해 durability와 latency를 함께 관리합니다.
- hot room은 모든 세션으로 동일 payload를 보내는 구조가 병목이 되므로 active/passive 세션과 partition 기반 fan-out으로 전송 work를 줄입니다.
- scale-out 환경에서는 route node, connected node, partition owner, subscriber readiness를 하나의 ownership contract로 맞춰 메시지 누락과 drain 실패를 방지합니다.
- 성능 검증은 단순 VU 수가 아니라 input TPS, DB rows, ACK count, visible freshness, fan-out work를 분리해서 해석합니다.

## 제출용 정리 기준

이 저장소는 제출용 clean snapshot입니다.

- 기존 개발 히스토리와 로컬 실행 산출물은 포함하지 않았습니다.
- 원본 작업 로그/내부 문서/agent 운영 문서는 제외했습니다.
- 민감 정보는 `.env.example`의 placeholder로만 제공합니다.
