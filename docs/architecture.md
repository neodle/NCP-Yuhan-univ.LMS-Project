<p align="right"><b>English</b> · <a href="architecture.ko.md">한국어</a></p>

# How the architecture evolved

Between the proposal (2025.07.29) and the final presentation (2025.08.21) the architecture was revised five times.
This document records **what each revision added and removed**, and why.

---

## v1 — A 1:1 mapping of the RFP

![v1](images/architecture/01-architecture-v1.png)

The first design simply translated every requirement in the RFP into an NCP service.

| Requirement | NCP service |
| --- | --- |
| Security systems (IPS, firewall, WAF) | Security Monitoring, Anti-DDoS, IDS, IPS, WAF |
| CDN (unlimited video delivery and transcoding) | CDN+, Object Storage |
| Production WEB/WAS | Load Balancer + Web Server |
| Database | MSSQL |
| Learning-log analytics | Data Forest |
| Storage | NAS, Object Storage |
| Academic-affairs integration | IPsec VPN |
| Video transcoding | Cloud Functions, VOD Station |

**Limitation** — nothing was missing, but the sheer number of services made it impossible to build and verify
in three weeks, and the fixed cost was excessive.

---

## v2 — An explicit upload path and a duplicated web tier

![v2](images/architecture/02-architecture-v2.png)

**Changes**

- Made the administrator upload path explicit: `Source File → Object Storage → Cloud Functions → VOD Station → Object Storage`
- Duplicated the web server to remove a single point of failure
- Fixed the flow where VOD Station transcoding output (1080p / 720p / 360p / Audio) lands back in Object Storage

**Intent** — you should be able to trace "how video comes in and how it goes out" on the diagram without a break.

---

## v3 — Dedicated upload servers, MySQL instead of MSSQL

![v3](images/architecture/03-architecture-v3.png)

**Changes**

- Moved video uploads onto a dedicated server group (Upload Servers)
- **Switched the DBMS from MSSQL to MySQL**
- Scaled the web tier up to six servers on the diagram

**Intent** — the RFP's own infrastructure inventory lists MySQL, and commercial software licences were to be
borne entirely by the contractor, so MySQL was the right answer on both requirements and cost.
Separating the upload path also keeps bulk upload traffic from competing with the servers that serve users.

---

## v4 — Three-tier subnets and Auto Scaling

![v4](images/architecture/04-architecture-v4.png)

**Changes**

- Split the VPC into `Web Subnet` / `WAS Subnet` / `DB Subnet`
- Applied Auto Scaling to the Web and WAS tiers
- Added a `DEV Subnet` (the RFP requires one development server)
- Duplicated Cloud DB for MySQL and added Cloud DB for Cache
- Placed IPsec VPN on both ends for the academic-affairs integration

**Intent** — isolate tiers with different security postures at the network level, and absorb the traffic
spikes that come with course registration and assignment deadlines through Auto Scaling.

---

## v5 — Final: the minimum that could actually be built

![v5](images/architecture/05-architecture-final.png)

**Removed**

| Dropped | Why |
| --- | --- |
| Auto Scaling | Cannot be verified in the available time once load testing is included |
| WAF · Anti-DDoS | Little verification value at prototype stage relative to cost |
| DEV subnet | Separating dev from production is unnecessary for a prototype |
| IPsec VPN (academic affairs) | No real system on the other end to integrate with |
| Cloud Functions | Replaced by triggering VOD Station manually |

**Kept**

1. **Three-tier VPC** — Web (public) / WAS (private) / DB (private) isolation
2. **Load Balancer** — a single entry point and room to scale later
3. **Cloud DB for MySQL (HA)** — Master / Standby Master
4. **VOD pipeline** — Object Storage → VOD Station → Global Edge (HLS)
5. **Security Monitoring + IPS** — a minimum security layer

The rule for this revision was to keep only what the project would lose its identity without.
Keeping video streaming outside the VPC survived every revision: video is the bulk of LMS traffic,
so that one decision largely determines the quality of the service.
