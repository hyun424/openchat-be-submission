# Architecture Overview

## 문제 정의

실시간 채팅은 단순히 WebSocket 연결 수를 늘리는 것만으로 안정성이 보장되지 않습니다. 한 사용자가 보낸 메시지는 저장, ACK, fan-out, 재접속 복구, node drain 상황까지 일관된 계약을 유지해야 합니다.

OpenChat Backend는 다음 문제를 중심으로 설계했습니다.

1. 여러 서버에 WebSocket 세션이 분산될 때 메시지를 누락 없이 전달한다.
2. hot room에서 fan-out work가 폭증하지 않도록 전송 대상을 줄이고 분산한다.
3. Redis/Kafka/DB 장애 경계에서 사용자가 본 메시지와 저장된 메시지의 정합성을 유지한다.
4. 서버 증설/종료/rolling restart 상황에서도 route와 ownership이 일관되게 유지되도록 한다.

## 핵심 구조

### Message ingest

- 메시지는 검증 후 DB에 저장됩니다.
- 저장된 메시지를 기준으로 ACK와 fan-out을 진행해, client가 본 메시지가 DB에 남지 않는 상황을 줄입니다.

### Realtime fan-out

- WebSocket 세션은 room 단위 registry로 관리됩니다.
- active 세션에는 full payload를 전송하고, passive 세션은 필요 시 messages-after API로 복구할 수 있도록 분리합니다.

### Room partition ownership

- hot room의 fan-out work를 room partition 단위로 나눕니다.
- route node, connected node, owner node, subscriber readiness를 함께 검증해 메시지 누락 가능성을 낮춥니다.

### Recovery and drain

- reconnect/resync 경로를 통해 passive 세션, node drain, partition 이동 상황에서 메시지를 다시 따라잡을 수 있게 합니다.
- node drain은 새 owner readiness와 남은 session 수를 확인한 뒤 진행합니다.

## 검증 관점

성능 검증은 다음 지표를 분리해 봅니다.

- input message TPS
- DB write count
- client ACK count
- WebSocket delivery/fan-out work
- visible freshness p95/p99
- route mismatch / fallback / failure
- cleanup residue after load-test runs

이렇게 분리하면 단순히 “동시 접속자가 많다”가 아니라, 어느 단계가 병목인지와 어떤 설계 변경이 효과가 있었는지를 설명할 수 있습니다.
