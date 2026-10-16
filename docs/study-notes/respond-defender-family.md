# Respond — Defender family (MDO, Entra ID Protection, MDI, MDCA, Defender for Cloud, Purview)

SC-200 → Respond to security incidents (35–40%) → Defender XDR group.
Outline: skills measured as of 21.10.2026 (this group unchanged).

## One row per product

| Product | Alert source | Investigate in | Typical remediation | Trap |
|---|---|---|---|---|
| **MDO** | Email/collab detections, user reports, ZAP | Defender portal → Threat Explorer (P2) / Real-time detections (P1), email entity page | Explorer → **Take action**: soft/hard delete, move to junk/inbox | Pending AIR actions are approved in **Action center → Pending**, not in Explorer |
| **Entra ID Protection** | Risk detections → risky sign-ins / risky users | Entra admin center → ID Protection; user entity in Defender | Confirm compromised / safe, dismiss, admin password reset, risk-based CA | **Confirm sign-in compromised → user risk = High** (triggers high-risk CA) |
| **MDI** | DC sensors (Kerberos, NTLM, LDAP, DNS, events) | Defender portal → incident, identity page (timeline, lateral movement paths) | **Disable user in AD**, **Force password reset in AD** | Disabling a *synced* user in Entra doesn't block on-prem auth and is overwritten by the next sync |
| **MDCA** | Anomaly / activity / OAuth / file policies, app governance | Defender portal → Cloud apps, activity log, app governance | **Ban / revoke OAuth app**, suspend user, governance actions | **Unsanctioned** = Cloud Discovery (shadow IT). It does not touch OAuth consents |
| **Defender for Cloud** | Workload plans (Servers, Storage, SQL, Containers, Key Vault…) | Defender for Cloud alert page; alerts also flow into Defender XDR incidents | Alert → *Take action*: mitigate, prevent, trigger automation, suppress | Auto-run a Logic App per alert type = **workflow automation**. Governance rules are for recommendations, not alerts |
| **Purview** | DLP alerts, insider risk alerts | Defender XDR incident queue (DLP); policy/content in Purview | Tune the policy in Purview; respond on user/device in Defender | Purview *Audit* is a different bullet (M365 activities) |

## MDO — removing delivered mail

| Need | Use | Not |
|---|---|---|
| Remove phish already in mailboxes, on demand | **Threat Explorer → Take action → Soft delete** (MDO P2) | ZAP (automatic only), TABL (future mail only), Hold (preserves mail) |
| Approve an AIR-proposed remediation | **Action center → Pending** (or the investigation inside the incident) | Explorer, Quarantine, Submissions |
| Audit what remediations were done | **Action center → History** | — |
| Release a false positive | **Quarantine → Release** | Action center |

## Entra ID Protection — risk actions

| Action | Effect |
|---|---|
| Confirm sign-in compromised | Sign-in risk → High **and user risk → High**. Feeds the ML model |
| Confirm user compromised | User risk → High |
| Confirm sign-in safe | Closes that sign-in's risk. Feeds the ML model |
| Dismiss user risk | User risk → None. No password change, no learning signal |
| Admin password reset / user secure password change | Remediates user risk |

- User-risk policy → secure password change. Sign-in-risk policy → MFA. Configure both in **Conditional Access**.
- Hybrid trap: an on-prem password change only resets user risk if the tenant allows it, which requires password hash sync.

## MDI — response actions

| Action | Where it acts | Executed by |
|---|---|---|
| Disable user in AD / Force password reset in AD | On-prem AD | MDI sensor on the DC. **Default: LocalSystem**; a gMSA action account is optional (least-privilege control) |
| Suspend user in Entra ID / Mark user compromised / Require sign-in again | Entra ID | Entra roles of the analyst (e.g. User Admin, Security Admin) |

- Actions are available from the identity page, the user side panel, Advanced Hunting and the Action center.
- Validate a disable in AD with event IDs 4725 and 4738.

## MDCA — which control for which problem

| Problem | Control |
|---|---|
| Malicious OAuth app with user consents | Ban / revoke on OAuth apps or app governance; OAuth app policy to auto-revoke |
| Shadow-IT app found in network traffic | Cloud app catalog → Unsanctioned (block via MDE integration or exported firewall script) |
| Real-time control of a session (download, copy) | Session policy (Conditional Access App Control) |
| Files overshared in a connected app | File policy + governance action (quarantine, remove sharing) |
| Unusual user behaviour (impossible travel, mass download) | Anomaly detection policies |
| What did MDCA actually do? | Governance log |

## Defender for Cloud — alert response

- Alert page → *Take action*: inspect resource context · mitigate the threat · prevent future attacks · trigger automated response · suppress similar alerts.
- Automatic Logic App on a specific alert type → **workflow automation**, configured in Defender for Cloud.
- Sentinel automation rule → only if the stem says to use Sentinel. AIR → Defender XDR / MDE / MDO, not Defender for Cloud.
- Governance rules assign owners and due dates to **recommendations**. Distractor in alert stems.
