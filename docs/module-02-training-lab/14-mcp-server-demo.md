# Module 2, Exercise 14 — Sentinel MCP Server Demo Prompts

**SC-200 domain:** Perform threat hunting (20–25%) — the Sentinel MCP Server is explicitly named as new outline material; also touches Manage a security operations environment via natural-language Advanced Hunting access.
**Prerequisites:** Exercises 1–4, 6–10 complete (Exercise 5 skipped). Data Lake not enabled (regional capacity unavailable — see `11-17-datalake-blocked.md`), so only the Triage-collection prompts, which don't depend on the data lake, were attempted.

---

## Summary

Connected VS Code to the Sentinel MCP server's **Triage** collection (Defender XDR API access, no data lake dependency) and ran the collection's demo prompts. One prompt produced a genuine, useful investigative result. Two executed correctly at the tool-calling level but returned no data — not the Data Lake gap, but a *separate*, already-known limitation: they depend on Microsoft Defender for Endpoint device telemetry, the same gap Exercise 5 already documented. Custom tool creation was unavailable in the portal.

## Correction to my own reasoning

Prompts were filtered only by MCP tool collection (Triage vs. Data Exploration) to dodge the Data Lake block — not cross-checked against the *separate* MDE device-data gap from Exercise 5. Prompts 7 and 8 both depend on Defender-for-Endpoint-sourced tables (`DeviceInfo`, `DeviceNetworkEvents`, `DeviceLogonEvents`, etc.), populated only by MDE-onboarded devices, which this project has never had. Result: both executed without error — real queries ran against real tables — but returned zero rows, since the underlying tables are genuinely empty in this tenant. Worth stating plainly: avoiding one known dependency doesn't guarantee avoiding all of them; this project now has two independent hard gaps (Data Lake, MDE), and either can silently block something that looks unrelated to it on the surface.

## Findings by prompt

**Prompt 1 — Incident Triage & Prioritization — genuinely successful.**
Correctly enumerated real incidents by severity (no Critical active, several High/Medium) and identified the newest High-severity incident — `Console login without MFA by backdoor-svc — Critical service` (from Exercise 8's watchlist-enrichment work) — as top triage priority. Reasoning cited recency, the suspicious `backdoor-svc` account name, absence of MFA as an identity-control failure, and the incident's four-tactic MITRE mapping (Initial Access, Persistence, Privilege Escalation, Defense Evasion) as evidence of active multi-stage compromise. Produced a full triage-order recommendation across all active incidents, correctly deprioritizing older multi-tactic endpoint incidents beneath the fresher identity-compromise pattern.

**Prompt 7 — Device & Lateral Movement — executed correctly, no data (MDE gap, not a bug).**
Searched `DeviceInfo`, `DeviceLogonEvents`, `DeviceNetworkInfo`, `DeviceNetworkEvents`, and vulnerability tables for the target IP — zero matches. Correctly concluded no MDE device record exists in this environment and explained why, citing this project's own Exercise 5 documentation, rather than inventing a plausible-looking result.

**Prompt 8 — Advanced Hunting via natural language — executed correctly, no data (MDE gap, not a bug).**
Self-discovered available hunting tables, identified `DeviceNetworkEvents` as relevant, ran the query against the real schema, and confirmed via a direct row count (`TotalEvents: 0`) that the table is empty for this tenant. Reported no matching devices rather than fabricating a result table.

**Prompt 11 — Custom MCP Tool creation — unavailable.**
"Save as tool" wasn't offered in the portal. Not independently confirmed, but the most consistent explanation with everything else observed this session: Microsoft likely gates the entire MCP custom-tool-creation surface behind Data Lake onboarding status at the platform level, even though the specific query involved only touches Analytics-tier data that doesn't itself require the lake.

## Technical Know-How

> **A prompt avoiding one known dependency doesn't guarantee it avoids all of them.** This project has two independent hard environment gaps — Data Lake (regional capacity) and MDE (no onboarded devices). Filtering MCP prompts by tool collection addressed the first; auditing each prompt's actual underlying tables against *all* known gaps, not just the one currently top of mind, would have caught the second before running them.

> **A well-grounded AI agent verifying against real data and reporting an honest "no results found" is a meaningfully more trustworthy behavior than confidently fabricating a plausible answer.** Both blocked prompts here checked live tables, confirmed emptiness with an explicit count, and said so — worth valuing this explicitly when evaluating AI-assisted SOC tooling; the right failure mode is "I checked and there's nothing," not a hallucinated device profile.

## Key Learnings

- Cross-check a task's *every* data dependency against known environment gaps, not just the first one identified.
- MCP-based AI tooling against real security data is genuinely useful when it behaves conservatively — verifying before answering, reporting absence honestly — independent of whether a given task happens to succeed in a specific environment.
- Not every "it didn't work" is the same kind of failure — distinguishing "tool malfunctioned" from "tool worked correctly, data genuinely absent" matters for honest documentation.

## Screenshots referenced

None — this exercise's output is long-form chat text, not something meaningfully captured as a screenshot; documented as prose above instead, consistent with how this project has always described KQL results rather than dumping raw output.

---

**Deliverable status:** Partially complete. 1 of 4 attempted prompts (#1) produced a full result; #7 and #8 confirmed correctly-functioning tooling against the pre-existing MDE gap; #11 unavailable, likely Data-Lake-gated. Prompts #2, 3, 4, 6, 9, 10 (Data Exploration collection) not attempted — see `11-17-datalake-blocked.md`.