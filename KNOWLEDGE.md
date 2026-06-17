# CMS HR Ops Command Centre — Project Knowledge
**Version:** 2.8.0 | **Last updated:** 17 Jun 2026 — RMG Workspace, TA Pipeline refresh, Workforce Module design

---

## What This Is
Single-file HTML HR Operations dashboard for CMS IT Services.
Used daily by HR team, HRBPs, Talent Acquisition, RMG, and executive leadership.

Covers: Overview (CXO command strip), TA pipeline, HRBP connects and scorecards,
HR Ops / payroll, L&D, Risks, Workforce Intelligence, Resignation tracker, OD/WFH
compliance, FnF tracker, Long Absenteeism, ATE tracker, Prospective Joiners,
RMG Workspace, Action Centre, Admin/Settings.

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

### Single-File Architecture Assessment (17 Jun 2026)
Assessed as sound for next 6 months. Risks are in specific large functions, not architecture itself.
Vite/TypeScript migration deferred — not needed now, revisit only if file becomes unmaintainable.

Function size thresholds (watch list):
- renderHRBP: 523 lines — highest risk for accidental edits
- renderATETab: 370 lines
- renderTA_Commitments: 357 lines
- renderWorkforce: 363 lines

Rule: functions over 300 lines should be split into orchestrator + sub-functions when next touched.

### Workflow: Cowork + Claude.ai
- Planning / diagnosis / prompt writing → Claude.ai chat
- Build sessions → Cowork + Claude Code
- Review / KNOWLEDGE.md → Claude.ai chat
- "Cowork builds, Claude.ai steers"

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
NEVER gate on function_type alone — it blocks admins. Always: `role==='admin' OR function_type==='X'`

### L&D Tab Architecture (redesigned 05 Jun 2026)
`renderLD(w, pw)` returns a static shell only. `loadLDTab(monthKey)` drives all data loading.
- Called by: render() (100ms), switchTab('ld'), ldMonthSel onChange
- `_loadLDSessions(mk)`: training_sessions → KPI chips + table
- `_loadLDPipeline(mk)`: training_pipeline (Planned/Confirmed) → pipeline table
- Month selector id: `ldMonthSel`
- Constants: `LD_TRAINERS` (7 names) + `LD_CATEGORIES` (6 types)

### Duplicate switchTab
Two definitions exist:
- Line ~3845: dead code (do not wire new tabs here)
- Line ~4608: LIVE — all new tab wiring goes here
Clean up in a future session.

### HRBP Site Visit Cache
`_hrbpSVCache` holds ALL site visits — not week-restricted. Journey Plan panel and
scorecard both read from this cache. loadHRBPSiteVisitsForTab() triggers deferred
re-render (50ms setTimeout) after populating the cache.

### Postgres: No CURRENT_DATE in Stored Generated Columns
IMMUTABLE requirement — Postgres rejects CURRENT_DATE / now() in stored generated columns.
- hrbp_er_cases.days_open + sla_breached: computed by trigger `_hec_compute_days`.
- account_positions_log.days_vacant: GENERATED STORED (expression does not reference today).

---

## Key Learnings

### ATE Conversion Emp Code Problem
ATEs have 22xxxxxxx emp codes. On FTE conversion ZingHR issues a new 11xxxxxxx code.
**Solution**: HRBP enters new FTE emp code in `fte_emp_code` at Conversion Initiated stage.
`processEmployeeMaster` reconciliation auto-confirms when `fte_emp_code` appears in active non-ATE rows.

### getEmpType() / ATE Detection
ATE identified by Grade (col CM) = 'ATE' via `isATE(r)` — NOT EmpType (col CK).
This corrects the count from 27 (EmpType) to 65 (Grade). FTC/Consultant/CONTRACT remain EmpType-based.

### Account Health — Attrition Formula
Rate is annualised: `exits / activeHC × (365/90) × 100`.
Thresholds 3% / 8% / 15% are annual rates. Label shows '(90d) · ann.'.
Exit count uses live LWD computation from `r.lwd` date string — NOT the frozen `days_to_lwd` integer.

### Account Health — Region Filter Pattern
- `_absentCases`: region field present ✅ | `FNF_DATA`: region field ✅ | `CMS_RES_DATA`: region field ✅
- `_hrbpSVCache` (last visit): NO region field — pan-account only ⚠
Filter pattern: `if (acct.r && r.region && r.region !== acct.r) return false;`

### Frozen days_to_lwd — Root Cause
`days_to_lwd` is baked into CMS_RES_DATA at upload time. Always recompute from `r.lwd` date string at render/compute time.

### HC Scope — KEY PRINCIPLES (confirmed 17 Jun 2026)
- Consultant and FTC are employment type labels only.
- For all HC reporting, workforce planning, account deployment, and backfill — treated identically to permanent FTE.
- Contractual workforce (HC breakdown) = Consultant + FTC + Contract + ATE.
- ATE shown separately only in ATE Tracker module.
- FTC/Consultant ARE included in total HC (both account_hc cache and WI regional totals). Not excluded anywhere.
- Total HC 2,621 includes all emp types.
- HC Budget from RMG covers total HC (FTE + FTC + Consultant). ATE grouped with contractual workforce for HC breakdown.

### Elah-as-Budget-Ceiling Principle (RMG rule)
Every billable req must trace to an Elah demand line. Net-new positions not in Elah are exceptions.
Only non-billable roles may sit outside Elah.
A position is open only if: Status != Closed AND Balance > 0 AND Billable = Yes.

### ZingHR Recruitment Data Issues (from 17 Jun 2026 analysis)
- TAT Customer Name blank on 758 of 774 rows — account name trapped in requisition title field
- 217 reqs filled-but-never-closed in ZingHR
- Raw open count: 405 → Reconciled open count: 182 (applying Elah-as-budget-ceiling)
- 66 critical resignations with no backfill raised
- 73 seats above Elah demand line need verification (Indian Hotels, SBI General, Avenue Supermarts)

### RMG Reconciliation Flag Logic
- RED — elah_demand_id is null/empty OR in_elah = false
- AMBER — position_type contains 'Backfill' AND backfill_emp_code is null/empty
- GREEN — elah_demand_id set AND (not a backfill OR backfill_emp_code set)

---

## Current File State — 17 Jun 2026
- Line count: ~12,794 (after commits aa507d7 + 40fd550)
- Latest commits:
  - 40fd550: feat: TA Pipeline — billing type filter, filled-unclosed banner, backfill emp column
  - aa507d7: feat: RMG Workspace — reconciliation grid, Add Position form, role-gated view/edit
  - c5f46bb: docs: KNOWLEDGE.md updated — session 09 Jun 2026
  - 8f9edc6: fix: Account Health — region filter on signals, live LWD computation, annualised attrition rate
  - c355d6d: feat: ATE conversion tracking — fte_emp_code field, Super Emp Master reconciliation, UI badges
  - 9f58ac4: docs: KNOWLEDGE.md v2.6.0 — 09 Jun 2026 session summary

### Commits — 17 Jun 2026
- 40fd550: TA Pipeline — billing type filter, filled-unclosed banner, backfill emp column
- aa507d7: RMG Workspace — reconciliation grid, Add Position form, role-gated view/edit

### Commits — 09 Jun 2026 (summary)
- 8f9edc6: Account Health — region filter, live LWD, annualised attrition
- c355d6d: ATE conversion tracking — fte_emp_code, reconciliation, UI badges
- 0119aff: Resignation tab — recompute days, exit reason, no-notice badge
- bf3f056: ATE detection — Grade (CM) not EmpType (CK)

### Commits — May–08 Jun 2026 (summary)
- e48a8f0: Action Centre — clickable KPI chips, filter bar, table view
- 3c5a178: Post-upload tab refresh — postUploadRefresh()
- 0d9450d: OD/WFH — wfh_od_actions, multi-instance badge, HRBP action dropdown
- ed35e5d: ATE Advance upload — DBT Advance panel, processATEAdvance()
- 4b9d846: HRBP schema — hrbp_visit_plans, hrbp_er_cases, retention/discipline
- Earlier: Overview rework, L&D tab redesign, HRBP training form, pipeline table

---

## Completed Phases

### PHASE 1 — Data Integrity ✅
nameMatch LOCAL (renderAccountHealth), attrition annualised, HC denominator, normaliseCustomer().

### PHASE 2 — Schema ✅ | PHASE 3 — Overview Rework ✅
Monthly trend table, region cards, data freshness strip, MTD Joiners, Absent Resolution chip, Offer→Join chip.

### HRBP Tab ✅ (08 Jun 2026)
Schema: hrbp_visit_plans, hrbp_er_cases, hrbp_retention_cases, hrbp_discipline_cases. 4 sub-tabs.

### OD/WFH Enhancements ✅ | ATE Advance Upload ✅ | Post-Upload Refresh ✅ | Action Centre Filter Bar ✅

### Resignation Tab Fixes ✅ (09 Jun 2026)
days_to_lwd recomputed at render; location column removed; exit reason shown; no-notice badge; Plat/Gold chip fix.

### ATE Detection Fix ✅ | ATE Conversion Tracking ✅ (09 Jun 2026)

### Account Health Signal & Attrition Fixes ✅ (09 Jun 2026)

### RMG Workspace Tab ✅ (aa507d7 — 17 Jun 2026)
- Tab id: rmg, icon: 🔗, label: RMG Workspace
- Access: admin/rmg = full edit; hrbp/ta/hrops = read-only; others = hidden
- HRBP auto-filter: region auto-set to _sbRole.region on load
- KPI strip: Total / No Elah ID / RED / AMBER / GREEN counts
- Filter bar: Region (All/North/South/East/West) / Status (All/Approved/Closed) / Flag (All/RED/AMBER/GREEN)
- Grid: 9 columns — Req ID, Elah ID (editable), Account, Region, Position Type (dropdown), Backfill Emp (editable), Status, Age (d), Flag
- Add Position form: 8 fields, accounts from customer_accounts; required: Req ID, Account, Position Type
- saveRMGField: optimistic update, revert on error, auto-sets in_elah when elah_demand_id updated
- showToast added globally at line 8795
- 10 new functions: renderRMGShell, loadRMGTab, _renderRMGGrid, _renderRMGAddForm, toggleRMGAddPanel, rmgSyncRegion, submitRMGNewPosition, _rmgKpi, _rmgReconFlag, saveRMGField
- Live switchTab wired at line 4668 (NOT dead ~3845)

### TA Pipeline Refresh ✅ (40fd550 — 17 Jun 2026)
- _taBilling global added (All / T&M / MS)
- Billing filter applied to data array; filter bar renders after Type filter
- filledUnclosed banner: warns when balance=0 but req still Open; renders between agHTML and pgAlert
- Req table expanded to 9 columns: added Backfill (separation_emp_code); amber if set, muted if not
- Grid template: 52px 155px 150px 60px 42px 45px 32px 75px 80px

---

## Pending Phases

### PHASE 4 — Account Health Tab (partial — signal fixes done)
Remaining: Position Lifecycle panel, composite health score (100pts), RAG <50/<70/>70, billing type badge.

### PHASE 5 — HRBP Scorecard Redesign
5-dimension computeHRBPScore() pending: Workforce Stability 25, Account Coverage 25, ER & Connect Quality 20, TA Partnership 15, Deployment Health 10.

### PHASE 6A — Resignation Backfill Trigger (next to build)
eSep upload flags resignations needing backfill. RMG reviews queue: Raise Req / Deploy from Bench / No Replacement Needed.
Designation enrichment: join contract_workforce by emp_code at eSep parse time.
Claude Code prompt written, not yet committed.

### PHASE 6B — ATE Management Module
- DBT Recovery Centre ✅ LIVE
- Bench-to-Billable Pipeline (rmg + hrops): pending

### PHASE 7 — HC Budget Module
Elah Pan India file = RMG's current-state deployed HC (Count of Emp Code, Billability=Billed).
Elah total: 2,529 vs ZingHR: 2,621 — gap of 92 employees. 11 employees multi-account tagged.
Bench in Elah: 146 (Field Services + Backup Pool).
Paused: waiting for reconciled Elah/ZingHR master (469 mapping exceptions to resolve).
Schema: hc_budget table to be created when data is clean.

### PHASE 8 — Tab Redesigns
Phase 8B priority: Past LWD escalation panel (211 employees, T&M billing impact).
Phase 8A: TA tab rebuild. Phase 8C-E: Prospective Joiners, OD, Absenteeism.

### PHASE 9 — Attrition Prediction
attrition_history table created, 0 rows. Blocked: needs Jan–May 2026 eSep history from Ramesh.

### On the Horizon
- Transfer Detection: week-on-week customer tag change in Super Emp Master — low-effort parser, not yet built
- Customer Contract Expiry: 3 new cols on customer_accounts (contract_end_date, renewal_status, contract_notes) — no data yet

---

## Build Sequence — What's Next

### NEXT (in order)
1. **Designation fix** — contract_workforce parser + eSep enrichment join (prompt written, not yet committed)
2. **Add RMG user_profiles rows** — after Mohit confirms Pruthvi SS + Chander Mohan emails
3. **Phase 6A — Resignation backfill trigger** — design complete, Claude Code prompt not yet written
4. **HC Budget module** — after Elah/ZingHR reconciliation complete (469 exceptions)
5. **ATE stale record cleanup** — after Ramesh confirms ZingHR headcount
6. **Resignation Past LWD section** — design decision pending (collapsed vs escalation panel)
7. **Account Health Phase 4** — Position Lifecycle panel + composite score
8. **Attrition Prediction** — after Ramesh loads Jan–May 2026 eSep historical data

---

## Supabase Tables — Complete List

### HR CC Owns
- weekly_reports, weekly_submissions, user_profiles (27 rows)
- data_cache: keys: fnf, resignation, od, summary, account_hc, ta_summary, elah_deployment, elah_expiry_risk, elah_ramp_down, ate_dbt_summary, esep_codes
- absent_cases: 323 rows. Uses `.select('*')` — all cols including region.
- prospective_joiners, attrition_history (empty), bench_deployments (empty)
- ate_tracker: ATE cases. Cols: fte_emp_code, fte_confirmed_at, fte_confirmed_by (added 09 Jun 2026)
- ate_advance_log: Sheetal's monthly advance + recovery log
- hrbp_site_visits (18 cols), hrbp_visit_plans, hrbp_er_cases, hrbp_retention_cases, hrbp_discipline_cases, hrbp_photos
- workforce_intel (id='current'), resignation_tracker, req_tracker, res_summary
- action_items, customer_accounts (150 rows, billing_type/contracted_hc/elah_hc/elah_name)
- account_positions_log (see schema below)
- elah_deployment (empty), planned_leaves (empty)
- training_sessions (31 rows seeded), training_pipeline

### account_positions_log Schema (17 Jun 2026)
278 rows seeded. Migration: rmg_positions_log_columns_and_rls (20260617).
RLS: 4 clean policies (aplog_select/insert/update/delete).

Key columns:
- zinghr_req_id, account_name, region, position_type, designation
- elah_demand_id text — Elah demand line ID, entered by RMG
- demand_region text — region per Elah
- emp_category text — FTE / ATE / Retainer
- backfill_emp_code text — emp code of departing employee
- backfill_emp_name text — name of departing employee
- req_status text — Approved / Closed (from ZingHR)
- req_age_days integer — days since req was raised
- in_elah boolean — true when elah_demand_id is set/verified
- days_vacant (GENERATED STORED), updated_by, updated_at, created_at

Seed data breakdown:
- position_type: Backfill — Resignation (188), New Position (67), Backfill — Internal Movement (22), Redeployment (1)
- req_status: Approved (208), Closed (66), null (4)
- regions: West (163), South (90), null (24), East (1)
- in_elah: true (249), not in elah (29)

### Shared With OPS360 — FLAG BEFORE TOUCHING
workforce_intel, absent_cases, resignation_tracker, prospective_joiners,
planned_leaves, employee_account_mapping, account_positions_log

### Postgres Functions
- normalise_account_name(raw_name text) → text (pg_trgm, similarity >0.4)

---

## Upload Parsers

### ZingHR Super Employee Master (Ramesh — weekly)
- File detection: cmsitn prefix
- Populates: _acctHC (composite ACCOUNTNAME|||REGION keys + flat keys), workforce_intel, ate_tracker
- ATE identified by Grade (col CM) = 'ATE' via isATE(r) — NOT EmpType (col CK)
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

## ATE Management

### NATS Formula
ROUND(region_fte_hc × 0.10) per region — dynamic from workforce_intel.

### DBT Flow
DBT goes directly govt → employee bank. CMS exposure = advance paid − recovered.
Sheetal logs advance (Sheet 1) and recovery after DBT intimation (Sheet 2).

### ate_tracker Reconciliation — 09 Jun 2026
4 exited ATEs set Active → Separated. DO NOT TOUCH: 22007479 Ashish Raj (Conversion Initiated ✓).
22003768 Divya Goel: tracker=Separated (13-Apr); 09-Jun master=Existing+Grade ATE. Confirm rejoin vs stale ZingHR tag.

---

## Billing Types — Key Reference
32 T&M / 113 MS / 2 AMC. T&M accounts (deployment gap = direct billing loss):
IHCL, Reliance Corporate IT Park, ONGC, Torrent Power, Poonawalla Fincorp, Sutherland,
Religare, Titan, Cadence, Bajaj Finance, Syngene, Dixon, SMS India, Vitech, Clix Capital.

---

## Workforce Intelligence — Current Numbers
- Active HC: 2,749 (Super Emp Master, 09 Jun 2026)
- Account HC cache (activeRows): 2,709 · Gap of 40 = FnF Initiated/transitional
- Region HC: North 502 / South 660 / East 502 / West 1,088
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
RMG users (Pruthvi SS + Chander Mohan): user_profiles rows pending — Mohit to confirm emails.

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

## Pending People Actions

| Person | Action | When |
|---|---|---|
| Mohit | Pruthvi SS and Chander Mohan email addresses for rmg user_profiles rows | This week |
| Ramesh | Confirm actual ATE headcount in ZingHR (61 active in tracker, last upload showed 27) | ASAP |
| Ramesh | Upload Jan–May 2026 eSep historical data for attrition_history population | Pending |
| Alex/Team | Elah/ZingHR reconciled master (469 mapping exceptions to resolve) before HC Budget module | Ongoing |
| Alex | Add Tier + Billing Type + Contract End Date + Contracted HC in ZingHR customer master | Before June 30 |
| Alex | Add Billing Type + Employee Category (FTE/ATE/Retainer) in ZingHR recruitment module | Before June 30 |
| Kesavan | Fill contracted_hc for Plat/Gold via Admin UI | This week |
| Sheetal | Use ATE_Advance_Upload_Template.xlsx from June 2026 | June 2026 |
| RMG team | Enter zinghr_req_id in HR CC for each backfill/new position | After Phase 6A builds |

---

## Known Debt — Carry Forward
- ATE stale records: ate_tracker has 61 Active but Super Emp shows 27. Cleanup awaiting Ramesh headcount confirmation.
- contract_workforce: ~50 ATE rows that should not be there. Cleanup pending.
- Resignation tab Past LWD: 211 already-left employees in main table. Design pending: collapsed vs escalation panel (Phase 8B).
- Account Health last visit: _hrbpSVCache has no region field — last visit date is pan-account.
- Duplicate switchTab: line ~3564 dead code, line ~4213 live. Clean up in future session.
- planned_leaves parser: filename prefix TBC from Ramesh.
- CEAT double-space duplicate in ZingHR: "CEAT LIMITED" + "CEAT  LIMITED".
- LIC naming: "LIC (EAST)" vs "LIC ( EAST )" — standardise in ZingHR.
- WI hardcoded fallback line ~845: stale values. Self-corrects on uploads.

---

## OPS360 Relationship
Supabase shared project. OPS360 reads: customer_accounts, account_positions_log, ate_tracker,
bench_deployments, elah_deployment, attrition_history, normalise_account_name().
Shared read/write: workforce_intel, absent_cases, resignation_tracker, planned_leaves.

---

## When to Update This File
UPDATE AFTER: schema change, phase complete, design decision confirmed, new upload parser, role change, bug fixed.
DO NOT update for: work in progress, items being debated, analysis outputs.
