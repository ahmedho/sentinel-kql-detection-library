# Respond — Defender for Endpoint (SC-200)

Domain: Respond to security incidents (35–40%) → Defender for Endpoint
Bullets: device timelines · device actions incl. live response and investigation packages · evidence and entity investigation · incidents from automatic attack disruption

## Device actions — which one for what

| Need | Action | Scope | Notes |
|---|---|---|---|
| Cut the device off the network, keep Defender connected | **Isolate device** | One device | Live response and investigation package still work while isolated. Undo = *Release from isolation* |
| Only Microsoft-signed binaries may run | **Restrict app execution** | One device | Device-level lockdown, not a file block |
| Snapshot of system state for forensics, no interactive session | **Collect investigation package** | One device | Zip → download from Action center |
| Interactive investigation / run scripts / pull files | **Live response** | One device | See below |
| Kill + quarantine a known-bad file that already exists | **Stop and quarantine file** (file page) | Existing copies | Acts on what's already present, not future |
| Prevent a file from running anywhere in future | **Add indicator** (hash) → Block / Block and remediate | Tenant | The "future" keyword → indicator |
| Detonate in sandbox for more info | **Submit for deep analysis** | — | Information, not remediation |
| Let AIR investigate | **Initiate automated investigation** | One device | New alerts on the device join the running investigation |
| AV sweep | **Run antivirus scan** (quick/full) | One device | |

Licensing: MDE Plan 1 = only a subset of manual response actions. The full set needs **Plan 2**.

## Investigation package (Windows) — what's inside

Autoruns (ASEP registry keys) · Installed programs · Network connections (incl. DNS cache) · Prefetch files · Processes · Scheduled tasks · Security event log · Services · SMB sessions · System information · Temp directories · Users and groups · WdSupportLogs · `CollectionSummaryReport.xls` (what was collected, which command was used, any errors).
Firewall log is only included if `pfirewall.log` exists in its default path.
Persistence questions → Autoruns + Scheduled tasks + Services.

## Live response

| Item | Detail |
|---|---|
| Enable | Settings → Endpoints → **Advanced features** → Live response. Needs **Manage Portal Settings** |
| Servers | Separate toggle: *Live response for servers* |
| Unsigned scripts | Separate toggle: *Live response unsigned script execution*. Without it `run` fails on an unsigned `.ps1` even with advanced rights |
| Permissions | MDE RBAC / URBAC: **basic** vs **advanced** live response capability. Advanced requires basic as well |
| Basic (read-only) | `cd`, `dir`, `processes`, `connections`, `services`, `scheduledtasks`, `persistence`, `registry`, `fileinfo`, `findfile`, `drivers`, `prefetch`, `startupfolders`, `trace`, `status` |
| Advanced | `getfile`, `putfile`, `run`, `library`, `remediate`, `analyze`, `scan`, `isolate`/`release`, `undo` |
| Script flow | Upload to **library** → `run <script>` (`putfile` = copy a library file to the device, not run it) |

## Device timeline

- Device page → *Timeline*: filter by time window around the alert, event group, user, process.
- **Flag event** = highlight individual events for other analysts. A **device tag** marks the whole device. **Link to incident** is for alerts.

## Evidence and entities

- File / user / IP / domain / URL pages show org-wide prevalence, related alerts and devices observed.
- File page actions: Stop and quarantine · Add indicator · Download/Collect file · Deep analysis.
- "Has this hit anywhere else?" → entity page prevalence, then Advanced Hunting.

## Automatic attack disruption

| Item | Detail |
|---|---|
| Recognise it | Incident tag **Attack Disruption** + threat type; summary card on the incident |
| Device | **Contain** (blocks communication), not isolate |
| User | Contain user / **disable account via MDI** remediation (on-prem AD) |
| Release | From the incident: select asset → *Release from containment* (device) / *Enable user* (user). Or **Action center** → action details pane |
| Hunt | Contain actions are in **`DeviceEvents`** |

## Traps

- Re-enabling the user in the Entra admin center doesn't undo an MDI disable of an on-prem account.
- Never release isolation to collect evidence — package and live response work while isolated.
- Automation level (device groups) controls AIR, not live response.
- "Manage security settings" ≠ "Manage portal settings". Enabling live response needs portal settings.
- "Prevent in future, all devices" = indicator. "Already present" = stop and quarantine.
