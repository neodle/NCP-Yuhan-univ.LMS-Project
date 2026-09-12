# 진행 일지

2025.07.29 제안 발표부터 2025.08.21 최종 발표까지의 기록입니다.

---

## 07.29 (화) — 사업 제안 발표

나라장터(KONEPS) 공고를 조사·분석하고 네 가지 선정 기준(목적 적합성, 실현 가능성, 예산 적정성, 안내 내용의 적절성)에 따라
**유한대학교 학습관리시스템(e-Class) 유지보수**를 최종 과제로 선정했습니다.
제안요청서의 인프라 현황과 요구 사양을 분석해 아키텍처 초안(v1)과 예상 비용을 발표했습니다.

---

## 08.01 (금) — NCP 인프라 생성

- VPC `lms-vpc`, `lms-web-subnet`(KR-1) 생성
- 서버 `web-server` 생성 — KVM · G3 세대 · `c2-g3a`(2vCPU / 4GB) · 서버 이미지 `rocky-9.6-base`
- 비공인 IP `10.0.1.6`, 공인 IP `101.79.10.254` 할당
- ACG(네트워크 접근 제어) 규칙 설정

![NCP 서버 콘솔](images/infra/ncp-server-console.png)

---

## 08.04 (월) — SSH 접속 환경 구성

Xshell로 `101.79.10.254:22`에 접속해 서버 운영 환경을 구성했습니다.
인증서(pem) 경로 문제와 호스트 이름 해석 오류를 잡는 과정에서 NCP 서버의 접근 제어 구조를 파악했습니다.

![SSH 접속](images/infra/ssh-web-server.png)

---

## 08.05 (화) — 화면 개발 착수

VS Code Remote-SSH로 서버에 직접 연결해 LMS 화면을 구현하기 시작했습니다.
로그인(`index.html`), 회원가입(`signup.html`), 비밀번호 찾기(`findpw.html`), 메인(`main.html`),
강의실(`LS.html`) 등 기본 골격을 잡았습니다.

![소스 구조](images/screens/source-tree.png)

---

## 08.06 (수) — 동영상 학습 페이지

`studyFS.html`, `studyDeepCV.html`을 구현했습니다.
좌측에 차시 목록과 학습 상태(학습완료 / 학습중 / 다운로드 완료)를 배치하고,
우측 플레이어 상단에 **학습시간 카운터**를 두어 시청 시간이 실시간으로 누적되도록 했습니다.

![동영상 학습](images/screens/study-vod.png)

---

## 08.07 (목) — Cloud DB 생성 및 실시간 강의

- **Cloud DB for MySQL** 생성 — `lms-name` / `lms-mysql-001` · G3 High Memory(2vCPU / 16GB) · MySQL 8.0.42 · `lms-db-subnet`(Private)
- OBS Studio로 실시간 방송을 송출하고, LMS의 `[Live] DIP-week 강의` 항목에서 수신하도록 연동

라이브 강의 항목은 목록에 `LIVE 방송 중` 상태로 표시되며, 녹화 영상 차시와 동일한 UI에서 재생됩니다.

![실시간 강의](images/screens/live-lecture.png)

---

## 08.08 (금) — 교육 과정 및 수료증

`curriculum.html`에 수료 대상 교육 목록을 구성하고, 수료 조건을 충족한 과정에 대해
`certificate.html`에서 증서를 출력하도록 구현했습니다.
소속 · 학번 · 이름 · 교육과정 · 발급일이 포함된 수료증이 별도 창으로 열립니다.

![수료증](images/screens/certificate.png)

---

## 08.11 (월) — 벤치마킹 및 학습 포트폴리오

실제 운영 중인 대학 LMS의 학습 포트폴리오 화면을 참고해
개인 정보 · 자기소개 · 활동 내역 · 학습 목표 네 영역으로 구성한 `portfolio.html`을 구현했습니다.

![학습 포트폴리오](images/screens/portfolio.png)

---

## 08.12 (화) — DB 고가용성 확인

Cloud DB for MySQL이 `Master`(`lms-mysql-001`)와 `Standby Master`(`lms-mysql-002`)로
이중화되어 있음을 콘솔에서 확인했습니다. 고가용성 `Y`, 백업 보관 1일(백업시간 06:00) 설정입니다.

![Cloud DB HA](images/infra/cloud-db-mysql-ha.png)

---

## 08.13 (수) — WAS ↔ DB 연동

`was-server`에서 Private Domain(`db-****.vpc-cdb.ntruss.com`, 포트 3306)으로 MySQL에 접속했습니다.
초기에 `ERROR 1045 (28000): Access denied` 가 발생했는데, DB 계정의 접속 허용 호스트를
WAS 서버의 사설 IP 대역으로 지정하지 않은 것이 원인이었습니다.

![MySQL 접속](images/infra/mysql-private-domain-login.png)

---

## 08.14 (목) — 회원가입 연동

회원가입 화면에서 입력한 값이 `users` 테이블(`id`, `username`, `full_name`, `email`, `phone`, `created_at`)에
적재되는지 검증했습니다. 실제 가입 데이터가 순차적으로 쌓이는 것을 확인했습니다.

---

## 08.18 (월) — VOD 파이프라인 완성

- 강의 영상 원본을 **Object Storage**에 업로드
- **VOD Station**으로 1080p / 720p / 360p / Audio 트랜스코딩 (MP4 · H.264 · AAC · Thumbnail)
- **Global Edge(CDN)** 도메인으로 HLS(`index.m3u8`) 배포
- 배포된 URL을 `videos` 테이블에 등록하고, `video_progress` 테이블 설계

![Global Edge HLS](images/infra/global-edge-hls-urls.png)

![진도율 집계](images/screens/progress-sql.png)

---

## 08.19 (화) — Load Balancer 적용

Load Balancer(`lms-lb-107699287-8e09815088dd.kr.lb.naverncp.com`)를 통해 동일한 서비스가
제공되는 것을 확인했습니다. 이후 접속 경로는 공인 IP 직접 접근 대신 LB 도메인을 기준으로 삼았습니다.

![LB 접속](images/screens/study-via-loadbalancer.png)

---

## 08.20 (수) — 진도율 집계 완성

사용자별 · 영상별 진도율을 집계하는 쿼리를 완성했습니다.
`last_position_sec`(최종 재생 위치)와 `max_position_sec`(최대 도달 위치)를 함께 보관해
스킵 여부를 판별할 수 있게 했고, 여기에서 `progress_pct`와 `completed`를 산출합니다.

![영상별 진도율](images/screens/progress-per-video.png)

![진도율 요약](images/screens/progress-summary.png)

---

## 08.21 (목) — 최종 발표

프로젝트 선정 배경, 아키텍처 5단계 변화 과정, 일자별 구현 내역, 어려웠던 점과 향후 계획을 정리해 발표했습니다.
향후 계획으로는 **프로젝트 저장 및 정리 / GitHub 기록 남기기**를 제시했으며, 이 저장소가 그 결과물입니다.
