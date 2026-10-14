# KQL Cheat Sheet — Personal Reference

Quick-reference for the operators covered in M1. Built for exam recall and fast lookup during detection engineering.

| Operator | Purpose | Syntax pattern | Watch out for |
|---|---|---|---|
| `where` | Filter rows | `\| where Column == "value"` | Place early — before `project`/`extend` — for performance |
| `project` | Select/reorder columns | `\| project Col1, Col2, Col3` | Drops any column not listed |
| `extend` | Add computed column, keep existing | `\| extend NewCol = ColA + ColB` | Doesn't remove columns like `project` does |
| `summarize` | Aggregate rows | `\| summarize count() by Column` | Grouping on raw timestamps gives one row per group — use `bin()` |
| `bin()` | Round a value to an interval | `bin(TimeGenerated, 15m)` | Use inside `summarize by` for time-bucketed aggregation |
| `between()` | Range filter | `\| where Col between (low .. high)` | Cleaner than two `where` clauses for ranges |
| `ago()` | Relative time filter | `\| where TimeGenerated > ago(24h)` | Common in every detection query |
| `join` | Combine tables horizontally on a key | `\| join kind=inner T2 on KeyCol` | Non-unique join key → row fan-out → inflated aggregates |
| `union` | Stack tables vertically | `\| union T2` | Same column name, different meaning → silent merge |
| `parse` | Extract fields, fixed pattern | `\| parse Col with "prefix" Field "suffix"` | Breaks silently on inconsistent row structure |
| `extract()` | Extract via regex | `extract(@"(\d+)mph", 1, Col)` | More tolerant than `parse` for variable text |
| `make-series` | Time series with gap-filling | `\| make-series Val=count() default=0 on TimeCol step 1d by Group` | Fills zero-count gaps; `summarize by bin()` does not |

## Defender XDR schema quick list

`DeviceEvents` · `DeviceProcessEvents` · `IdentityLogonEvents` · `EmailEvents` · `SigninLogs` · `AuditLogs` · `AzureActivity`

## Fast decision rules

- **`join` vs `union`** — need columns from two tables side by side → `join`. Need rows from two tables stacked → `union`.
- **`parse` vs `extract()`** — structure is 100% consistent → `parse` (faster). Structure varies → `extract()` (regex, more tolerant).
- **`summarize by bin()` vs `make-series`** — just need a count per bucket → `summarize by bin()`. Need every bucket represented, including zeros, for trend/anomaly analysis → `make-series`.
- **Before trusting any `join` result:** compare row count before and after. More rows out than in on the "one" side of a 1:1 assumption = fan-out.
