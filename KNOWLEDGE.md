# CMS HR Ops Command Centre — Project Knowledge
**Version:** 2.7.0 | **Last updated:** 09 Jun 2026 — ATE conversion tracking, Account Health region/attrition fixes, resignation tab fixes, eSep audit

---

## What This Is
Single-file HTML HR Operations dashboard for CMS IT Services.
Used daily by HR team, HRBPs, Talent Acquisition, RMG, and executive leadership.

Covers: Overview (CXO command strip), TA pipeline, HRBP connects and scorecards,
HR Ops / payroll, L&D, Risks, Workforce Intelligence, Resignation tracker, OD/WFH
compliance, FnF tracker, Long Absenteeism, ATE tracker, Prospective Joiners,
Action Centre, Admin/Settings.

---

## Live Deployment
- URL: cms-hr-dashboard.vercel.app
- Repo: alexaugustine1-rgb/cms-hr-dashboard
- Branch: main | File: index.html (always at repo root)
- Vercel auto-deploys on every push to main
- Git author: alexaugustine1@gmail.com
- PAT: [redacted — store in password manager, not in repo]
- vercel.json: no-cache headers on / and /index.html — keep committed

---

## Supabase
- Project ID: mzyrcrkwgbqgwajkjdnp
- Shared with OPS360 (different schemas, same project)
- execute_sql for DML/queries | apply_migration for DDL
- RLS with zero policies = 400 errors. Always add policies on CREATE.
- pg_trgm extension: ENABLED (for normalise_account_name function)

---

## Architecture — Non-Negotiable Rules
- Single-file HTML. Never split.
- No build toolchain. No npm, webpack, React.
- All persistent data in Supabase. No hardcoded arrays with real data.
- str_replace ONLY for edits. Never rewrite full file.
- node --check after EVERY edit. Extract scripts to temp .js first (Node v24 won't check .html directly). No output = clean pass.
- Node.js path: C:\Program Files\nodejs\node.exe (confirmed installed)

---

## Architecture Notes

### Absent Resolution Chip
Reads `window._absResolvedCount` and `window._absTotalCount` — set by two separate
queries in sbBoot() (ILIKE 'resolved%' count + total head count).
NOT derived from `_absentCases` (that array excludes resolved cases by design).
Do NOT modify loadAbsentCases() query — it intentionally excludes resolved.

### innerHTML Does NOT Execute `<script>` Tags
Any function callable from onclick handlers in dynamically rendered HTML MUST be
declared as a true global (top-level script scope), not inside an innerHTML string.
Confirmed crash: hscToggle was inside innerHTML `<script>` — moved to global (25 May 2026).

### Role Gate Pattern — Use Both Fields
`_sbRole.role` and `_sbRole.function_type` are DIFFERENT fields. Role gate checks must read both:
```javascript
var role   = _sbRole ? (_sbRole.role   || '') : '';
var fnType = _sbRole ? (_sbRole.function_type || '') : '';
```
Comparing only function_type blocks admin users (function_type='admin', not 'hrops').

### L&D Tab Architecture (redesigned 05 Jun 2026)
`renderLD(w, pw)` returns a static shell only. `loadLDTab(monthKey)` drives all data loading.
- Called by: render() (100ms), switchTab('ld'), ldMonthSel onChange
- `_loadLDSessions(mk)`: training_sessions → KPI chips + table
- `_loadLDPipeline(mk)`: training_pipeline (Planned/Confirmed) → pipeline table
- Month selector id: `ldMonthSel`
- Constants: `LD_TRAINERS` (7 names) + `LD_CATEGORIES` (6 types)

### L&D Form Data Flow
- f_ld_pipeline (structured grid) → training_pipeline on submit
- HRBP form Section 1 → training_sessions; Section 2 → training_pipeline

### HRBP Site Visit Cache
`_hrbpSVCache` holds ALL site visits — not week-restricted. Journey Plan panel and
scorecard both read from this cache. loadHRBPSiteVisitsForTab() triggers deferred
re-render (50ms setTimeout) after populating the cache.

### Postgres: No CURRENT_DATE in Stored Generated Columns
IMMUTABLE requirement — Postgres rejects CURRENT_DATE / now() in stored generated columns.
- hrbp_er_cases.days_open + sla_breached: computed by trigger `_hec_compute_days`.
- account_positions_log.days_vacant: GENERATED STORED (expression does not reference today).

---

## Known Crash Patterns — Check Every Edit
Any of these crashes login with "ReferenceError: sbLogin is not defined":
- Orphaned em-dash in string concatenation
- await inside a non-async function
- Block-scoped function declarations inside try blocks
- const or let declared inside try blocks

When node --check reports an error: print 3 lines above and 3 below before attempting any fix.

---

## Key Learnings

### ATE Conversion Emp Code Problem
ATEs have 22xxxxxxx emp codes. On FTE conversion ZingHR issues a new 11xxxxxxx code —
old code disappears from Super Emp Master. Pure emp_code matching always fails for
conversion detection.

**Solution**: HRBP enters new FTE emp code in `fte_emp_code` field at Conversion Initiated
stage. `processEmployeeMaster` reconciliation auto-confirms when `fte_emp_code` appears in
active non-ATE rows. If old 22xxxxxxx is missing AND no `fte_emp_code` set → appends
warning note to `action_notes`.

### getEmpType() Field Priority
Reads: EmpType → Emp Type → EmployeeType → Employee Type → Employment Type → Grade
(first non-empty wins). ZingHR column CK = EmpType. Exact uppercase match required.
Cell must say 'ATE' — no partial match.

ATE is now identified by Grade (col CM) = 'ATE' via `isATE(r)` helper — NOT EmpType.
This corrects the count from 27 (EmpType) to 65 (Grade), because 38 ATEs are tagged
'Trainee' in EmpType but 'ATE' in Grade.

### Account Health — Attrition Formula
Rate is annualised: `exits / activeHC × (365/90) × 100`.
Thresholds 3% / 8% / 15% are **annual** rates. Label shows '(90d) · ann.'.
Exit count uses live LWD computation from `r.lwd` date string — NOT the frozen
`days_to_lwd` integer baked in at upload time.

### Account Health — Region Filter Pattern
- `_absentCases`: `.select('*')` — region field present ✅
- `FNF_DATA`: has region field ✅
- `CMS_RES_DATA`: has region field ✅
- `_hrbpSVCache` (last visit): NO region field — pan-account only ⚠
Filter pattern: `if (acct.r && r.region && r.region !== acct.r) return false;`

### Frozen days_to_lwd — Root Cause
`days_to_lwd` is baked into CMS_RES_DATA (and processResignationData output) at upload
time. Any code that uses it directly will drift as days pass. Always recompute from
the `r.lwd` date string at render/compute time. Same fix applied to both renderResignation
and renderAccountHealth.

---

## Current File State — 09 Jun 2026
- Line count: ~12,376 (latest confirmed after commit 8f9edc6)
- Latest commit: 8f9edc6
- Local path: D:\Dropbox\CMS_IT_Services\Claude_Projects\Claude_HR_Automation\HR_OPS_COMMAND\cms-hr-dashboard\index.html

### Commits — 09 Jun 2026
- 8f9edc6: Fix: Account Health — region filter on signals, live LWD computation, annualised attrition rate
- c355d6d: Feat: ATE conversion tracking — fte_emp_code field, Super Emp Master reconciliation, UI badges
- 9f58ac4: Docs: KNOWLEDGE.md v2.6.0 — 09 Jun 2026 session summary
- bf3f056: Fix: ATE detection — Grade (CM) not EmpType (CK), corrects ATE count 27 → 65
- 0119aff: Fix: resignation tab — recompute days at render, remove Location, add exit reason, same-date no-notice badge
- 6061d8e: Fix: FnF upload refresh type, super_emp in-memory WI + _acctHC reload
- c4da088: Fix: account health HC — region-aware composite key in _ahcMap
- Supabase only: ate_tracker reconciliation — 4 exited ATEs set Active → Separated

### Commits — 08 Jun 2026
- e48a8f0: Feat: Action Centre — clickable KPI chips, filter bar, table view for admin/exec
- 3c5a178: Fix: post-upload tab refresh — postUploadRefresh() force-renders affected tabs
- RLS fix (Supabase only): workforce_intel + data_cache write policies for hrops role

### Commits — May–07 Jun 2026 (summary)
- 0d9450d: OD/WFH enhancements — wfh_od_actions, multi-instance badge, HRBP action dropdown
- ed35e5d: ATE Advance upload — DBT Advance panel, processATEAdvance(), ate_advance_log
- 4b9d846: HRBP schema — hrbp_visit_plans, hrbp_er_cases, retention/discipline cases
- 4af7593/9f7255e/c67b8b8: HRBP training form, pipeline table, L&D tab redesign
- 13222f4–7520369: Overview rework, second KPI row, absent resolution chip

---

## Completed Phases

### PHASE 1 — Data Integrity ✅
nameMatch LOCAL (renderAccountHealth), attrition annualised, HC denominator,
normaliseCustomer(), candidate parser req_id join.

### PHASE 2 — Schema ✅ | PHASE 3 — Overview Rework ✅
See prior sessions. Key: monthly trend table, region cards, data freshness strip,
MTD Joiners, Absent Resolution chip, Offer→Join chip.

### HRBP Tab — Sessions 1–4 ✅ (08 Jun 2026)
Schema: hrbp_visit_plans, hrbp_er_cases, hrbp_retention_cases, hrbp_discipline_cases,
hrbp_site_visits extended. renderHRBP() → 4 sub-tabs. Journey Plan from hrbp_visit_plans.

### OD/WFH Enhancements ✅ (0d9450d) | ATE Advance Upload ✅ (ed35e5d)
OD: wfh_od_actions, multi-instance badge, HRBP action dropdown.
ATE Advance: processATEAdvance(), DBT Advance panel, loadATEAdvanceLog().

### Post-Upload Refresh ✅ (3c5a178) | Action Centre Filter Bar ✅ (e48a8f0)
postUploadRefresh(type) covers: super_emp, esep, fnf, od, attendance.
Action Centre: filter bar, clickable chips, table view for admin/exec.

### Resignation Tab — 5 Fixes ✅ (0119aff, 09 Jun 2026)
- days_to_lwd + days_since recomputed at render (not frozen at upload)
- Location column removed; grid 9→8 cols
- Exit reason shown as italic sub-line under account name
- Same resign+LWD date: LWD shows —, name shows red "No notice" badge
- Plat/Gold No Req chip: green "All covered ✓" when 0

### ATE Detection Fix ✅ (bf3f056, 09 Jun 2026)
isATE(r) reads Grade column (col CM), not EmpType. Corrects ATE count 27 → 65.
cwRows guard excludes ATE rows from FTC/Consultant/CONTRACT filter.

### ATE Conversion Tracking ✅ (c355d6d, 09 Jun 2026)
- ate_tracker: 3 new columns (fte_emp_code, fte_confirmed_at, fte_confirmed_by)
- renderATETab: fte_emp_code input shown when status = Conversion Initiated / Converted to FTE
- saveATEUpdate: saves fte_emp_code to Supabase
- processEmployeeMaster reconciliation: auto-confirms when fte_emp_code found in active FTE rows
- Missing 22xxxxxxx with no fte_emp_code → warning appended to action_notes
- loadATECases SELECT updated to include new columns
- UI badges: green (confirmed) / amber (pending) / red (code not entered)

### Account Health — Signal & Attrition Fixes ✅ (8f9edc6, 09 Jun 2026)
- Absent, FnF, attrition signals now filtered by acct.r region
- Attrition: live LWD computation from r.lwd date string (not frozen days_to_lwd)
- Attrition rate annualised: × (365/90). Label updated to show 'ann.'
- Last visit: no region field in _hrbpSVCache — pan-account; comment added

### CLAUDE.md Overwrite Prevention ✅
Sentinel: saveODAction|loadWFHODActions|_wfhOdActions|multiBadge (20 hits confirmed).
50-line delta is a trip wire for rewrites, not a hard cap.

---

## Pending Phases

### PHASE 4 — Account Health Tab (partial — signal fixes done)
Remaining:
- Position Lifecycle panel (resign → backfill req → sourcing → offer → joined)
- Composite health score (100pts): deployment gap 35, attrition 25, absent 20, HRBP visit 10, reqs 10
- RAG: red <50 / amber 50–70 / green >70
- Billing type badge: T&M red, MS blue, AMC amber
- T&M priority flag in TA aging; Plat/Gold aging panel (reqs >45d)

### PHASE 5 — HRBP Scorecard Redesign (tab structure done)
5-dimension computeHRBPScore() pending: D1 Workforce Stability 25, D2 Account Coverage 25,
D3 ER & Connect Quality 20, D4 TA Partnership 15, D5 Deployment Health 10.
Leaderboard: ALL roles visible. Calendar month cadence. Transparent scoring panels.

### PHASE 6 — Elah Parser + RMG Tab (URGENT — sunset July 1 2026)
processElahDeployment(wb): "Billed Resource" sheet → per-account HC, expiry, ramp-down.
File detection: resource_deployment / elah / deployment_detail.
RMG role: function_type='rmg' — Mohit to confirm names.

### PHASE 6B — ATE Management Module
- ATE Mix Dashboard: NATS = ROUND(region_fte_hc × 0.10) — dynamic from workforce_intel
- DBT Recovery Centre ✅ LIVE (25 May 2026)
- Bench-to-Billable Pipeline (rmg + hrops): pending

### PHASE 7 — Workforce Planning Tab | PHASE 8 — Tab Redesigns
Phase 8B priority: Past LWD escalation panel (211 employees, T&M billing impact).
Phase 8A: TA tab rebuild. Phase 8C-E: Prospective Joiners, OD, Absenteeism.

### PHASE 9 — Attrition History + Prediction
Requires Alex to upload 2-year eSep data first.

---

## Supabase Tables — Complete List

### HR CC Owns
- weekly_reports, weekly_submissions, user_profiles (27 rows)
- data_cache: keys: fnf, resignation, od, summary, account_hc, ta_summary,
  elah_deployment, elah_expiry_risk, elah_ramp_down, ate_dbt_summary, esep_codes
- absent_cases: 323 rows. Uses `.select('*')` — all cols including region.
- prospective_joiners, attrition_history (empty), bench_deployments (empty)
- ate_tracker: ATE cases. New cols added 09 Jun 2026:
  - fte_emp_code text — new FTE 11xxxxxxx code entered by HRBP at Conversion Initiated
  - fte_confirmed_at timestamptz — auto-set by processEmployeeMaster reconciliation
  - fte_confirmed_by text — 'system-auto' (upload) or HRBP name (manual)
- ate_advance_log: Sheetal's monthly advance + recovery log
- hrbp_site_visits (18 cols), hrbp_visit_plans, hrbp_er_cases,
  hrbp_retention_cases, hrbp_discipline_cases, hrbp_photos
- workforce_intel (id='current'), resignation_tracker, req_tracker, res_summary
- action_items, customer_accounts (150 rows, billing_type/contracted_hc/elah_hc/elah_name)
- account_positions_log (23 cols, 6 indexes, 0 rows)
- elah_deployment (empty), planned_leaves (empty)
- training_sessions (31 rows seeded), training_pipeline

### Shared With OPS360 — FLAG BEFORE TOUCHING
workforce_intel, absent_cases, resignation_tracker, prospective_joiners,
planned_leaves, employee_account_mapping, account_positions_log

### Postgres Functions
- normalise_account_name(raw_name text) → text (pg_trgm, similarity >0.4)

---

## ATE Management

### NATS Formula
ROUND(region_fte_hc × 0.10) per region — dynamic from workforce_intel.
Current headroom: West 78 / South 62 / North 41 / East 30 / Total 211.

### DBT Flow
DBT goes directly govt → employee bank. CMS exposure = advance paid − recovered.
Sheetal logs advance (Sheet 1) and recovery after DBT intimation (Sheet 2).

### ATE Deployment Status (auto-classified 24 May 2026)
West: 4 Billable / 27 Training · East: 2 Billable / 18 Training
North: 0 Billable / 9 Training · South: 0 Billable / 4 Training

### ate_tracker Reconciliation — 09 Jun 2026
4 exited ATEs set Active → Separated (08-Jun master):
22007523 Rohit Nangare (FnF InProcess, DOL 04-Jun), 22007561 Akshat Patil (FnF Locked, DOL 17-Mar),
22007566 Rishabh Swarankar (FnF InProcess, DOL 09-Jun), 22007571 Jyoti Jaiswar (FnF InProcess, DOL 13-May).
DO NOT TOUCH: 22007479 Ashish Raj (Conversion Initiated ✓), 22003768 Divya Goel (see Pending Actions).

---

## Upload Parsers

### ZingHR Super Employee Master (Ramesh — weekly)
- File detection: cmsitn prefix
- Populates: _acctHC (composite ACCOUNTNAME|||REGION keys + flat keys), workforce_intel, ate_tracker
- **ATE identified by Grade (col CM) = 'ATE'** via isATE(r) — NOT EmpType (col CK)
- FTC / Consultant / CONTRACT remain EmpType-based
- ATE reconciliation pass: auto-confirms fte_emp_code against active FTE rows; flags missing ATEs
- 09-Jun numbers: activeRows 2,709 · WI HC 2,749 · 380 HC cache keys · ATE 65 · FTC 106 · Consultant 324 · Contract 89

### ZingHR eSep Transaction (Ramesh — monthly)
Populates: CMS_RES_DATA, workforce_intel attrition. HC denominator: hc_snap[mon] || 2830.
Scoped to 2026 exits only. Excludes FnF Locked and revoked from CMS_RES_DATA (intentional).

### ZingHR Req TAT / Candidate Transactional (Ramesh — weekly)
normaliseCustomer() on every customer field. billing_type from customer_accounts.
_reqMap from TA_ACTIVE_REQS. _unlinked flag on unmatched candidates.

### ATE Advance Upload (Sheetal — monthly)
File detection: ate_advance or ate_dbt. Two-sheet: "Advance Paid" + "Recovery".
Header auto-detect (scans first 5 rows). processATEAdvance(wb) — LIVE.

### Elah Resource Deployment (Ramesh — weekly, until July 1 sunset)
Parser: processElahDeployment(wb) — NOT YET BUILT (Phase 6 — URGENT).

---

## Account Positions Log — Workflow
RMG enters zinghr_req_id in HR CC → dashboard joins: resignation ↔ req ↔ candidates ↔ offer ↔ joiner.
Key cols: zinghr_req_id, linked_resignation_id, linked_absent_case_id, days_vacant (GENERATED STORED), status.

---

## Billing Types — Key Reference
32 T&M / 113 MS / 2 AMC. T&M accounts (deployment gap = direct billing loss):
IHCL, Reliance Corporate IT Park, ONGC, Torrent Power, Poonawalla Fincorp, Sutherland,
Religare, Titan, Cadence, Bajaj Finance, Syngene, Dixon, SMS India, Vitech, Clix Capital.

---

## Workforce Intelligence — Current Numbers
- Active HC: 2,749 (Super Emp Master, 09 Jun 2026)
- Account HC cache (activeRows): 2,709 · Gap of 40 = FnF Initiated/transitional state
- DO NOT use active_hc_mar for display
- Region HC: North 502 / South 660 / East 502 / West 1,088 (pending 09-Jun confirmation)
- Attrition (annualised): Jan 34.6% · Feb 20.5% · Mar 19.9% · Apr 1.2% · May 19.1%

---

## User Profiles — 27 Users

| Name | Role | Function Type | Region |
|---|---|---|---|
| Alex Augustine | admin | admin | Pan India |
| Shankar Sengupta | executive | executive | Pan India |
| Vinay Kumar | executive | executive | Pan India |
| Amit Dasgupta | executive | executive | Pan India |
| Neeraj Sharma | executive | executive | Pan India |
| Kesavan | executive | hrbp | Pan India |
| Shambhavi Prathamesh Joshi | hrbp | hrbp | West |
| Somasri Sukumar Samanta | hrbp | hrbp | West |
| Priya Paul | hrbp | hrbp | South |
| Abhishek Singh | hrbp | hrbp | North |
| Surajit Sen | hrbp | hrbp | East |
| Sheetal Sadashiv Pachangane | hrops | hrops | Pan India |
| Mohit Kumar | hrops | payroll | Pan India |
| R Ramesh | hrops | payroll | Pan India |
| Deepak Kumar Shetty | ld | ld | Pan India |
| Ashok B C | facilities | facilities | Pan India |
| Ajith Inguva | ta | ta | Pan India |
| Caral Anitha Dsouza | ta | ta | South |
| Dhananjay Kumar Singh | ta | ta | West |
| Juee Nilesh Patil | ta | ta | West |
| Kumari Puja | ta | ta | West |
| Mousumi Priyadarsini Behera | ta | ta | South |
| Neha Kaur Sammi | ta | ta | Pan India |
| Nikita Yadav | ta | ta | West |
| Nilam Sunil Patil | ta | ta | West |
| Pijush Dutta | ta | ta | East |
| Salma Saifi | ta | ta | North |

TA users corrected from role='hrbp' → role='ta' on 24 May 2026.
Recruiter scorecard must filter by function_type='ta'.

---

## HRBP Region Assignments
Abhishek Singh: North · Surajit Sen: East · Priya Paul: South
Shambhavi Prathamesh Joshi: West · Somasri Sukumar Samanta: West

HRBP_REGION lookup objects (two — keep in sync):
1. HRBP_REGION const (full name keys, line ~1654)
2. HRBP_REGION_MAP const (line ~4336, first-name based)

---

## Absent Cases — Status Values (exact as stored)
"Uncontacted" | "OD Not Regularised" | "Absconding — eSep Initiated"
"Resolved — Resigned" | "Resolved — Returned" | "Missed Punch"
"On Leave (Unplanned)" | "Resignation — HR Chain Pending"
"Under Investigation" | "Warning Letter Issued" | "Medical / Emergency"

Resolution filter must use indexOf('resolved') not includes() — em-dash may cause Unicode mismatch.

---

## OPS360 Relationship
Supabase shared project. OPS360 reads: customer_accounts, account_positions_log, ate_tracker,
bench_deployments, elah_deployment, attrition_history, normalise_account_name().
Shared read/write: workforce_intel, absent_cases, resignation_tracker, planned_leaves.

---

## Pending People Actions

| Person | Action | When |
|---|---|---|
| Abhishek Singh | Divya Goel (22003768): tracker=Separated (13-Apr); 09-Jun master=Existing+Grade ATE. Confirm rejoin (→ reset Active) vs stale ZingHR tag (→ Ramesh corrects). | ASAP |
| Ramesh | Confirm actual ATE headcount in ZingHR (Grade=ATE, Existing/NewJoinee) — 27 in today's upload vs 65 active in ate_tracker. Needed before delete-stale logic added. | This week |
| Ramesh | Upload complete May eSep to reconcile April exits | Pending |
| Ramesh | Clarify HC scope: FTE only or all types in Super Emp extract | Pending |
| Ramesh | Confirm Retainer employees (293 in Elah) in Super Emp extract | This week |
| Ramesh | Confirm planned_leaves filename prefix | Pending |
| Alex | Add Tier + Billing Type + Contract End Date + Contracted HC in ZingHR customer master | Before June 30 |
| Alex | Add Billing Type + Employee Category (FTE/ATE/Retainer) in ZingHR recruitment module | Before June 30 |
| Alex | Upload 2-year eSep data for attrition_history | Phase 9 |
| Kesavan | Fill contracted_hc for Plat/Gold via Admin UI | This week |
| Mohit | Confirm RMG team member names for user_profiles | Pending |
| Sheetal | Use ATE_Advance_Upload_Template.xlsx from June 2026 | June 2026 |
| RMG team | Enter zinghr_req_id in HR CC for each backfill/new position | After Phase 6 builds |

---

## Known Debt — Carry Forward
- ATE stale records: ate_tracker has 65 Active but today's Super Emp has only 27. Parser upserts only, never deletes. Cleanup awaiting Ramesh headcount confirmation. Same pattern as contract_workforce (which already has cleanup step).
- contract_workforce: ~50 ATE rows that should not be there (~14 duplicate ate_tracker + ~36 orphans). Cleanup pending.
- Resignation tab Past LWD: 211 already-left employees mixed into main table. Design pending: collapsed section vs separate escalation panel (Phase 8B).
- Account Health last visit: _hrbpSVCache has no region field — last visit date is pan-account. Remains until site visit data captures region at entry.
- R1 filter: 'Approved' button matches r1_status === 'Accepted'. Low risk but should be hardened to match both.
- owner field in Action Centre task cards: confirm renders inline.
- WI hardcoded fallback line ~845: stale values. Self-corrects on uploads.
- CEAT double-space duplicate in ZingHR: "CEAT LIMITED" + "CEAT  LIMITED".
- LIC naming: "LIC (EAST)" vs "LIC ( EAST )" — standardise in ZingHR.
- Duplicate switchTab: line ~3564 dead code, line ~4213 live. Clean up in future session.
- planned_leaves parser: filename prefix TBC from Ramesh.

---

## When to Update This File
UPDATE AFTER: schema change, phase complete, design decision confirmed, new upload parser, role change, bug fixed.
DO NOT update for: work in progress, items being debated, analysis outputs.

---

## Build Sequence — What's Next

### COMPLETE THIS SESSION ✅
- Resignation tab 5 fixes (0119aff)
- ATE detection Grade fix (bf3f056)
- ATE conversion tracking + fte_emp_code (c355d6d)
- Account Health region filter + live LWD + annualised attrition (8f9edc6)
- ate_tracker manual reconciliation (4 rows, Supabase only)

### NEXT SESSION
- ATE stale record cleanup (awaiting Ramesh headcount confirm — see Pending Actions)
- Account Health Phase 4: Position Lifecycle panel + composite score
- Resignation tab 8B: Past LWD escalation panel (design decision needed)
- Phase 6: Elah parser — URGENT before July 1 sunset

### FOLLOWING SESSIONS
- Phase 5: HRBP scorecard 5-dimension redesign
- Phase 6B: Bench-to-Billable Pipeline
- Phase 7: Workforce Planning tab
- Phase 8: TA / Prospective Joiners / OD / Absenteeism redesigns
- Phase 9: Attrition history + prediction

### DEFERRED — no decision yet
- Email/notification system (CMS Gmail MCP vs SMTP relay vs in-dashboard)
- HRBP full scorecard transparency: confirmed all HRBPs see all scores
- Manage Customer Accounts tab: pending ZingHR attribute decisions
