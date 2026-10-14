# Manage — Sentinel roles and detection-rule mechanics

**SC-200 domain:** Manage a security operations environment (40–45%)
**Bullets:** Configure the Sentinel SIEM → roles · Configure detections → Sentinel analytics rules · Configure detections → Defender XDR custom detection rules
**Sources:** [Sentinel roles](https://learn.microsoft.com/en-us/azure/sentinel/roles) · [Custom detection rules](https://learn.microsoft.com/en-us/defender-xdr/custom-detection-rules) · [Fusion](https://learn.microsoft.com/en-us/azure/sentinel/fusion) · checked 13.10.2026

---

## 1. Sentinel roles — "to do X you need Y"

| Task | Least-privilege role | Trap |
|---|---|---|
| View data, incidents, workbooks | Microsoft Sentinel **Reader** | — |
| Triage / assign / close incidents | Microsoft Sentinel **Responder** | Reader can't change incidents |
| Create/edit analytics rules, install Content hub solutions | Microsoft Sentinel **Contributor** | Content hub needs Contributor at **RG** scope |
| Run playbooks manually | Microsoft Sentinel **Playbook Operator** | Sentinel Contributor can't run playbooks |
| Create/edit playbooks | **Logic App Contributor** | Not a Sentinel role — playbooks are Logic Apps |
| Let automation rules run playbooks | Sentinel's **service account** gets **Microsoft Sentinel Automation Contributor** on the playbook RG ("Manage playbook permissions"); granting it needs **Owner** | Automation Contributor is **never assigned to users** |
| Create/delete workbooks | Sentinel role **+ Workbook Contributor** | Sentinel Contributor alone is not enough |
| Guest user assigns incidents | Responder **+ Directory Reader** (Entra role) | Members have Directory Reader by default; guests don't |
| Connect a data source | **Write** on the workspace (+ connector-specific perms) | — |
| Manage XDR custom detections on Sentinel data | Sentinel **Contributor** or higher | — |

**Defaults to remember**
- Assign at the **resource group** holding the workspace, so Logic Apps are covered too. If you assign at workspace scope, you must also assign on the `SecurityInsights` solution resource.
- Role assignments are cumulative: Reader + Contributor = Contributor.
- Per-table / per-resource access → **table-level RBAC** or **resource-context RBAC**, not a custom Sentinel role.

**Typical personas**

| Persona | Roles (RG scope) |
|---|---|
| SOC analyst | Responder + Playbook Operator |
| Security engineer | Contributor + Logic App Contributor |
| Automation SP | Contributor |

**URBAC vs Azure RBAC**
- Unified RBAC (Defender portal → Permissions) is becoming the default for Defender **and** Sentinel portal experiences (rollout late Sep–Dec 2026). Sentinel Reader, Responder and Contributor are auto-imported.
- Logic Apps / playbooks are still Azure resources → **Azure RBAC**.
- Data lake: tenant-wide read = Entra roles (Global Reader, Security Reader/Operator/Admin). Creating or managing data lake **jobs** = Security Operator / Security Admin / Global Admin. Per-workspace or system-table scoping = custom URBAC role.

---

## 2. Sentinel analytics rule types

| Type | You write the KQL | Entity mapping | Frequency you set | Notes |
|---|---|---|---|---|
| Scheduled | ✅ | ✅ | ✅ | Workhorse; full alert enrichment |
| NRT | ✅ | ✅ | ❌ (runs ~every minute) | Playbooks only via automation rules |
| Microsoft security | ❌ (filter only) | ❌ | ❌ | Creates incidents from product alerts — redundant in the Defender portal |
| Fusion | ❌ | ❌ | ❌ | **Disabled once onboarded to the Defender portal** — XDR correlation engine replaces it |
| ML behavior analytics | ❌ | ❌ | ❌ | Built-in ML (e.g. anomalous SSH/RDP logon) |
| Threat Intelligence | ❌ | ❌ | ❌ | Matches Microsoft-generated TI against your logs |
| Anomaly | ❌ (thresholds tunable) | ❌ | ❌ | Writes to `Anomalies` table, not incidents |

**Rule of thumb:** a stem asking for entity mapping, custom logic or custom frequency → **scheduled** (or NRT if it says "near real time").

---

## 3. Defender XDR custom detection rules

**Required output columns**

| Data | Project |
|---|---|
| MDE tables (`Device*`) | `Timestamp` + `DeviceId` + `ReportId` → correct device-group scope + process tree |
| Other Defender tables | `Timestamp` + `ReportId` from the same event |
| Impacted asset | One strong ID: `DeviceId`/`DeviceName`, `AccountObjectId`/`AccountSid`/`AccountUpn`, `RecipientEmailAddress`… |
| Email actions | `NetworkMessageId` + `RecipientEmailAddress` |
| Sentinel scoping | `SentinelScope_CF`, otherwise alerts are hidden from scoped analysts |

**After `summarize`** — keep the columns with `arg_max`:
```kusto
| summarize (Timestamp, ReportId)=arg_max(Timestamp, ReportId), count() by DeviceId
```
A `project` after `summarize` can't bring the columns back.

**Don't** filter on `Timestamp`/`TimeGenerated` — the service already prefilters on the lookback (it uses `ingestion_time()`).

**Frequency → lookback**

| Frequency | Lookback (Defender data) | Constraint |
|---|---|---|
| Continuous (NRT) | Real time | One table, no `join`/`union`/`externaldata`, supported operators only, **no comments**, supported tables, GA columns only |
| Every hour | 4 h | — |
| Every 3 h | 12 h | — |
| Every 12 h | 48 h | — |
| Every 24 h | 30 d | — |
| Custom (5 min – 14 d) | Configurable | **Sentinel-only data** |

On save, a rule first runs against the last 30 days. Each run generates at most 150 alerts.

**Permissions:** Defender data → *Security settings (manage)* or Security Administrator. Security Operator works only if MDE RBAC is off; otherwise it also needs *Manage security settings*. You also need permissions on every device group in the rule scope.

**Response actions:** device (isolate, investigation package, AV scan, AIR, restrict app execution) · file (allow/block, quarantine) · user (mark compromised, disable, reset auth) · email (move to folder, soft/hard delete).

---

## Traps

- "Which **two** roles" with one option containing both → pick that one option only.
- Automation Contributor in an option for a **user** → wrong.
- "Onboarded to the Defender portal" + "enable Fusion" → wrong. The XDR correlation engine already does multistage correlation.
- `join` in a custom detection → not NRT. If the data is Defender data, Custom frequency isn't available either → **Every hour**.
- Adding `where Timestamp > ago(...)` to fix a custom detection → distractor.
