# Module 2, Exercises 11–17 — Blocked: Sentinel Data Lake Unavailable

**Status:** Not completed — genuine platform limitation, not a configuration issue.

---

## Root cause

Attempting to enable Sentinel Data Lake (`System → Settings → Microsoft Sentinel → Data lake → Start setup`) returned:

> *"Data lake setup will be available as soon as regional capacity is sufficient."*

Confirmed against Microsoft's own troubleshooting documentation as a known, real scenario:

> *"Symptom: Onboarding doesn't complete in specific regions due to capacity constraints. Resolution: Use an alternate supported region... create a new workspace in a different region (for example, Central US)."*

**Microsoft's own official fix doesn't apply to this subscription.** The same subscription-level Azure Policy already discovered in Onboarding (`Restrict to Switzerland North`) locks this Azure for Students subscription to `switzerlandnorth`, and only `switzerlandnorth`, for every resource. Microsoft's suggested workaround requires moving to a region with data lake capacity (their own example: Central US); this subscription is structurally forbidden from deploying anywhere else. Two independent constraints — Microsoft's regional data lake rollout and this subscription's hard region lock — combine into a genuine dead end, unlike every prior blocker in this module, none of which had a client-side workaround.

**This differs in kind from the MDTI and MDE blockers.** Those are permanent — tied to account/license type, unresolvable without a fundamentally different subscription. This one is temporary: Microsoft's own GA announcement states data lake regional availability is being *"expanded... progressively over the coming weeks."* Genuinely possible this resolves before Module 12 (this project's dedicated Data Lake phase); no way to predict a timeline. **Plan: re-check data lake availability once, deliberately, immediately before Module 12 begins** — not worth repeated checking before then, since this is Microsoft backend provisioning, not something that changes hour to hour.

## What each exercise would have covered

Documented for exam-study purposes even without hands-on completion, since this material remains examinable (Sentinel Graph, hunting graphs, KQL jobs in the Data Lake, and Notebooks are explicitly named as new content in the current SC-200 outline).

**Exercise 11 — Data Lake KQL Jobs.** Create a scheduled KQL job that promotes aggregated data from the data lake tier to the analytics tier — the core "data promotion" pattern the outline names explicitly. Would have built the `PaloAlto_ThreatSummary_KQL_CL` table that detection rules `S8` and `E7` (deferred since Onboarding) depend on.

**Exercise 12 — Data Lake vs Real-Time Detection.** Directly compares real-time syslog detection (`CommonSecurityLog`, seconds-latency) against data-lake-aggregated detection (`PaloAlto_ThreatSummary_KQL_CL`, hours-latency batch), teaching when each approach is appropriate. Requires Exercise 11 complete first — transitively blocked.

**Exercise 13 — Data Lake Notebooks.** Interactive Jupyter-based investigation against the data lake via VS Code and the Sentinel Spark engine, using a pre-built notebook analyzing the same Palo Alto telemetry from earlier exercises. Notably also requires the data lake's managed identity to hold **Log Analytics Contributor** to write custom tables back — an additional RBAC layer worth knowing about even without completing this hands-on.

**Exercise 15 — Data Federation with ADLS Gen2.** Queries external Azure Data Lake Storage Gen2 data alongside native Sentinel tables *without ingesting it* — useful for large historical datasets or business-context enrichment (HR records, asset inventories) that don't warrant full ingestion.

**Exercise 16 — Data Transformation: Split Ingestion by Tier.** Split transformation rules that route high-value events to the (expensive, fast) Analytics tier while sending lower-priority events straight to the (cheap, slower) Data lake tier via KQL-defined conditions — a cost-optimization pattern directly extending Exercise 9 and Exercise 10's retention/tiering work.

**Exercise 17 — Custom Graph: Cross-Source Attack Chain.** Build a custom Sentinel graph (6 node types, 8 edge types) correlating entities across all six of this project's data sources — the most advanced exercise in the lab, going beyond the built-in XDR graph to model relationships specific to this environment.

**Also affected — Exercise 14's Data Exploration prompts** (Prompts 2, 3, 4, 6, 9, 10 — kill chain diagram, cross-source hunting, user entity analysis, entity relationship graph, alert heatmap, data source health — all using the `query_lake` tool). Documented separately in `14-mcp-server-demo.md`, alongside its Triage-collection prompts, which don't depend on the data lake and were completed.

## Key Learnings

- A regional-capacity limitation is a fundamentally different kind of blocker than a licensing or business-verification gate — worth distinguishing "temporary, actively being resolved by the vendor" from "permanent, tied to account type" when deciding whether to keep checking back.
- Microsoft's own documented fixes aren't universal — always check whether a suggested workaround (here: change region) is actually compatible with this specific subscription's other constraints before assuming it applies.
- A single upstream platform gap can silently block a large fraction of otherwise-unrelated-looking content — six of this module's seven remaining exercises, plus half of an eighth, all trace to this one root cause.

---

**Deliverable status:** Blocked, documented, re-check scheduled before Module 12.