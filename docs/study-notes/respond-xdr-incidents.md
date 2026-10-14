# Respond — Defender XDR incidents, case management, Copilot, multi-domain attacks

SC-200 domain: **Respond to security incidents (35–40%)** → Defender XDR
Bullets: investigate Sentinel incidents in XDR · agentic AI incl. embedded Security Copilot · multi-stage / multi-domain / lateral movement · case management

---

## Alert vs incident vs case

| Object | What it is | Created by | Key point |
|---|---|---|---|
| Alert | One detection from one source (MDE, MDO, MDI, MDCA, Entra ID Protection, Sentinel rule…) | The detecting product / rule | Can be moved: **Manage alert → Link alert to another incident** |
| Incident | Correlated group of alerts + assets + evidence for one attack | Defender XDR correlation engine (replaces Sentinel Fusion after onboarding) | One queue for Defender and Sentinel incidents in `security.microsoft.com` |
| Case | SecOps work container: owner, tasks with due dates, custom statuses, notes, linked incidents | Analyst (or automation) | Use when several incidents = one investigation and you must **not** change their correlation |

## Incident page — where to look

| Need | Tab |
|---|---|
| How the attack unfolded, entity graph | Attack story (incident graph) |
| Which detections, from which product | Alerts |
| Affected devices, users, mailboxes, apps | Assets |
| AIR status for the incident | Investigations |
| Files, processes, IPs, emails + pending/completed remediation | Evidence and response |

## Resolve + classify

| Situation | Classification |
|---|---|
| Real attack | True positive → determination (multistage attack, compromised account, malware, phishing…) |
| Pen test / red team / approved tool | **Informational, expected activity** → security test |
| Detection logic was wrong | False positive |

- Incidents are never deleted. Resolve and classify.
- Resolving an incident resolves its alerts.

## Embedded Security Copilot

| Need | Capability |
|---|---|
| "What happened?" (auto-generated on opening the incident) | Incident summary |
| "What do I do now?" (actions you can apply) | Guided response |
| "What did we do?" (timeline, actions, analysts — post-incident) | Incident report |
| Obfuscated PowerShell / script on an alert | Script analysis |
| Suspicious file | File analysis |
| Device posture, vulnerable software, unusual behaviour | Device summary |
| Plain-language question → hunting query | NL-to-KQL (Advanced Hunting) |

- Requires provisioned Security Copilot access. Not part of an E5 licence by itself.
- The same skills exist in the standalone portal via the Defender XDR plugin.
- Standalone prompts should include the word "Defender".

## Multi-domain / lateral movement

| Need | Tool |
|---|---|
| Which non-sensitive accounts/devices lead to Domain Admin via cached credentials (on-prem AD) | **MDI lateral movement paths** (identity page) |
| Attack paths across cloud workloads | Defender for Cloud attack path analysis |
| What happened on one device, in order | MDE device timeline (S03) |
| Campaign context, exposure, mitigations | Threat analytics |

## Sentinel incidents in the Defender portal

- After onboarding, Sentinel incidents appear in the unified Defender queue.
- Status, owner and classification sync in both directions with the Azure portal.
- XDR correlation replaces Fusion.

## Traps

- "Keep the original alerts/incidents unchanged" plus "one investigation with tasks" → **case**. Not linking alerts, not tags.
- Summary, guided response and report are all real Copilot features. Match each one to the verb in the stem.
- "Cloud" in the stem → Defender for Cloud attack paths. "On-prem AD credentials" → MDI LMPs.
- Pen test is not a false positive. The detection was correct.
- "Delete the incident" is never the answer.
