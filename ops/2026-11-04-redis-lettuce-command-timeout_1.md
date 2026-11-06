# Lettuce: failover 후에도 요청이 계속 실패한 이유와 타임아웃 조합

- 클러스터 failover 완료(수 초) ≠ 클라이언트 복구. Lettuce가 옛 토폴로지를 보고 죽은 노드로 계속 명령을 보냄
- command timeout 60초가 문제: 죽은 노드에 이미 발행된 명령은 토폴로지가 갱신돼도 재라우팅되지 않고 60초를 꽉 채워 기다림
- "5초로 줄이면 갱신 주기(30초)보다 빨라서 또 죽은 노드로 가지 않나?" → adaptive refresh가 장애 징후에 반응해 즉시 갱신하므로 괜찮음
- 설정 방향: command timeout 짧게 + adaptive refresh 활성화 + `disconnectedBehavior(REJECT_COMMANDS)`로 끊긴 노드에 명령 안 쌓기

교훈: 타임아웃은 command/connect/토폴로지 갱신이 맞물린 조합이다. "길게 = 안전"이 아니라 긴 command timeout이 오히려 복구를 지연시킨다.
