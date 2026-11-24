# ShedLock이 있다는 건 "배치가 어느 노드에서 돌지 모른다"는 뜻 — 방화벽은 전 노드에서 열어야 한다

- 배치 모듈에서 다른 네트워크 존의 admin API를 호출해 통계 데이터를 수집해야 했고, 존 간 통신이라 방화벽 오픈 요청이 필요했음. "AP 서버 전부 열어야 하나?"라는 질문에 답하려다 배운 것
- **ShedLock은 분산 스케줄러가 아니라 중복 실행 방지 락**: 모든 노드에서 스케줄이 동시에 트리거되고, 공유 저장소(DB의 `shedlock` 테이블 등)에 `lock_until` 레코드를 먼저 쓰는 노드 하나만 실행, 나머지는 스킵. "누가 이기는지"는 매 실행마다 비결정적
- 따라서 ShedLock을 쓰고 있다는 사실 자체가 "이 잡은 배치가 배포된 **아무 노드에서나** 돈다"는 증거. 방화벽을 1대만 열면 그 노드가 락을 잡은 날만 성공하는 **간헐 실패**가 되고, 이런 장애는 재현이 안 돼 원인 찾기가 제일 괴로움
- 결론: AP 서버 전부가 아니라 **배치 모듈이 배포된 노드 전부**(1대면 1대, N대면 N대)에서 오픈. "배치가 몇 대에 배포돼 있는지"를 먼저 확인하는 게 방화벽 요청의 선행 질문
- **네트워크 존(security zone)**: 동일한 보안 요구사항을 가진 네트워크 구간의 묶음. 방화벽 정책은 출발지 존→목적지 존 단위로 정의되고, 존 간 트래픽은 기본 차단(default deny)이라 출발지→목적지 IP:포트를 건건이 허용해야 함 — 그래서 출발지(배치 노드) 목록이 정확해야 함
- 덤: 목적지는 서버 IP 직결보다 **nginx 도메인 경유**로 잡기. 목적지 앱이 다른 서버로 옮겨가도 방화벽·설정 재작업이 없음. 목적지가 단일 인스턴스라면 수집 실패를 전제로 멱등 적재(DELETE+INSERT) + 누락일 재수집을 설계에 넣을 것

교훈: 분산 락은 "한 노드만 실행"을 보장하지 "어느 노드가 실행"은 보장하지 않는다. 인프라 요청(방화벽)은 애플리케이션 아키텍처(락·배포 토폴로지)를 알아야 정확해진다.

## References

- [ShedLock — GitHub](https://github.com/lukas-krecan/ShedLock)
- [ShedLock in Spring Scheduler (Baeldung)](https://www.baeldung.com/shedlock-spring)
- [Network Segmentation Using Zones (Palo Alto Networks)](https://docs.paloaltonetworks.com/pan-os/10-2/pan-os-admin/zone-protection-and-dos-protection/network-segmentation-using-zones.html)
