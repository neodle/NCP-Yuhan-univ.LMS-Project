<p align="right"><a href="architecture.md">English</a> · <b>한국어</b></p>

# 아키텍처 변화 과정

제안 발표(2025.07.29)에서 최종 발표(2025.08.21)까지 아키텍처는 다섯 번 개정되었습니다.
각 단계에서 **무엇을 더했고 무엇을 덜어냈는지**, 그리고 그 판단의 이유를 정리합니다.

---

## v1 — 제안요청서 1:1 매핑

![v1](images/architecture/01-architecture-v1.png)

제안요청서에 적힌 요구사항을 NCP 서비스로 그대로 옮긴 첫 설계입니다.

| 요구사항 | 매핑한 NCP 서비스 |
| --- | --- |
| 보안 시스템 (IPS, 방화벽, WAF) | Security Monitoring, Anti-DDoS, IDS, IPS, WAF |
| CDN (무제한 동영상 전송·변환) | CDN+, Object Storage |
| 운영 WEB/WAS | Load Balancer + Web Server |
| DB | MSSQL |
| 학습 로그 분석 | Data Forest |
| 저장소 | NAS, Object Storage |
| 학사행정 연계 | IPsec VPN |
| 동영상 변환 | Cloud Functions, VOD Station |

**한계** — 요구 항목은 빠짐없이 담았지만, 서비스 수가 많아 3주 안에 구축·검증하기 어렵고
고정 비용도 과다했습니다.

---

## v2 — 업로드 경로 명시와 Web 이중화

![v2](images/architecture/02-architecture-v2.png)

**변경점**

- 관리자 업로드 경로를 명시: `Source File → Object Storage → Cloud Functions → VOD Station → Object Storage`
- Web Server를 2대로 이중화하여 단일 장애점 제거
- VOD Station 트랜스코딩 산출물(1080p / 720p / 360p / Audio)을 다시 Object Storage에 적재하는 흐름 확정

**의도** — "영상은 어떻게 들어와서 어떻게 나가는가"를 그림 위에서 끊김 없이 따라갈 수 있게 만드는 것이 목표였습니다.

---

## v3 — 업로드 서버 분리, DBMS를 MySQL로

![v3](images/architecture/03-architecture-v3.png)

**변경점**

- 영상 업로드 전용 서버군(Upload Servers)을 별도 영역으로 분리
- **DBMS를 MSSQL에서 MySQL로 변경**
- Web Server를 6대 규모로 확장 표기

**의도** — 제안요청서의 인프라 현황상 DBMS가 MySQL이었고, 상용 라이선스 비용을 계약상대자가 부담해야 하는
조건이었기 때문에 MySQL이 요구사항과 비용 양쪽에서 타당했습니다.
또한 대용량 업로드 트래픽이 서비스용 Web Server와 자원을 다투지 않도록 경로를 분리했습니다.

---

## v4 — 3-Tier 서브넷과 Auto Scaling

![v4](images/architecture/04-architecture-v4.png)

**변경점**

- VPC 내부를 `Web Subnet` / `WAS Subnet` / `DB Subnet`으로 분리
- Web·WAS에 Auto Scaling 적용
- `DEV Subnet` 추가 (제안요청서의 개발서버 1식 요구 반영)
- Cloud DB for MySQL 이중화, Cloud DB for Cache 추가
- 학사행정 시스템 연계를 위한 IPsec VPN 이중 배치

**의도** — 보안 등급이 다른 계층을 네트워크 수준에서 격리하고, 학기 초 수강신청·과제 마감처럼
트래픽이 몰리는 구간을 Auto Scaling으로 흡수하는 구조를 목표로 했습니다.

---

## v5 — 최종: 구축 가능한 최소 구성

![v5](images/architecture/05-architecture-final.png)

**덜어낸 요소**

| 제외 항목 | 이유 |
| --- | --- |
| Auto Scaling | 부하 테스트까지 포함하면 기간 내 검증 불가 |
| WAF · Anti-DDoS | 비용 대비 프로토타입 단계에서의 검증 가치가 낮음 |
| DEV Subnet | 개발/운영 분리는 프로토타입 단계에서 불필요 |
| IPsec VPN (학사행정) | 연계 대상 시스템이 실재하지 않음 |
| Cloud Functions | VOD Station 수동 트리거로 대체 |

**남긴 핵심 축**

1. **3-Tier VPC** — Web(public) / WAS(private) / DB(private) 격리
2. **Load Balancer** — 단일 진입점 및 확장 여지 확보
3. **Cloud DB for MySQL (HA)** — Master / Standby Master 이중화
4. **VOD 파이프라인** — Object Storage → VOD Station → Global Edge(HLS)
5. **Security Monitoring + IPS** — 최소 보안 계층

프로토타입에서 **빼면 프로젝트의 정체성이 사라지는 것**만 남기는 것이 이 단계의 기준이었습니다.
동영상 스트리밍을 VPC 밖으로 분리한 결정은 끝까지 유지했는데, LMS 트래픽의 대부분이
영상이라는 점에서 이 구조가 곧 서비스 품질을 좌우하기 때문입니다.
