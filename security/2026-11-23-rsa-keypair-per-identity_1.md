# RSA 키쌍은 "서버별"이 아니라 "신원별"로 만든다

- 배치 모듈이 다른 존의 admin API를 전자서명으로 인증해 호출하는 작업에서 "키쌍을 몇 번 생성해야 하나?"에 답하다 정리한 기준
- **키는 서버의 소유물이 아니라 호출자(신원)의 증명 수단**. 공개키를 등록받는 쪽은 "이 서명이 '운영 배치'라는 신원에서 왔는가"를 검증할 뿐, 어느 물리 서버에서 왔는지는 보지 않는다
- 그래서 단위는 신원 × 환경:
  - **환경별로는 분리** (스테이징 1쌍 + 운영 1쌍): 서버가 달라서가 아니라, 스테이징 키가 유출됐을 때 운영 API까지 뚫리는 구조를 막기 위해(폭발 반경 격리). 보안 점검 관점에서도 환경별 키 분리가 정석
  - **같은 환경 내 노드엔 동일 개인키를 복사**: "운영 배치"라는 신원은 하나이므로 키쌍도 하나. 노드별로 키를 따로 만들면 검증 서버에 등록할 공개키가 노드 수만큼 늘어나 관리가 폭발하고, 노드 증설·교체 때마다 등록 작업이 따라붙는다
- 본인확인 서비스 도메인 맥락: 시스템 간 통계·정산 API 호출은 "누가 호출했는가"를 매 요청 전자서명(RSASSA-PSS 등)으로 증명하고, 검증 측은 DB 설정 테이블에 등록된 공개키로 검증 + 요청 타임스탬프 유효시간(예: 5분) 제한으로 리플레이를 막는 구조. 공개키 등록도 신원 단위라, 신원이 깔끔해야 등록·폐기(rotation)도 깔끔해진다

## RSA 키쌍 조사 메모

- 비대칭키: 개인키로 서명 → 공개키로 검증. 개인키는 호출자만 보관(chmod 600, 깃 커밋 금지), 공개키는 검증자에게 배포해도 안전
- Java에서 바로 쓰는 표준 인코딩: 개인키는 **PKCS#8** PEM(`PKCS8EncodedKeySpec`), 공개키는 **X.509/SPKI** PEM(`X509EncodedKeySpec`) — openssl `genpkey`가 PKCS#8을, `rsa -pubout`이 SPKI를 내놓는다

  ```bash
  openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out private.pem  # PKCS#8
  openssl rsa -in private.pem -pubout -out public.pem                            # X.509/SPKI
  grep -v "PUBLIC KEY" public.pem | tr -d '\n'   # DB 등록용 한 줄 Base64 = PEM 바디
  ```
- 키 관리 원칙(NIST SP 800-57): 키는 용도·엔티티별로 분리하고, 유출 시 영향 범위가 키의 공유 범위와 같아진다 — "하나의 신원 = 하나의 키쌍"은 이 원칙의 실무 번역

교훈: 키 생성 횟수를 묻는 질문의 진짜 답은 "신원이 몇 개인가"다. 환경 분리는 폭발 반경 때문에, 노드 공유는 신원이 하나이기 때문에 — 둘 다 같은 원칙에서 나온다.

## References

- [NIST SP 800-57 Part 1 Rev.5 — Recommendation for Key Management](https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final)
- [RSA Key Formats (CryptoSys PKI)](https://cryptosys.net/pki/rsakeyformats.html)
- [Best Practices for Public/Private Key Management (Keyfactor)](https://www.keyfactor.com/blog/best-practices-for-public-key-vs-private-key-management/)
