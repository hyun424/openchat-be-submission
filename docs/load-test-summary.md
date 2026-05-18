# Load Test Summary

## 목적

OpenChat의 부하 테스트는 WebSocket 동시 접속 수 자체보다 메시지 입력량, fan-out work, DB 저장 정합성, 사용자 체감 지연을 분리해서 관측하는 데 초점을 둡니다.

## 접근 방식

- k6로 REST + WebSocket 시나리오를 구성했습니다.
- sender, observer, validator 역할을 분리해 부하 생성 비용과 관측 비용을 구분했습니다.
- 단일 hot room, multi hot room, active/passive session, partition fan-out 시나리오를 분리했습니다.
- 테스트 후 DB row count, ACK count, WebSocket 지표를 함께 비교했습니다.

## 주요 학습

- 모든 연결 세션에 full payload를 보내면 사용자 수가 늘수록 fan-out work가 급격히 증가합니다.
- active/passive 세션 분리는 서버 전송량을 줄이면서도 messages-after 복구 경로로 정합성을 유지할 수 있습니다.
- partition 기반 fan-out은 DB 저장/ACK는 한 번만 수행하면서 delivery work를 여러 realtime node로 분산하는 데 효과적입니다.
- 부하 생성기 자체의 CPU/관측 비용도 병목이 될 수 있으므로 테스트 환경과 애플리케이션 병목을 분리해야 합니다.
