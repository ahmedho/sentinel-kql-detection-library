# Module 2, Exercise 5 — Cross-Platform Response Actions (Device Isolation)

**SC-200 domain:** Respond to security incidents (35–40%) — device isolation and cross-platform response actions.
**Status:** Skipped — not applicable to this environment.

---

## Why this exercise was skipped

The exercise's own prerequisites state it explicitly: *"This exercise requires a machine onboarded to Microsoft Defender for Endpoint (MDE)."* This project has no M365 E5 tenant and, by deliberate design, no MDE-onboarded devices exist or ever will within this project's scope. There is no workaround available for this one.

This is the same limitation already surfaced once before: the packaged custom detection rule `Lab Stage E5 - Device Isolation Response (CrowdStrike)` (deployed in Onboarding, after a query-bug patch) was noted at the time as **permanently inert** for exactly this reason — its query joins against `DeviceInfo`, which only populates from MDE-onboarded devices. This exercise is very likely the guided walkthrough companion to that same rule/scenario, so the gap is consistent across both.

## What this covers conceptually, for the exam

Even without hands-on access, worth knowing what this exercise would demonstrate: resolving a security alert (e.g., a CrowdStrike detection) to its corresponding MDE `DeviceId`, then invoking a cross-platform response action (device isolation) directly from the unified incident — the kind of action that requires the Defender XDR device-management layer specifically, not just Sentinel's data plane. This is study-only material per the project roadmap's existing Defender XDR track (§1.3), not something this module needed to build.

## Screenshots referenced

None — no hands-on work performed.

---

**Deliverable status:** Skipped (environment limitation, documented). Proceeding to Exercise 6.
