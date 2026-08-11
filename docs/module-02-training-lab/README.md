# Module 2 — Sentinel Training Lab

Guided learning module — not portfolio material. Deploys `Azure/Azure-Sentinel`'s [Training Lab](https://github.com/Azure/Azure-Sentinel/tree/master/Tools/Microsoft-Sentinel-Training-Lab) and works through its exercises to see real incident mechanics (correlation, entity graphs, playbooks) before building original detections from Module 3 onward.

| Doc | Status | SC-200 domain(s) |
|---|---|---|
| [00 — Prerequisites & Onboarding](./00-prerequisites-onboarding.md) | ✅ Complete | Manage a security operations environment · Respond to security incidents |
| [01 — Exploration: Hunting Across Your Data](./01-exploration-hunting.md) | ✅ Complete | Perform threat hunting |
| [02 — Threat Intelligence: Microsoft Defender Threat Intelligence](./02-threat-intelligence-mdti.md) | ✅ Complete | Manage a security operations environment · Perform threat hunting |
| [03 — MITRE ATT&CK Coverage](./03-mitre-attack-coverage.md) | ✅ Complete | Manage a security operations environment |
| 04 — Automation Rules | ⬜ Not started | |
| 05 — Cross-Platform Response Actions (Device Isolation) | ⬜ Not started | |
| 06 — Port Scan Detection & Threshold Tuning | ⬜ Not started | |
| 07 — Okta MFA Factor Manipulation | ⬜ Not started | |
| 08 — Watchlist Integration | ⬜ Not started | |
| 09 — Cost Management & Ingestion Analysis | ⬜ Not started | |
| 10 — Table Management: Tiers & Retention | ⬜ Not started | |
| 11 — Data Lake KQL Jobs | ⬜ Not started | |
| 12 — Data Lake vs Real-Time Detection | ⬜ Not started | |
| 13 — Data Lake Notebooks | ⬜ Not started | |
| 14 — Sentinel MCP Server Demo Prompts | ⬜ Not started | |
| 15 — Data Federation with ADLS Gen2 | ⬜ Not started | |
| 16 — Data Transformation: Split Ingestion by Tier | ⬜ Not started | |
| 17 — Custom Graph: Cross-Source Attack Chain | ⬜ Not started | |

**Known open items, carried forward:**
- `S8` and `E7` detection rules (from the packaged deployment) depend on Data Lake KQL jobs — deferred to Exercise 11 / Module 12, not yet resolved.
- `CrowdStrikeCases` telemetry table failed ingestion due to an upstream invalid-stream bug in the packaged template — unresolved, low impact (1 of 21 tables).
- The native MDTI data connector requires business verification (`verification.microsoft.com`) not achievable on this subscription — structural, permanent limitation, not revisited. Exercise 2 completed via a manually-seeded TI indicator instead.
