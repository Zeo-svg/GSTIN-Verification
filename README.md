# GSTIN Compliance & Verification (ServiceNow scoped app)

GSTIN verification and ongoing GST compliance monitoring for Indian suppliers in **Supplier Lifecycle Operations (SLO)**.
Validates the GSTIN format and checksum, checks it against a GST provider (mock in this phase), compares legal names, logs every attempt, and blocks supplier activation until the GSTIN is verified or an approved override exists. Active suppliers are re-checked daily; a Cancelled or Suspended GSTIN places a compliance hold.

| | |
|---|---|
| Application | GSTIN Compliance & Verification |
| Scope | `x_2078844_gstin_0` |
| Release | ServiceNow Australia (PDI dev297253) |
| Team | Rahul Basutkar · Hitesh Gohad · Rutul Jadhav |
| Status | Week 1 – setup and foundation (see project board) |

---

## What is in this repository

- The **scoped application** (tables, script includes, flows, ACLs, roles, portal widgets, GST Compliance Workspace), committed from ServiceNow Studio.
- `/companion-update-sets` – XML export of the update set **"GST – SLO playbook changes"** (Supplier Case Management scope). The SLO onboarding playbook is an OOB record and cannot live in the scoped app, so the India variant, evaluation point, 3.6 override and 2.8 fix travel here.
- This README.

## Solution overview

```
Supplier (portal /supplier)            Agent (GST Compliance Workspace)
   │  Provide GST details task              │  review queue, decision panel, Verify Now
   ▼                                        ▼
GST Compliance Profile ──► Verify GSTIN (F1) ──► Provider adapter (mock / live)
   │                          │
   │                          └─► GST Verification Log (every attempt)
   ▼
Playbook India variant: GST gate (Wait For Condition) ─► Activate supplier record
Safety rule on sn_fin_supplier ─► blocks Onboarded = Yes without Verified / override
Daily re-check (F3, 02:00 IST) ─► hold + critical task on Cancelled / Suspended
```

Main custom tables (prefix `x_2078844_gstin_0_`): `gst_profile`, `gst_verification_log`, `gst_review_task` (extends task), `gst_mock_registry`.
OOB tables used (no fields added): `sn_fin_supplier`, `sn_fin_org_tax_detail`, `sn_fin_tax_type`, `sn_slm_case`, `sn_slm_task`.

---

## Working rules (one shared instance, three developers)

### Git
1. **One shared branch: `sn_instances/dev297253`.** ServiceNow commits here. **Never use "Switch branch"** in Studio – it replaces the app for all three developers.
2. **`main` = released history.** At each milestone the branch is merged into `main` and tagged: `week1-done`, `week2-done`, `v1.0`.
3. **One agreed committer** commits at least once a day and before any risky change.
4. **Commit messages start with the story IDs**, e.g. `GST-E1-02, GST-E1-03: profile and log tables`.
5. At each milestone, export the companion update set as XML and commit it to `/companion-update-sets`.
6. **No secrets in the repo.** The GitHub token and any API keys live only in credential records on the instance.
7. Do not edit app files directly on GitHub in `sn_instances/dev297253` – change them in ServiceNow and commit from Studio.

### Update sets
- One story = one update set, named after the story (e.g. `GST-E1-02 Profile table`), in the app scope. Mark it **Complete** when the story is Done; never reuse it.
- Playbook changes only in the companion update set `GST – SLO playbook changes` (Supplier Case Management scope). Only the named playbook editor changes the playbook.
- Before every change, check the header: correct application scope **and** correct update set.

### Board
- Stories (GST-E0-01 … GST-E8-08) are tracked on the CWM board **GST Verification And Compliance** (space *GST Verification*) on the instance, in sprints Week 1 / Week 2 / Week 3.
- Status flow: To do → In progress → In review → Done (Blocked with a note). Another developer checks the acceptance criteria before Done.

### Definition of Done
Built in the right scope and update set · all acceptance criteria checked with evidence · committed with the story ID · no OOB record edited outside the companion update set · Must stories with logic have an ATF or written manual test · board updated.

---

## Install on another instance (summary – full guide in GST-E8-06)

1. Install the SLO plugins: Supplier Lifecycle Operations, Supplier Case Management, Supplier Collaboration Portal, Source-to-Pay Workspace.
2. Import this app from source control (Studio › Import from source control).
3. Load and commit the companion update set from `/companion-update-sets`.
4. Activate the Supplier onboarding playbook; check the cross-scope access records.
5. Load demo data (mock registry, GSTIN tax type, groups) if required.

## Known OOB issues found on the PDI
- Supplier invite / password-reset link opens `/cab` (404) – supplier contacts log in at `/supplier`.
- Playbook activity 2.8 "Perform risk assessment" has no activity definition – avoid the "Perform risk assessment" path; fix tracked in GST-E0-09.

---
*Case study project – not for production use. Mock GST data only.*
