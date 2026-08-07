# Module 2 — Sentinel Training Lab: Prerequisites & Onboarding

**SC-200 domains touched:** Manage a security operations environment (40–45%) — workspace setup, roles, primary-workspace concept; Respond to security incidents (35–40%) — incident correlation, entity graph
**Resource:** [`Azure/Azure-Sentinel` → `Tools/Microsoft-Sentinel-Training-Lab`](https://github.com/Azure/Azure-Sentinel/tree/master/Tools/Microsoft-Sentinel-Training-Lab)
**Type:** Guided learning — not portfolio material. The goal is to see real incident mechanics (correlation, entity graphs, playbooks) before building original detections from Module 3 onward.

---

## 1. Portal correction discovered during this module

As of **1 July 2026**, Microsoft Sentinel's classic Azure portal experience is retired. Sentinel is managed exclusively through the **Microsoft Defender portal** (`security.microsoft.com`) now, with no E5 or Defender XDR license required to do so. This wasn't reflected in the project roadmap (v2.1, 29.07.2026) because the cutover is very recent. Every Sentinel-related "Portal" step from this module forward means the Defender portal, not `portal.azure.com`.

> 💡 **Technical Know-How:** A Log Analytics workspace with Sentinel enabled can be connected to Defender XDR's unified security operations platform as either the **primary** workspace or a **secondary** one. Only one workspace per tenant can be primary. This distinction is examinable — it affects where incidents get created and correlated.

---

## 2. Prerequisites

### 2.1 Subscription role
Confirmed Owner on the Azure for Students subscription — no action needed as sole subscription owner.

### 2.2 Custom Detection Rules Setup — originally attempted via Managed Identity

The packaged lab deploys pre-built custom detection rules to Defender XDR via Microsoft Graph, which requires an identity holding the `CustomDetection.ReadWrite.All` Graph application permission. The documented path is a User-Assigned Managed Identity (UAMI), created and granted via Cloud Shell:

```bash
curl -fsSLo sentinel-training-lab-onboarding.sh \
  https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/Tools/Microsoft-Sentinel-Training-Lab/Tools/sentinel-training-lab-onboarding.sh
bash sentinel-training-lab-onboarding.sh rg-soctraining-chn-01
```

This UAMI approach was ultimately abandoned — see Errors §3.1 — in favor of a manually created App Registration (Service Principal), using the identical Graph permission grant pattern:

```bash
APP_ID=$(az ad app create --display-name sentinel-training-detection-rules-spn --query appId -o tsv)
az ad sp create --id $APP_ID
SP_OBJECT_ID=$(az ad sp show --id $APP_ID --query id -o tsv)
SECRET=$(az ad app credential reset --id $APP_ID --years 1 --query password -o tsv)
TENANT_ID=$(az account show --query tenantId -o tsv)

GRAPH_SP_ID=$(az ad sp show --id "00000003-0000-0000-c000-000000000000" --query id -o tsv)
BODY=$(printf '{"principalId":"%s","resourceId":"%s","appRoleId":"e0fd9c8d-a12e-4cc9-9827-20c8c3cd6fb8"}' "$SP_OBJECT_ID" "$GRAPH_SP_ID")
az rest --method POST --uri "https://graph.microsoft.com/v1.0/servicePrincipals/$GRAPH_SP_ID/appRoleAssignedTo" \
  --headers "Content-Type=application/json" --body "$BODY"
```

The App Registration and its client secret were deleted after use (`az ad app delete --id $APP_ID`) — a one-time-use credential, no reason to leave it standing.

> 💡 **Technical Know-How:** Graph **application permissions** (app roles, like `CustomDetection.ReadWrite.All`) are granted to a service principal directly and used for app-only auth (Managed Identity or Service Principal with client secret) — distinct from **delegated permissions**, which act on behalf of a signed-in user and depend on what the calling client app has been consented for. This is why a Service Principal was a clean substitute for the UAMI: same permission model, different hosting mechanism.

---

## 3. Onboarding — Setting up the environment

### Step 1 — Log Analytics workspace
Created `log-soctraining-chn-01` in resource group `rg-soctraining-chn-01`, region **Switzerland North**, via Microsoft Sentinel → Create → Create a new workspace.

### Step 2 — Add Microsoft Sentinel
Microsoft Sentinel → Create → selected the workspace → Add.

### Step 3 — Connect via the Defender portal
Signed in to `security.microsoft.com` → Settings → Microsoft Sentinel → Workspaces → Connect. This step didn't exist before the July 2026 portal cutover — see §1.

### Step 4 — Deploy the Training Lab solution
Used the repo's "Deploy to Azure" button (custom ARM deployment), targeting `rg-soctraining-chn-01` / `log-soctraining-chn-01`, with the App Registration's identity supplied for the detection-rules identity parameter. Post-deploy, granted the enrichment playbook's system-assigned identity the **Microsoft Sentinel Contributor** role on the resource group — without this the playbook has permissions to nothing and silently never fires.

---

## 4. Errors Encountered and Resolved

This module surfaced more infrastructure friction than any prior one — almost none of it was Sentinel-specific; most was Azure-for-Students subscription behavior and Cloud Shell environment quirks that are worth knowing regardless of what's being deployed.

### 4.1 Automation Accounts blocked on Student subscriptions, in any region
The packaged template's telemetry ingestion and detection-rule deployment both ran via Azure Automation Account runbooks. First deploy failed with:
> `Free Trial and Student subscriptions cannot create accounts in this location. Please select from the allowed regions: [eastus, eastus2, westus, northeurope, southeastasia, japanwest]`

This is a well-documented, long-standing Azure platform restriction on Automation Accounts specifically for Free Trial/Student subscription tiers — confirmed against multiple Microsoft Q&A threads, not specific to this project.

**Attempted fix:** redeploy in `northeurope` (the only EU region on that list). This failed differently — `InvalidResourceLocation`, because the template unconditionally re-declares the Log Analytics workspace resource itself using the same location parameter that drives the Automation Accounts, and workspace location is immutable.

**Actual blocker:** the subscription carries a *second*, independent restriction — a stacked Azure Policy assignment. `az policy assignment list` revealed two "Allowed locations" policies simultaneously in effect:
- `Restrict to Switzerland North` → `{switzerlandnorth}`
- `Allowed resource deployment regions` → `{switzerlandnorth, italynorth, polandcentral, norwayeast, spaincentral}`

Multiple such policies apply as an **intersection**, not independently — so the only region this subscription can deploy *anything* into is `switzerlandnorth`. Zero overlap exists with the Automation Account allow-list, meaning Automation Accounts are structurally unavailable to this subscription, in any region, full stop — not a placement problem.

> 💡 **Technical Know-How:** Before assuming a region change will fix a regional resource restriction, check `az policy assignment list` for stacked "Allowed locations" policies. Multiple assignments restrict via intersection, and Azure for Students subscriptions commonly carry a narrow, tenant-specific allow-list independent of any resource-type-specific restriction.

**Resolution:** abandoned the Automation-Account-based flow entirely. Ran the underlying PowerShell scripts (`IngestCSV.ps1`, `DeployDetectionRules.ps1`) directly from Cloud Shell instead of via runbook — neither script actually requires an Automation Account; that was only the packaged solution's chosen execution context.

### 4.2 Cloud Shell environment quirks
Running the scripts manually surfaced several Cloud-Shell-specific issues the packaged (Automation Account) flow never would have hit:

- **`$env:TEMP` is unset on Linux.** The scripts assume a Windows Automation sandbox; `Join-Path -Path $env:TEMP` fails with a null path. Fixed by explicitly setting `$env:TEMP` before running.
- **Cloud Shell's `/tmp` is ephemeral.** Anything outside `$HOME` is wiped when the underlying container recycles (roughly 20 minutes idle). Lost a fully-downloaded working directory mid-session waiting on an unrelated RBAC propagation delay. Fixed by working entirely out of `$HOME` (e.g. `~/soctrain`) instead of `/tmp`.
- **Cloud Shell's built-in MSI identity only supports a fixed audience allow-list** for token issuance — `https://monitor.azure.com/` (needed for the Logs Ingestion API) isn't on it, confirmed against Microsoft's own documented list of supported audiences. Fixed with an explicit interactive login that bypasses the restricted broker: `az login --scope "https://monitor.azure.com//.default" --use-device-code`.

### 4.3 Data-plane vs. control-plane RBAC
Even as subscription Owner, telemetry ingestion failed with `403 Forbidden` — `"The authentication token provided does not have access to ingest data for the data collection rule..."`.

> 💡 **Technical Know-How, genuinely exam-relevant:** Owner/Contributor covers **control-plane** operations (create, modify, delete resources). The Logs Ingestion API is a **data-plane** operation, authorized completely separately. Writing to a Data Collection Rule requires the **Monitoring Metrics Publisher** role explicitly, regardless of any control-plane role already held. The same control-plane/data-plane split applies to Storage blobs and Key Vault (RBAC mode).

Granted `Monitoring Metrics Publisher`, initially scoped per-DCR (as the ingestion script supports natively via `-AssigneeObjectId`), then — after repeated propagation-delay failures on new DCRs — re-scoped once at the **resource group** level instead, covering every current and future DCR/DCE in one grant.

Microsoft's own documentation for this API specifies propagation can take **up to 30 minutes**, longer than general Azure RBAC guidance elsewhere. A precise elapsed-time check (using the role assignment's own `createdOn` timestamp rather than estimating from conversation pacing) confirmed the RG-scope grant had genuinely propagated before the successful retry.

### 4.4 `Get-AzAccessToken` SecureString breaking change
Detection-rule deployment failed with `IDX14102: Unable to decode ... as Base64Url encoded string` — not a permissions error, a malformed-token error. Root cause: Az.Accounts 5.0 (Az PowerShell 14.x) changed `Get-AzAccessToken`'s `.Token` property to return a `SecureString` by default instead of plain text. The script (written before this change) string-interpolated the token directly into the Authorization header, which silently produced the literal text `Bearer System.Security.SecureString` rather than erroring.

**Fix:** patched the script to detect and unwrap a `SecureString` via `[System.Net.NetworkCredential]::new("", $secureToken).Password` before use.

### 4.5 Custom Detection Rules API rate limit
Bulk-creating 22 rules in one pass hit `429 TooManyRequests` — capped at 10 calls/minute per app. The script has no backoff logic. Resolved by simple retry once the window cleared; the script's own "fetch existing rules first" check made retries safely idempotent (skips already-created rules).

### 4.6 Packaged rule content bugs
Two of the 22 rules (`Lab Stage 3`, `Lab Stage E5`) failed with column-resolution errors traced to a genuine authoring bug in the upstream `rules.json`: each query's `project` step dropped a column (`Sha256`, `Device` respectively) that a later `extend` step still referenced. Patched the locally downloaded copy of `rules.json` (string replacement, inserted post-download in the deployment script) to retain the needed columns before parsing.

`Lab Stage E5` deploys successfully after the patch but will never actually fire in this environment — its query joins against `DeviceInfo`, which only populates from MDE-onboarded devices, and this project deliberately has none (no E5 tenant, no VMs, by design). Deployed anyway for portfolio/demonstration value of the join pattern; documented as permanently inert here rather than left unexplained.

### 4.7 Known unresolved: `CrowdStrikeCases` ingestion failure
The built-in `CrowdStrikeCases` DCR deployment failed:
> `'OutputStream' stream 'Microsoft-CrowdStrikeCases' must be a custom stream or one of the allowed streams.`

This is a content bug in the packaged template itself — targets a stream name Azure Monitor doesn't recognize as valid. Not fixable via permissions or timing. Left as a known limitation; 20 of 21 telemetry tables ingested successfully, and this one table's absence doesn't block anything downstream.

### 4.8 Deferred by design, not a bug: `S8`, `E7`
Both reference `PaloAlto_ThreatSummary_KQL_CL`, a table populated only by a Sentinel Data Lake KQL summary job (Module 12 territory — Exercise 6 in this lab's own numbering). Deliberately left unresolved here rather than pulling Data Lake scope forward out of sequence; revisit at Module 12.

### 4.9 Zero incidents on first check — a timing issue, not a failure
Immediately after deployment, both Incidents and Alerts showed empty in the Defender portal despite everything reporting success. Verified data was actually present and fresh (`TimeGenerated` well within rule lookback windows via direct KQL query), which ruled out stale/mistimed telemetry as the cause. Resolved itself once the custom detection rules had time for their first scheduled execution — no fix required, just elapsed time.

---

## 5. Live state confirmed

Post-resolution: **20 of 21 telemetry tables ingested**, **20 of 22 detection rules deployed and active**, and real incidents visible in the Defender portal — including a genuinely strong result: a single correlated multi-stage incident (`Multi-stage incident involving Privilege escalation on one endpoint`) grouping 10 alerts spanning Initial Access through Impact on one device/account pair, alongside 5 additional standalone High-severity incidents from distinct devices/accounts.

---

## 6. Key Learnings

- **Check for stacked Azure Policy region restrictions before assuming a resource-type-specific region limitation is the whole story.** `az policy assignment list` reveals the actual effective allow-list; multiple assignments intersect rather than apply independently.
- **Azure RBAC data-plane permissions are separate from control-plane, and propagate on their own timeline** — up to 30 minutes for the Logs Ingestion API specifically, longer than general RBAC guidance suggests. Measure elapsed time from the role assignment's actual `createdOn` timestamp rather than estimating.
- **Cloud Shell has real, easy-to-miss constraints:** `/tmp` is ephemeral across container recycles (~20 min idle), `$HOME` is persistent; the built-in MSI broker only issues tokens for a fixed audience allow-list, requiring a genuine interactive login for anything outside it.
- **A recent Az PowerShell module version can silently break older scripts** — `Get-AzAccessToken` returning `SecureString` instead of plain text is a breaking change with no error message, just a garbled downstream token.
- **Packaged/community lab content isn't guaranteed bug-free.** Two rule-authoring bugs and one invalid-stream-name bug were all genuine upstream issues, not configuration mistakes — worth verifying rather than assuming a "no" from Azure means something on this end is wrong.
- **Automation-Account-based designs aren't the only way to run a script against Azure** — when the compute wrapper is unavailable, check whether the underlying script can run directly against `az`/interactive auth instead of assuming the whole approach is blocked.

---

## 7. Screenshots referenced

- Entity graph view of the correlated multi-stage incident

![sentinel-training-lab-multistage-incident-entity-graph](/diagrams/module-02/sentinel-training-lab-multistage-incident-entity-graph.png)

- The incidents queue showing all 6 live incidents

![sentinel-training-lab-incidents-queue-defender-portal](/diagrams/module-02/sentinel-training-lab-incidents-queue-defender-portal.png)

- The 20 deployed rules in Hunting → Custom detection rules

![entinel-training-lab-custom-detection-rules-list](/diagrams/module-02/sentinel-training-lab-custom-detection-rules-list.png)


---



