<p align="right"><b>English</b> · <a href="rfp-summary.ko.md">한국어</a></p>

# RFP summary and cost basis

Source: *"Yuhan University 2025 LMS – eClass Maintenance and Cloud Operations"* request for proposal, published on KONEPS.

---

## 1. Contract overview

| Item | Detail |
| --- | --- |
| Title | Yuhan University Learning System (e-Class) maintenance |
| Term | March 2025 – February 2026 (about one year) |
| Award method | Open competitive bidding (lump sum) |
| Payment | Monthly report submitted and approved, then tax invoice issued |

## 2. Why the contract exists

1. Sustain teaching quality and operational flexibility as online education grows in importance
2. Provide an environment for analysing and monitoring learning outcomes, evaluation, feedback loops and education-policy research
3. Build a system that supports systematic educational innovation and learner analytics
4. Raise user satisfaction through stable, professionally managed LMS operations

---

## 3. Existing infrastructure

![Existing infrastructure](images/infra/rfp-infra-stack.png)

| Category | Product | Note |
| --- | --- | --- |
| DBMS | MySQL | |
| WEB / WAS | Apache | |
| LMS platform | COURSEMOS LMS | |
| Storage | NCP | Unlimited |
| CDN | NCP | Unlimited |
| IBT | Coursemos IBT | |
| Certificates | PKI solution | |
| Remote support | ezhelp | |
| DB redundancy | HA solution | |

## 4. Existing infrastructure diagram

![Infrastructure diagram](images/infra/rfp-infra-diagram.png)

The LMS zone and the CDN zone are joined by VPC peering. Operator PCs reach the system only through a
security gateway, and the on-campus academic-affairs system is linked over VPN. 24×365 security monitoring is assumed.

---

## 5. Required cloud services

![Required cloud services](images/infra/rfp-cloud-requirements.png)

| Category | Item | CPU (vcore) | MEM (GB) | DISK (GB) | Qty |
| --- | --- | --- | --- | --- | --- |
| Production | WEB/WAS | 4 | 8 | 50 | 1 |
| Production | WEB/WAS | 8 | 16 | 50 | 2 |
| Production | DB | 8 | 16 | 500 | 1 |
| Cache | — | 2 | 8 | 50 | 1 |
| Development | — | 2 | 4 | 200 | 1 |
| **Subtotal** | | | | | **6** |

Other requirements

- **System software** — OS (Linux) ×6, web server and its technical support, DBMS and its technical support
  (any commercial software licence is borne entirely by the contractor)
- **CDN** — VOD CDN capable of delivering and transcoding video for LMS users (unlimited)
- **Security** — intrusion prevention system (IPS), firewall, web application firewall (WAF)
- **Operations** — load balancer (1 month, 720 hours), integrated operations management
- **Storage** — NAS storage 5,000 GB, snapshot backup, Object Storage 2,000 GB

---

## 6. Cost basis

Costs were estimated at the proposal stage against NCP's published price list, on the following basis.

- Contract term: March 2025 – February 2026 (about one year)
- Payment: monthly report submitted and approved, then tax invoice issued
- **VAT** — 10% is charged separately on every paid service (standard NCP policy)
- Usage-based line items (traffic, transcoding volume) vary with actual consumption, so **only fixed costs were estimated**

The estimate covers the production servers (WEB/WAS, DB), cache server, development server, CDN+, NAS,
Object Storage and the load balancer, priced against the NCP rate card current at the time of the proposal.
