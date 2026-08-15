# Module 2 — Sentinel Training Lab

Guided learning module. Deploys `Azure/Azure-Sentinel`'s [Training Lab](https://github.com/Azure/Azure-Sentinel/tree/master/Tools/Microsoft-Sentinel-Training-Lab) and works through its exercises to see real incident mechanics (correlation, entity graphs, playbooks) before building original detections from Module 3 onward.

## Final status
 
| Doc | Status | SC-200 domain(s) |
|---|---|---|
| [00 — Prerequisites & Onboarding](./00-prerequisites-onboarding.md) | ✅ Complete | Manage a security operations environment · Respond to security incidents |
| [01 — Exploration: Hunting Across Your Data](./01-exploration-hunting.md) | ✅ Complete | Perform threat hunting |
| [02 — Threat Intelligence: Microsoft Defender Threat Intelligence](./02-threat-intelligence-mdti.md) | ✅ Complete (adapted) | Manage a security operations environment · Perform threat hunting |
| [03 — MITRE ATT&CK Coverage](./03-mitre-attack-coverage.md) | ✅ Complete | Manage a security operations environment |
| [04 — Automation Rules](./04-automation-rules.md) | ✅ Complete | Manage a security operations environment |
| [05 — Cross-Platform Response Actions (Device Isolation)](./05-device-isolation-response.md) | ⏭️ Skipped — requires MDE | Respond to security incidents |
| [06 — Port Scan Detection & Threshold Tuning](./06-port-scan-threshold-tuning.md) | ✅ Complete | Manage a security operations environment |
| [07 — Okta MFA Factor Manipulation](./07-okta-mfa-manipulation.md) | ✅ Complete | Perform threat hunting · Manage a security operations environment |
| [08 — Watchlist Integration](./08-watchlist-integration.md) | ✅ Complete | Manage a security operations environment |
| [09 — Cost Management & Ingestion Analysis](./09-cost-management.md) | ✅ Complete | Manage a security operations environment |
| [10 — Table Management: Tiers & Retention](./10-table-management.md) | 🟡 Complete (Steps 4–5 deferred) | Manage a security operations environment |
| [11–17 — Data Lake dependent exercises](./11-17-datalake-blocked.md) | ⛔ Blocked — Data Lake unavailable (regional capacity) | Perform threat hunting |
| [14 — Sentinel MCP Server Demo Prompts](./14-mcp-server-demo.md) | 🟡 Partially complete (Triage collection only) | Perform threat hunting |
 
**9 of 17 exercises fully complete, 1 adapted, 1 partial, 1 partially complete, 1 skipped by design, 6 blocked by an external platform limitation.**
 
## What happened, honestly
 
This module took substantially longer than planned — Prerequisites and Onboarding alone absorbed most of that budget fighting a chain of genuine infrastructure issues (Automation Accounts unavailable to Student subscriptions in any region, stacked subscription-level region policies, Cloud Shell's ephemeral storage and restricted token broker, an Az PowerShell breaking change, RBAC data-plane/control-plane confusion) before any lab content could even be attempted. From there, most exercises surfaced at least one real, unplanned issue — stale detection-rule time windows against static telemetry, incomplete reference data, bugs in the packaged content itself, and finally a hard regional capacity wall on Sentinel Data Lake that blocked six of the seven remaining exercises outright.
 
None of this reflects poorly on the underlying skills being built — if anything, the volume of real troubleshooting required is itself strong evidence of hands-on Azure engineering ability, arguably more representative of real SOC engineering work than a clean, friction-free walkthrough would have been. But it's worth being direct about the time cost: this single module likely consumed multiple weeks' worth of the project's planned 1–2h/day budget. Worth factoring into pacing expectations for the modules ahead.
 
## Known open items
 
- **Data Lake regional capacity** — re-check before Module 12 begins (see `11-17-datalake-blocked.md` for full detail). If resolved by then, Exercises 11–17 and detection rules `S8`/`E7` become completable; Module 12 proceeds as originally scoped either way, with fuller hands-on content if the capacity issue has cleared.
- **`CrowdStrikeCases`** telemetry table failed ingestion — upstream invalid-stream bug in the packaged template, unresolved, low impact (1 of 21 tables).
- **MDTI connector** requires business verification not achievable on this subscription — permanent, not revisited. Exercise 2 completed via a manually-seeded TI indicator instead.
- **All 20 deployed detection rules** use short, hardcoded `ago()` lookback windows against telemetry whose `TimeGenerated` was fixed at ingestion (6 August, refreshed once more on 12 August for Exercise 7). By now, none will fire via their normal schedule or a manual Run without either re-ingestion or individual query edits — validate via widened-window test queries if any future work depends on live incident creation from these specific rules.
- **Exercise 10, Steps 4–5** (Analytics ↔ Data lake tier switch demo) deferred alongside the Data Lake block above.
- **`Lab Stage E5`** deploys successfully but is permanently inert — its query depends on MDE device data this project doesn't have, consistent with Exercise 5 and Exercise 14's Prompts 7/8.
## Next
 
Module 2's hands-on work is complete to the extent this environment allows. Proceeding to **Module 3 — Workspace, Roles & Retention**, the start of the portfolio-track P3 build (`log-soc-chn-01`, persistent through Week 12).
