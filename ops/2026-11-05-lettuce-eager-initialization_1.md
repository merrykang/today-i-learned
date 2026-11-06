# Lettuce setEagerInitialization(true)를 검토 후 적용하지 않은 이유

- 발단: lazy 초기화라 트래픽 1건 전엔 커넥션이 없어 토폴로지 갱신 검증이 안 됨 → eager를 켜면 기동 시 바로 연결돼 검증 편해짐
- 대가: 기동 시 Redis가 장애면 커넥션 수립이 블로킹 → **Redis 가용성이 앱 기동의 전제조건**이 됨 (fail-fast vs 복원력 트레이드오프)
- 이 서비스는 Redis가 흔들려도 앱은 떠 있다가 복구 후 붙는 복원력이 더 중요 → 미적용
- 검증은 워밍업 요청 1건 + `MONITOR`에서 `lettuce#ClusterTopologyRefresh` 주기 확인으로 대체

교훈: 검증 편의 때문에 운영 동작(장애 전파 모델)을 바꾸면 안 된다. 채택하지 않은 결정도 이유와 함께 기록하면 근거가 된다.
