# 유한대학교 LMS(e-Class) 클라우드 전환 프로젝트

> 나라장터(KONEPS) 공고 **「유한대학교 학습관리시스템(e-Class) 유지보수」** 사업을 과제로 삼아,
> 제안요청서의 요구사항을 **Naver Cloud Platform(NCP)** 위에 직접 재설계·구현한 현장실습 프로젝트입니다.

<p>
  <img alt="Naver Cloud Platform" src="https://img.shields.io/badge/Cloud-Naver%20Cloud%20Platform-03C75A">
  <img alt="Rocky Linux" src="https://img.shields.io/badge/OS-Rocky%20Linux%209.6-10B981">
  <img alt="MySQL" src="https://img.shields.io/badge/DB-Cloud%20DB%20for%20MySQL%208.0.42-4479A1">
  <img alt="VOD" src="https://img.shields.io/badge/Media-VOD%20Station%20%2B%20Global%20Edge-6366F1">
  <img alt="Period" src="https://img.shields.io/badge/Period-2025.07%20~%202025.08-64748B">
</p>

| 항목 | 내용 |
| --- | --- |
| 프로젝트 기간 | 2025.07.29 (제안) ~ 2025.08.21 (최종 발표) |
| 수행 형태 | 클라우드스퀘어(주) 현장실습 개인 프로젝트 |
| 수행자 | 강민수 (건양대학교 인공지능학과) |
| 대상 사업 | 유한대학교 학습관리시스템(e-Class) 유지보수 — 2025.03 ~ 2026.02, 일반경쟁입찰(총액) |
| 사용 클라우드 | Naver Cloud Platform (VPC 환경) |
| 서비스 접점 | `101.79.10.254` (Web 공인 IP) / `lms-lb-107699287-8e09815088dd.kr.lb.naverncp.com` (Load Balancer) |

---

## 목차

1. [프로젝트 배경](#1-프로젝트-배경)
2. [데모 영상](#2-데모-영상)
3. [최종 아키텍처](#3-최종-아키텍처)
4. [아키텍처 변화 과정](#4-아키텍처-변화-과정)
5. [구현 기능](#5-구현-기능)
6. [저장소 구조](#6-저장소-구조)
7. [인프라 구성 상세](#7-인프라-구성-상세)
8. [학습 진도율 설계](#8-학습-진도율-설계)
9. [진행 일지](#9-진행-일지)
10. [한계와 배운 점](#10-한계와-배운-점)
11. [문서](#11-문서)

---

## 1. 프로젝트 배경

대한민국 조달청이 운영하는 국가종합전자조달시스템(**KONEPS, 나라장터**)의 공고 중에서
아래 네 가지 기준으로 실제 진행할 사업을 선정했습니다.

1. 사업 목적 및 방향성과의 적합성
2. 기술적·행정적 실현 가능성
3. 예산의 적정성
4. 사업 안내 내용의 적절성

그 결과 선정한 사업이 **유한대학교 학습관리시스템(e-Class) 유지보수**입니다.

COVID-19를 계기로 전면 도입된 LMS(Learning Management System)는 감염병 상황이 종료된 뒤에도
**학습자 맞춤형 학습 환경**을 구성할 수 있다는 장점 때문에 여전히 대학 운영의 핵심 시스템으로 남아 있습니다.
LMS를 직접 사용하는 학생 입장에서의 문제의식과 호기심이 이 사업을 고른 또 하나의 이유였습니다.

**사업 배경 및 필요성 (제안요청서 기준)**

- 온라인 교육의 중요성 증대에 맞춘 교육의 질·운영 유연성 확보
- 교육성과 분석/모니터링, 평가·환류, 교육정책 연구 및 개발 환경 확보
- 체계적 교육혁신 지원 및 학습자 분석을 지원하는 시스템 마련
- 전문 업체를 통한 안정적·체계적 LMS 운용관리로 사용자 만족도 제고

---

## 2. 데모 영상

실제로 구축한 LMS 서비스의 동작 화면입니다. 원본 mp4는 Release 자산으로 제공합니다.

| 일자 | 내용 | 링크 |
| --- | --- | --- |
| 2025-08-04 | 웹 서버 구축 및 초기 화면 구성 | [▶ 보기](https://github.com/neodle/NCP-Yuhan-univ.LMS-Project/releases/download/v1.0-demo/20250804.mp4) |
| 2025-08-05 | 로그인 · 강의실 · 학습 페이지 이동 | [▶ 보기](https://github.com/neodle/NCP-Yuhan-univ.LMS-Project/releases/download/v1.0-demo/20250805.mp4) |
| 2025-08-06 | 동영상 강의 재생 및 학습 진행 | [▶ 보기](https://github.com/neodle/NCP-Yuhan-univ.LMS-Project/releases/download/v1.0-demo/20250806.mp4) |
| 2025-08-07 | 실시간(LIVE) 강의 송출 데모 | [▶ 보기](https://github.com/neodle/NCP-Yuhan-univ.LMS-Project/releases/download/v1.0-demo/20250807.mp4) |

> 전체 목록은 [Releases](https://github.com/neodle/NCP-Yuhan-univ.LMS-Project/releases) 탭에서 확인할 수 있습니다.

---

## 3. 최종 아키텍처

![최종 아키텍처](docs/images/architecture/05-architecture-final.png)

**구성 요약**

- **VPC - LMS** 내부를 3계층 서브넷으로 분리
  - `Web Subnet (public)` — Web Server
  - `WAS Subnet (private)` — Web Application Server
  - `DB Subnet (private)` — Cloud DB for MySQL, Cloud DB for Cache
- 외부 트래픽은 **Security Monitoring → IPS → Load Balancer** 경로로만 유입
- 강의 영상은 **Source File → VOD Station(1080p/720p/360p/Audio, MP4·H.264·AAC·Thumbnail) → Object Storage → Global Edge(CDN)** 로 분리 전달
- 학습 자료·첨부 파일 보관을 위한 **NAS**

트래픽 경로를 두 갈래로 나눈 것이 핵심입니다. 웹/API 요청은 VPC 내부의 Web·WAS·DB 계층에서 처리하고,
용량이 큰 **동영상 스트리밍은 VPC 밖의 Object Storage + Global Edge**가 담당하도록 하여
Web 서버가 영상 트래픽의 영향을 받지 않게 했습니다.

---

## 4. 아키텍처 변화 과정

설계는 총 5단계에 걸쳐 다듬어졌습니다.

<details>
<summary><b>v1 — 제안 단계 초안 (2025.07.29)</b></summary>

![v1](docs/images/architecture/01-architecture-v1.png)

제안요청서의 요구사항(CDN+, Object Storage, Anti-DDoS/IDS/IPS/WAF, MSSQL, Data Forest, NAS, IPsec VPN)을
그대로 NCP 서비스로 1:1 매핑한 단계입니다. 요구 항목은 모두 담겼지만 비용과 구축 난도가 지나치게 높았습니다.

</details>

<details>
<summary><b>v2 — 업로드 경로 정리</b></summary>

![v2](docs/images/architecture/02-architecture-v2.png)

관리자가 영상 원본을 올리는 경로(Source File → Object Storage → Cloud Functions → VOD Station)를 명시하고,
Web Server를 이중화하여 단일 장애점을 줄였습니다.

</details>

<details>
<summary><b>v3 — 업로드 서버 분리 및 DBMS 변경</b></summary>

![v3](docs/images/architecture/03-architecture-v3.png)

영상 업로드 전용 서버군을 별도 영역으로 분리하고, DBMS를 MSSQL에서 **MySQL**로 변경했습니다.
제안요청서의 인프라 현황상 DBMS가 MySQL이며, 실제 구축 비용도 낮출 수 있었습니다.

</details>

<details>
<summary><b>v4 — 3-Tier 서브넷 + Auto Scaling</b></summary>

![v4](docs/images/architecture/04-architecture-v4.png)

Web / WAS / DB 서브넷으로 계층을 분리하고 Web·WAS에 Auto Scaling을 적용했습니다.
DEV 서브넷과 학사행정 연계를 위한 IPsec VPN도 이 단계에서 추가되었습니다.

</details>

<details open>
<summary><b>v5 — 최종 (실제 구축 대상)</b></summary>

![v5](docs/images/architecture/05-architecture-final.png)

현장실습 기간(약 3주) 안에 실제로 구축 가능한 **최소 구성의 프로토타입**으로 축소했습니다.
Auto Scaling·WAF·DEV 서브넷·VPN 등 검증에 시간이 오래 걸리는 요소를 덜어내고,
**3-Tier VPC + Load Balancer + Cloud DB(HA) + VOD/CDN 파이프라인**이라는 핵심 축만 남겼습니다.

</details>

---

## 5. 구현 기능

| 화면 | 파일 | 설명 |
| --- | --- | --- |
| 로그인 | [`index.html`](index.html) | `/api/login` · `/api/me` 호출, 세션 확인 후 메인으로 이동 |
| 회원가입 / 비밀번호 찾기 | [`signup.html`](signup.html), [`findpw.html`](findpw.html) | `/api/signup`으로 가입 정보 전송 |
| 메인 · 강의실 | [`main.html`](main.html), [`Pages/LS.html`](Pages/LS.html) | 수강 과목 목록 및 강의실 진입 |
| 캘린더 | [`Pages/calendar.html`](Pages/calendar.html) | 학사 일정 표시 |
| 학습 포트폴리오 | [`Pages/portfolio.html`](Pages/portfolio.html) | 개인 정보 · 자기소개 · 활동 내역 · 학습 목표 |
| 기초학습역량 | [`Pages/basiclearn.html`](Pages/basiclearn.html) | 기초학습역량 진단 영역 |
| 교과 · 강의 관리 | [`Pages/curriculum.html`](Pages/curriculum.html), [`Pages/DeepCV.html`](Pages/DeepCV.html), [`Pages/FSlecture.html`](Pages/FSlecture.html) | 교육 과정 및 강의 관리 |
| 학습(동영상) | [`Pages/studyFS.html`](Pages/studyFS.html), [`Pages/studyDeepCV.html`](Pages/studyDeepCV.html) | HLS 재생 · 학습시간 측정 · 진도 저장 |
| 수료증 출력 | [`Pages/certificate.html`](Pages/certificate.html) | 법정의무교육 수료증 발급 |
| 성적 조회 | [`Pages/Gradeinquiry.html`](Pages/Gradeinquiry.html) | 성적 조회 |

<table>
  <tr>
    <td width="50%"><img src="docs/images/screens/login.png" alt="로그인 화면"><br><sub><b>로그인</b> — 유한대학교 e-Class 스타일 진입 화면</sub></td>
    <td width="50%"><img src="docs/images/screens/study-vod.png" alt="동영상 학습"><br><sub><b>동영상 학습</b> — 좌측 차시 목록, 학습완료 상태, 학습시간 카운터</sub></td>
  </tr>
  <tr>
    <td><img src="docs/images/screens/live-lecture.png" alt="실시간 강의"><br><sub><b>실시간 강의</b> — OBS로 송출한 LIVE 방송 수신 (인물 영역 모자이크 처리)</sub></td>
    <td><img src="docs/images/screens/certificate.png" alt="수료증"><br><sub><b>수료증 출력</b> — 수료 조건 충족 시 증서 발급</sub></td>
  </tr>
  <tr>
    <td><img src="docs/images/screens/portfolio.png" alt="학습 포트폴리오"><br><sub><b>학습 포트폴리오</b> — 개인 정보 및 학습 목표 관리</sub></td>
    <td><img src="docs/images/screens/study-via-loadbalancer.png" alt="로드밸런서 접속"><br><sub><b>Load Balancer 접속</b> — LB 도메인으로 동일 서비스 제공</sub></td>
  </tr>
</table>

---

## 6. 저장소 구조

```
.
├── index.html            # 로그인 (진입 화면)
├── signup.html           # 회원가입
├── findpw.html           # 비밀번호 찾기
├── main.html             # 메인 · 수강 과목
├── Pages/                # 로그인 이후 화면
│   ├── LS.html               # 강의실
│   ├── calendar.html         # 캘린더
│   ├── curriculum.html       # 교육 과정
│   ├── certificate.html      # 수료증 출력
│   ├── portfolio.html        # 학습 포트폴리오
│   ├── basiclearn.html       # 기초학습역량
│   ├── Gradeinquiry.html     # 성적 조회
│   ├── DeepCV.html           # 딥러닝 영상처리 강의
│   ├── FSlecture.html        # 소방안전교육 강의
│   ├── studyDeepCV.html      # 딥러닝 영상처리 학습(동영상)
│   └── studyFS.html          # 소방안전교육 학습(동영상)
├── img/                  # 로고 · 배경 · 프로필 이미지
└── docs/                 # 프로젝트 문서 · 다이어그램 · 화면 캡처
```

### 6.1 프론트엔드 ↔ 백엔드 연동

화면은 Web Server가 정적으로 서빙하고, `/api` 이하 요청은 **WAS 서버로 프록시**되는 구조입니다.
(WAS 애플리케이션 코드는 본 저장소에 포함되어 있지 않습니다.)

| 엔드포인트 | 메서드 | 용도 |
| --- | --- | --- |
| `/api/login` | POST | 로그인 |
| `/api/me` | GET | 로그인 세션 확인 |
| `/api/signup` | POST | 회원가입 |
| `/api/video/progress` | GET | 이어보기 위치(`last_pos_sec`) 조회 |
| `/api/video/progress` | POST | 진도 저장(UPSERT) |

동영상은 [hls.js](https://github.com/video-dev/hls.js)로 Global Edge의 HLS 스트림을 재생합니다.

```html
<div class="video-item" data-vid="1001"
     data-src="http://oeeggchm11489.edge.naverncp.com/hls/VC~.../firevideo.mp4/index.m3u8">
```

진도 저장은 재생 중 주기적으로 이뤄지고, **페이지를 떠나는 순간에는 `navigator.sendBeacon`** 으로
마지막 위치를 전송해 브라우저 종료 시 기록이 유실되지 않도록 했습니다.

---

## 7. 인프라 구성 상세

### 7.1 실제 구축 자원

| 구분 | 서비스 | 설정 |
| --- | --- | --- |
| 네트워크 | VPC | `lms-vpc` — Web / WAS / DB 서브넷 분리 |
| Web | Server (KVM, G3) | `web-server` · Rocky Linux 9.6 · c2-g3a (2vCPU / 4GB) · 공인 IP `101.79.10.254` |
| WAS | Server | `was-server` · 애플리케이션 로직 및 DB 연동 |
| DB | Cloud DB for MySQL | `lms-mysql-001`(Master) + `lms-mysql-002`(Standby Master) · G3 High Memory 2vCPU / 16GB · MySQL 8.0.42 · **고가용성 Y** · Private Domain `db-****.vpc-cdb.ntruss.com:3306` |
| 부하 분산 | Load Balancer | `lms-lb-107699287-8e09815088dd.kr.lb.naverncp.com` |
| 미디어 변환 | VOD Station | 1080p / 720p / 360p / Audio · MP4 · H.264 · AAC · Thumbnail |
| 미디어 배포 | Object Storage + Global Edge | HLS(`index.m3u8`) 형태로 CDN 배포 |

![NCP 서버 콘솔](docs/images/infra/ncp-server-console.png)

<sub>NCP 콘솔에서 생성한 `web-server` 인스턴스 상세</sub>

![Cloud DB for MySQL HA](docs/images/infra/cloud-db-mysql-ha.png)

<sub>Master / Standby Master 이중화로 구성한 Cloud DB for MySQL</sub>

![Global Edge HLS](docs/images/infra/global-edge-hls-urls.png)

<sub>VOD Station이 변환한 영상이 Global Edge 도메인의 HLS 스트림으로 DB에 등록된 모습</sub>

### 7.2 제안요청서 요구 사양

<table>
  <tr>
    <td width="50%"><img src="docs/images/infra/rfp-infra-stack.png" alt="인프라 현황"><br><sub>기존 인프라 현황 — MySQL / Apache / COURSEMOS LMS / NCP Storage·CDN(무제한) / IBT / PKI / HA</sub></td>
    <td width="50%"><img src="docs/images/infra/rfp-server-spec.png" alt="서버 요구 사양"><br><sub>요구 서버 사양 — 운영 WEB/WAS 3식, DB 1식, CACHE 1식, 개발 1식 (총 6식)</sub></td>
  </tr>
</table>

전체 요구 항목과 비용 산정 기준은 [`docs/rfp-summary.md`](docs/rfp-summary.md)에 정리했습니다.

---

## 8. 학습 진도율 설계

LMS의 핵심은 **"누가, 어떤 영상을, 어디까지 봤는가"** 를 신뢰할 수 있게 기록하는 것입니다.
단순 재생 완료 플래그가 아니라 **최종 재생 위치**와 **최대 도달 위치**를 함께 저장해
구간 건너뛰기(스킵)를 걸러낼 수 있도록 설계했습니다.

**주요 테이블**

- `videos` — `id`, `title`, `url`(Global Edge HLS), `duration_sec`, `duration_hms`, `created_at`
- `video_progress` — `user_id`, `video_id`, `last_position_sec`, `max_position_sec`, `watched_seconds`, `duration_sec`, `completed`, `completed_at`, `first_started_at`, `last_updated_at`
- `users` — `id`, `username`, `full_name`, `email`, `phone`, `created_at`

**사용자별 총 진도율 집계**

```sql
SELECT user_id,
       COUNT(*)                                 AS videos_touched,
       SUM(completed)                           AS videos_completed,
       ROUND(AVG(GREATEST(last_position_sec, max_position_sec) /
             NULLIF(duration_sec, 0)) * 100, 1) AS overall_progress_pct
FROM   video_progress
WHERE  user_id = 1;
```

![진도율 집계](docs/images/screens/progress-sql.png)

![영상별 진도율](docs/images/screens/progress-per-video.png)

<sub>영상별 시청 위치 · 시청 시간 · 완료 여부가 초 단위로 누적되는 모습</sub>

---

## 9. 진행 일지

| 일자 | 진행 내용 |
| --- | --- |
| 07.29 (화) | 사업 제안 발표 — 사업 선정 근거, 인프라 현황 분석, 아키텍처 초안, 예상 비용 산정 |
| 08.01 (금) | NCP VPC · 서브넷 · ACG 생성, `web-server`(Rocky Linux 9.6) 인스턴스 구축 및 공인 IP 할당 |
| 08.04 (월) | SSH(Xshell) 접속 환경 구성, 웹 서버 기동 |
| 08.05 (화) | VS Code Remote-SSH로 개발 환경 연결, LMS 화면 구조(로그인 · 메인 · 강의실) 구현 |
| 08.06 (수) | 동영상 학습 페이지 구현 — 차시 목록, 플레이어, 학습시간 카운터 |
| 08.07 (목) | Cloud DB for MySQL 생성, OBS Studio로 실시간(LIVE) 강의 송출 연동 |
| 08.08 (금) | 교육 과정 목록 및 수료증 출력 기능 구현 |
| 08.11 (월) | 실제 대학 LMS 화면 벤치마킹, 학습 포트폴리오 화면 구현 |
| 08.12 (화) | Cloud DB for MySQL 고가용성(Master / Standby Master) 구성 확인 |
| 08.13 (수) | WAS 서버에서 Private Domain으로 DB 접속 연동, 계정 · 권한 설정 |
| 08.14 (목) | 회원가입 → `users` 테이블 연동, 사용자 데이터 적재 검증 |
| 08.18 (월) | Object Storage → VOD Station 변환 → Global Edge(HLS) 배포 파이프라인 완성, `video_progress` 설계 |
| 08.19 (화) | Load Balancer 적용 및 LB 도메인 서비스 확인, 진도율 기록 검증 |
| 08.20 (수) | 사용자별 진도율 집계 쿼리 완성, 전체 기능 통합 점검 |
| 08.21 (목) | 최종 발표 |

![SSH 접속](docs/images/infra/ssh-web-server.png)

<sub>Xshell로 NCP `web-server`에 접속한 화면</sub>

![소스 구조](docs/images/screens/source-tree.png)

<sub>VS Code Remote-SSH로 연결한 서버 내 LMS 소스 구조</sub>

---

## 10. 한계와 배운 점

**남긴 것**

- 제안요청서를 읽고 → 아키텍처로 옮기고 → 실제 클라우드 자원으로 구축하는 **전 과정**을 한 번에 경험
- 요구사항을 그대로 구현하는 것과, 주어진 기간·비용 안에서 **무엇을 덜어낼지 판단하는 것**은 다른 문제라는 점을 확인
- 3-Tier VPC 분리, DB 고가용성, VOD/CDN 분리 배포 등 **운영을 염두에 둔 설계**를 직접 구성

**덜어낸 것 (Future Works)**

- Auto Scaling 기반 Web / WAS 탄력적 확장
- WAF · Anti-DDoS 등 보안 계층 전체 적용
- Cloud DB for Cache 연동을 통한 세션 · 조회 성능 개선
- DEV 서브넷 및 IPsec VPN을 통한 학사행정 시스템 연계
- Cloud Functions를 이용한 업로드 → 트랜스코딩 자동화

아키텍처의 모든 요소를 구현하기에는 비용과 시간이 크게 소요되어,
**최소 요소로 구성한 프로토타입**을 우선 완성하는 전략을 택했습니다.

---

## 11. 문서

| 문서 | 내용 |
| --- | --- |
| [`docs/architecture.md`](docs/architecture.md) | 아키텍처 5단계 변화 과정과 각 단계의 설계 의도 |
| [`docs/rfp-summary.md`](docs/rfp-summary.md) | 제안요청서 요약, 요구 인프라 사양, 비용 산정 기준 |
| [`docs/journal.md`](docs/journal.md) | 일자별 상세 진행 일지 |

---

### 고지

- 본 저장소는 나라장터 공고를 **학습 목적으로 재구성**한 개인 프로젝트 기록입니다. 유한대학교 및 클라우드스퀘어(주)의 공식 산출물이 아닙니다.
- 화면에 등장하는 유한대학교 로고 · 디자인은 UI 재현 학습 목적으로만 사용되었습니다.
- 스크린샷 내 개인정보(이메일, 인물 영상)는 마스킹 처리했습니다.

### 작성자

**강민수** — 건양대학교 인공지능학과 · 2025 클라우드스퀘어(주) 현장실습
