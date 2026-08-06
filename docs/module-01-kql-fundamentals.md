# Module 01 — KQL Fundamentals

**SC-200 domain:** Not a standalone domain — KQL is the underlying skill tested across **Manage a security operations environment (40–45%)** (analytics rule authoring, workbooks, MITRE mapping) and **Perform threat hunting (20–25%)** (Advanced Hunting queries, Sentinel Graph, hunting graphs). It also appears throughout **Respond to security incidents (35–40%)** wherever KQL is used for investigation.

**Learning path:** [Create queries for Microsoft Sentinel using Kusto Query Language (KQL)](https://learn.microsoft.com/en-us/training/paths/sc-200-utilize-kql-for-azure-sentinel/) — 4 modules:
1. Construct KQL statements for Microsoft Sentinel
2. Analyze query results using KQL
3. Build multi-table statements using KQL
4. Work with data in Microsoft Sentinel using Kusto Query Language

**Practice environment used:** Kusto free help cluster, `Samples` database (`https://dataexplorer.azure.com/clusters/help/databases/Samples`) — substituted for the Log Analytics demo workspace after the latter proved unreliable. See *Errors Encountered and Resolved* below.

---

## Part 1 — Filtering and Shaping

`where` filters rows; `project` selects and reorders columns. Chain `where` before `project` — filtering first shrinks the working set before the engine has to reshape it, which matters directly on large tables like `SigninLogs`.

```kql
StormEvents
| where State == "TEXAS" and EventType == "Flood"
| project StartTime, EndTime, State, EventType, DamageProperty
```

💡 **Technical Know-How:** Query performance in KQL is dominated by how early you reduce the row count. A `where` clause placed early can turn a multi-minute query over millions of rows into a sub-second one. This is directly examinable — expect questions that test whether you understand operator *order*, not just operator syntax.

`extend` adds a computed column while preserving the existing ones, unlike `project`, which requires explicitly re-listing every column you want to keep.

```kql
StormEvents
| where State == "FLORIDA"
| extend TotalDamage = DamageProperty + DamageCrops
| project StartTime, EventType, DamageProperty, DamageCrops, TotalDamage
```

---

## Part 2 — Aggregation and Time Bucketing

`summarize` aggregates; `bin()` rounds a timestamp down to the nearest interval boundary before grouping. Grouping directly on a raw timestamp column is a common mistake — `TimeGenerated` is unique to the millisecond, so grouping on it directly produces one bucket per row and no real aggregation happens.

```kql
StormEvents
| where StartTime between (datetime(2007-08-01) .. datetime(2007-08-31))
| summarize EventCount = count() by bin(StartTime, 1d), State
| order by EventCount desc
```

💡 **Technical Know-How:** `between()` is cleaner than two chained `where` clauses with `>` / `<` for range filters, and reads unambiguously in a code review. `bin()` is the operator that makes time-bucketed detections (e.g. "N failed logons per 15-minute window") possible — swap `1d` for `15m` and this becomes a brute-force detection query.

---

## Part 3 — Multi-Table Queries: `join` vs `union`

These look similar but solve different problems, and mixing them up is a known exam trap.

- **`union`** stacks rows from multiple tables vertically. If both tables have a column with the same name but a different meaning, `union` silently merges the values into one column — the query runs without error and returns plausible-looking results that are actually meaningless.
- **`join`** links rows from different tables horizontally on a shared key. If that key isn't unique on one (or both) sides, matching rows multiply — a phenomenon called fan-out. Aggregations run *after* a fan-out join will be inflated, not missing.

```kql
let CountByState = StormEvents | summarize EventCount = count() by State;
let DamageByState = StormEvents | summarize TotalDamage = sum(DamageProperty) by State;
CountByState
| join kind=inner DamageByState on State
```

Fan-out reproduced deliberately, by joining on a non-unique key:

```kql
StormEvents
| where State == "TEXAS"
| join kind=inner (StormEvents | where State == "TEXAS") on State
| count
```

💡 **Technical Know-How:** always compare row counts before and after a join during development. If the joined result has more rows than either input, the join key isn't unique on at least one side — check for that before trusting any `summarize` built on top of the join.

Silent-merge trap with `union`, avoided by aliasing to a shared column name deliberately:

```kql
StormEvents
| where EventType == "Flood"
| project State, EventType, Damage = DamageProperty
| union (
    StormEvents
    | where EventType == "Tornado"
    | project State, EventType, Damage = DamageCrops
  )
```

---

## Part 4 — Extracting Structure from Free Text

`parse` and `extract()` both pull values out of unstructured strings, but they solve different problems:

- **`parse`** expects a *consistent* delimiter pattern and field order on every row. It's efficient but brittle — inconsistent rows return nulls silently.
- **`extract()`** uses a regular expression to match by pattern rather than position, so it tolerates rows where the surrounding text varies.

```kql
StormEvents
| where EventNarrative has "mph"
| extend WindSpeed = extract(@"(\d+)\s?mph", 1, EventNarrative)
| project EventNarrative, WindSpeed
| take 10
```

💡 **Technical Know-How:** free-text fields in security logs (alert descriptions, email subjects, process command lines) are rarely perfectly consistent. Default to `extract()` for anything sourced from a human-writeable or variable-format field; reserve `parse` for machine-generated logs with a fixed structure.

---

## Part 5 — Time Series with `make-series`

`make-series` differs from `summarize by bin()` in one important way: it fills gaps. A day with zero events shows up as an explicit zero, not as a missing row.

```kql
demo_make_series1
| make-series NumEvents = count() default = 0 on TimeStamp step 1d by Country
| take 5
```

💡 **Technical Know-How:** for anomaly detection, a gap in the data must not look identical to a gap in attacker activity. `make-series` is the operator underpinning anomaly detection and the newer Sentinel Graph / hunting-graph material in the current outline — don't treat it as optional.

---

## Part 6 — Defender XDR Schema (studied in parallel, not deferred)

Table-selection questions appear directly in the threat-hunting domain. Tables reviewed this module: `DeviceEvents`, `DeviceProcessEvents`, `IdentityLogonEvents`, `EmailEvents`, `SigninLogs`, `AuditLogs`, `AzureActivity`.

---

## Errors Encountered and Resolved

**Problem:** the Log Analytics demo workspace (`aka.ms/lademo`, shared workspace `CH1-LA`) returned stale or no data when running the Microsoft Learn module exercises. The commonly suggested community fix — adjusting the query time range — did not resolve it.

**Cause:** this is a known, long-running issue with Microsoft's shared demo environment, reported repeatedly on Microsoft Q&A since late 2024, with no permanent fix at the time of this module. It is not caused by anything on the user side.

**Resolution:** switched practice to the Kusto free help cluster (`https://dataexplorer.azure.com/clusters/help/databases/Samples`), which is separate infrastructure from the broken demo workspace and has a different authentication path. The `StormEvents`, `demo_make_series1`, and related sample tables don't match Sentinel's security schema, but every KQL operator behaves identically, making this a complete substitute for learning the *language* even though it isn't security data. Security-specific schema practice resumes in M2 (Sentinel Training Lab, real pre-recorded telemetry from six products) and M3 onward (own workspace).

---

## Key Learnings

- Operator order affects performance, not just correctness — `where` before `project`, always.
- `union` and `join` fail differently: `union` merges silently on shared column names; `join` multiplies rows silently on non-unique keys. Both produce a query that *runs successfully* while returning a wrong answer — neither throws an error.
- Comparing row counts before/after a join is a fast, reliable sanity check.
- `extract()` beats `parse` for anything not perfectly consistent field-to-field.
- `make-series` is not optional — it's the basis for anomaly detection and is explicitly new-outline material.
- The Log Analytics demo workspace cannot be relied on for time-boxed study; the Kusto help cluster is a stable fallback for pure KQL practice when it fails.
