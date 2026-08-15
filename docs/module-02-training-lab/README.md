# Module 2 — Sentinel Training Lab

Guided learning module. Deploys `Azure/Azure-Sentinel`'s [Training Lab](https://github.com/Azure/Azure-Sentinel/tree/master/Tools/Microsoft-Sentinel-Training-Lab) and works through its exercises to see real incident mechanics (correlation, entity graphs, playbooks) before building original detections from Module 3 onward.

| Doc | Status | SC-200 domain(s) |
|---|---|---|
| [00 — Prerequisites & Onboarding](./00-prerequisites-onboarding.md) | ✅ Complete | Manage a security operations environment · Respond to security incidents |
| [01 — Exploration: Hunting Across Your Data](./01-exploration-hunting.md) | ✅ Complete | Perform threat hunting |
| [02 — Threat Intelligence: Microsoft Defender Threat Intelligence](./02-threat-intelligence-mdti.md) | ✅ Complete | Manage a security operations environment · Perform threat hunting |
| [03 — MITRE ATT&CK Coverage](./03-mitre-attack-coverage.md) | ✅ Complete | Manage a security operations environment |
| [04 — Automation Rules](./04-automation-rules.md) | ✅ Complete | Manage a security operations environment |
| [05 — Cross-Platform Response Actions (Device Isolation)](./05-device-isolation-response.md) | ⏭️ Skipped — requires MDE | Respond to security incidents |
| [06 — Port Scan Detection & Threshold Tuning](./06-port-scan-threshold-tuning.md) | ✅ Complete | Manage a security operations environment |
| [07 — Okta MFA Factor Manipulation](./07-okta-mfa-manipulation.md) | ✅ Complete | Perform threat hunting · Manage a security operations environment |
| [08 — Watchlist Integration](./08-watchlist-integration.md) | ✅ Complete | Manage a security operations environment |
| [09 — Cost Management & Ingestion Analysis](./09-cost-management.md) | ✅ Complete | Manage a security operations environment |
| [10 — Table Management: Tiers & Retention](./10-table-management.md) | 🟡 Complete (Steps 4–5 deferred) | Manage a security operations environment |
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
- **All 20 deployed detection rules use short, hardcoded `ago()` lookback windows (1–4h) against telemetry whose `TimeGenerated` was fixed at ingestion on 6 August.** As of Exercise 4 (12 August), none can fire via their normal schedule or manual Run — confirmed for `Lab Stage 7`, logic and telemetry both verified sound via a widened test query. Expect this to affect any remaining exercise that assumes a rule can be freshly triggered; validate via widened-window test queries rather than relying on live incident creation going forward, unless the rules are individually re-edited or telemetry is re-ingested.
- Exercise 10, Steps 4–5 (switching `OfficeActivity_CL` between Analytics and Data lake tiers) deferred alongside `S8`/`E7` — same Data Lake dependency. Three items now waiting on Module 12's Data Lake work: two detection rules and one table-tier demo.
 
