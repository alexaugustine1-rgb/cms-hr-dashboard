# CMS HR Ops Command Centre — Project Knowledge
**Version:** 2.4 | **Last updated:** 05 Jun 2026 — HRBP schema (Session 1 of 4)

---

## What This Is
Single-file HTML HR Operations dashboard for CMS IT Services.
Used daily by HR team, HRBPs, Talent Acquisition, RMG, and
executive leadership.

Covers: Overview (CXO command strip), TA pipeline, HRBP connects
and scorecards, HR Ops / payroll, L&D, Risks, Workforce Intelligence,
Resignation tracker, OD/WFH compliance, FnF tracker, Long Absenteeism,
ATE tracker, Prospective Joiners, Action Centre, Admin/Settings.

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
- node --check after EVERY edit.
  Extract scripts to temp .js first (Node v24 won't check .html directly).
  No output = clean pass.
- Node.js path: C:\Program Files\nodejs\node.exe (confirmed installed)

---

## Architecture Notes

### Absent Resolution Chip
Reads `window._absResolvedCount` and `window._absTotalCount` — set by two separate
queries in sbBoot() (ILIKE 'resolved%' count + total head count).
NOT derived from `_absentCases` (that array excludes resolved cases by design).
Fallback: `_absentCases.length + _resolvedAbs` if counts not ready.
Do NOT modify loadAbsentCases() query — it intentionally excludes resolved.

### innerHTML Does NOT Execute `<script>` Tags
Browsers ignore `<script>` blocks injected via innerHTML.
Any function that must be callable from onclick handlers in dynamically rendered HTML
MUST be declared as a true global (at top-level script scope), not inside an innerHTML string.
Confirmed crash pattern: hscToggle was declared inside innerHTML `<script>` — moved to global scope (25 May 2026).

### Role Gate Pattern — Use Both Fields
`_sbRole.role` and `_sbRole.function_type` are DIFFERENT fields.
Role gate checks must read both:
```javascript
var role   = _sbRole ? (_sbRole.role   || '') : '';
var fnType = _sbRole ? (_sbRole.function_type || '') : '';
```
Comparing only function_type blocks admin users (function_type='admin', not 'hrops').

### L&D Tab Architecture (redesigned 05 Jun 2026)
`renderLD(w, pw)` returns a static shell only — 5 KPI chips show "—/Loading…",
two panel divs (`ldSessionsPanel`, `ldPipelinePanel`) show "Loading…" placeholders.

`loadLDTab(monthKey)` drives all data loading:
- Called by: render() (100ms after every full render), switchTab('ld'), ldMonthSel onChange
- Fires both `_loadLDSessions(mk)` and `_loadLDPipeline(mk)` in parallel
- Month selector id: `ldMonthSel` (was trainerMonthSel — removed)

`_loadLDSessions(mk)`:
- Queries training_sessions filtered by month_key
- Updates ld-kpi-sessions + ld-kpi-pax KPI chips
- Renders: category strip + 9-col unified table + summary bar

`_loadLDPipeline(mk)`:
- Queries training_pipeline WHERE status IN ('Planned','Confirmed')
- Also queries all rows for Plan vs Actual calc
- Updates ld-kpi-pipeline + ld-kpi-planvact KPI chips + ld-pipeline-badge
- Renders 9-col pipeline table with status/priority colour coding

Constants: `LD_TRAINERS` (7 names) + `LD_CATEGORIES` (6 types)
Helpers: `_ldTh(label, align)`, `_fmtMonthKey(mk)`, `weekLabelToMonthKey(label)`

### L&D Form Data Flow
LD form (Deepak submits weekly):
- f_ld_pipeline (structured grid) → training_pipeline on submit
- f_ld_upcoming (hidden, kept for compat) — no longer shown in form UI
- ld_sessions + ld_coverage still written to weekly_submissions (KPI cards NOT
  driven by these anymore — now driven by training_sessions aggregate)

HRBP form (each HRBP submits weekly):
- Section 1 "Training Conducted": f_hrbp_training_entries → training_sessions
  (writes: trainer=submitter_name, category=Behavioural, mode=Classroom)
- Section 2 "Training Planned": f_hrbp_pipeline → training_pipeline
  (writes: status=Planned, priority=Medium by default)

### HRBP Site Visit Cache
`_hrbpSVCache` holds ALL site visits from hrbp_site_visits — not week-restricted.
Journey Plan panel and scorecard both read from this cache.
loadHRBPSiteVisitsForTab() triggers a deferred re-render of the Journey Plan panel
(50ms setTimeout) after populating the cache — needed because renderHRBP() fires
before the async cache is ready.

---

## Known Crash Patterns — Check Every Edit
Any of these crashes login with "ReferenceError: sbLogin is not defined":
- Orphaned em-dash in string concatenation
- await inside a non-async function
- Block-scoped function declarations inside try blocks
- const or let declared inside try blocks
When node --check reports an error: print 3 lines above and 3 below
before attempting any fix.

---

## Current File State — 05 Jun 2026
- Line count: 11,250 lines (latest confirmed after commit c67b8b8)
- Latest commit: c67b8b8
- Local path: D:\Dropbox\CMS_IT_Services\Claude_Projects\Claude_HR_Automation\
  HR_OPS_COMMAND\cms-hr-dashboard\index.html

### Commits — 24 May 2026
- 13222f4: Overview rework (WH strip removed, trend table, region cards)
- c739d46: Second KPI row (4 chips added)
- 1ac9369: Trend table HC/joiners fix + week resolution to latest week
- 90d1d1b: Absent resolution fix (indexOf), Offer→Join from _taSummaryCache,
           week selector hidden on Overview, data freshness strip added
- 7407319: MoM future-month guard, MTD Joiners card, Absent deferred re-render fix
- 7520369: Absent resolution chip — separate count query in sbBoot

### Commits — 05 Jun 2026 (schema only — no index.html change)
- HRBP schema Session 1: hrbp_site_visits extended (6 cols), hrbp_visit_plans created,
  hrbp_er_cases created (trigger-based days_open/sla_breached),
  hrbp_retention_cases + hrbp_discipline_cases created,
  hrbp_photos.site_visit_id FK added

### Commits — 05 Jun 2026 (index.html)
- 4af7593: Feat: HRBP training form → training_sessions, category breakdown in trainer panel
- 9f7255e: Feat: training_pipeline table + structured pipeline forms (LD+HRBP), weekLabelToMonthKey helper
- c67b8b8: Feat: Full L&D tab redesign — loadLDTab() unified sessions + pipeline panels, KPI async

### Commits — 04 Jun 2026
- 2e927b3: Feat: Trainer Activity panel — training_sessions table, Maaz Khan + Deepak sessions seeded
- 98d6fad: Fix: defer loadTrainerActivity 50ms + surface res.error explicitly
- a11002f: Fix: auto-load trainer activity in render() — tab content resets on week-change
- 5af6dec: Fix: loadTrainerActivity wired into live switchTab (line 4170)

### Commits — 25 May 2026
- 2e230a5: Fix: DBT Recovery Centre visible to admin/executive (role gate + CSS vars)
- b6427de: Feat: Region filter for DBT Recovery Centre
- a6de0f5: Fix DBT Recovery Centre: dates DD/MM/YYYY, emp status badge, cleaner traffic lights
- d91b49c: Upload access for Sheetal: FnF + ATE Advance/DBT Recovery card
- f63fd50: Fix: hscToggle global scope, Journey Plan moved above connects, visit cache deferred re-render

---

## Completed Phases

### PHASE 1 — Data Integrity ✅
- FIX 1: nameMatch LOCAL function inside renderAccountHealth() fixed.
  Requires both strings >8 chars for substring match.
  Prevents LIC/SBI/IHCL cross-contamination.
- FIX 2: Attrition annualised — exits/HC × (12/3) × 100.
  Label = "Annualised (90-day)". attrition_rate_ann not used for display.
- FIX 3: Super Emp parser HC denominator confirmed correct.
  hc_snap[mon] first, fallbackHC = active_hc_current || 2830.
- FIX 4: normaliseCustomer() added before processReqTAT().
  Strips role suffix after hyphen. Fuzzy match vs CUSTOMER_LIST >0.82.
- FIX 5: Candidate parser req_id join.
  _reqMap from TA_ACTIVE_REQS. _unlinked flag. Console warning.

### PHASE 2 — Schema ✅
All migrations applied. See Supabase Tables section below.

### PHASE 3 — Overview Rework ✅
- HC source: active_hc_mar removed entirely from display
- Workforce Health Indicators strip: REMOVED
- HRBP Engagement scorecard: REMOVED from Overview
- Recruiter Leaderboard: REMOVED from Overview (stays on TA tab)
- Monthly Trend Table: ADDED (6 months, all roles)
- Region cards: absent count added, 3-col grid
- Second KPI row (cmdStrip2): 4 chips — Attrition MoM, Offer→Join %, Plat/Gold At Risk, Absent Resolution %
- Week selector: REMOVED from Overview, replaced with data freshness strip
- Absent resolution chip: reads _absResolvedCount / _absTotalCount (separate sbBoot queries, indexOf-safe)
- Offer→Join chip: reads _taSummaryCache (data_cache.ta_summary)
- MoM future-month guard: _monKeys capped at current month (idx ≤ getMonth())
- MTD Joiners card added
- Week resolution: fixed to latest available week

---

## Pending Phases

### PHASE 4 — Account Health Tab (NEXT after Ramesh upload)
- Load billing_type, contracted_hc, elah_hc at boot into CUSTOMER_LIST
  Add: bt, contracted_hc, elah_hc fields to each entry
- Composite health score (100pts):
  Deployment gap 35pts (T&M accounts only)
  Attrition 25pts (vs 15% fixed target)
  Absent burden 20pts
  HRBP visit recency 10pts
  Open critical reqs 10pts
- RAG: red <50 / amber 50-70 / green >70
- Billing type badge per card: T&M red, MS blue, AMC amber
- Deployment gap: (contracted_hc − (active_hc − absent)) / contracted_hc
- HC source priority: Elah HC (<7 days) → Super Emp → headcount field
- TA-scoped view: 3-state health only, no FnF/ER/HRBP details
- T&M priority flag in TA aging: red badge, T&M-first sort
- Plat/Gold aging panel: reqs >45 days at Platinum/Gold accounts
- POSITION LIFECYCLE PANEL (new, confirmed Phase 4 scope):
  Per account: resignation → backfill req raised → sourcing → offer → joined
  Data: resignation_tracker + account_positions_log + req_tracker + 
        prospective_joiners joined via backfill_req_id / zinghr_req_id
  Shows: days since vacancy, req status, candidates in pipeline,
         expected fill date, billing impact for T&M accounts

### PHASE 5 — HRBP Tab Redesign
- computeHRBPScore() redesigned to 5 dimensions (100pts):
  D1 Workforce Stability 25pts: attrition vs 15% fixed target + absent resolution
  D2 Account Coverage 25pts: tier-weighted visits (Plat 2pt, Gold 1pt) + quarterly TH/R&R
  D3 ER & Connect Quality 20pts: documentation + note heuristic (>100 chars + keywords = 1.5x)
  D4 TA Partnership 15pts: critical req engagement + backfill speed
  D5 Deployment Health 10pts: weighted gap rate Plat/Gold accounts
  Cap: score ≤70 if regional attrition >25%
  Geo-tagging: DROPPED until infrastructure ready
- Leaderboard: ALL roles, all peer names visible
- Visit scoring: calendar month cadence not rolling 4 weeks
- Monthly visit summary panel (admin/CXO/executive only)
- D4 TA Partnership panel (transparent scoring)
- Days-to-first-contact metric per HRBP (feeds D1 from absent_cases)

### PHASE 6 — Elah Parser + RMG Tab
- processElahDeployment(wb): reads "Billed Resource" sheet
  Builds: per-account HC, expiry register, ramp-down register
  Writes: data_cache (elah_deployment, elah_expiry_risk, elah_ramp_down)
  Upserts: elah_deployment table (UNIQUE emp_code + upload_week)
- File detection: resource_deployment / elah / deployment_detail
- RMG tab panels: Contract Expiry, Ramp-Down Pipeline
- RMG role: function_type='rmg' — not yet in user_profiles (Mohit to confirm names)
- Elah sunset target: July 1 2026

### PHASE 6B — ATE Management Module
- ATE Mix Dashboard (all roles):
  NATS limit = ROUND(region_fte_hc × 0.10) — DYNAMIC from workforce_intel
  Current: West 109/31, South 66/4, East 50/20, North 50/9, Total 275/64
- DBT Recovery Centre ✅ LIVE (25 May 2026):
  Visible to: hrops, payroll, admin, executive (role gate checks both role + function_type)
  Data from ate_advance_log aggregated per emp_code
  Net outstanding = total_advance − total_recovery
  Traffic lights: Cleared (green, 0 outstanding) | No Recovery (red, 0 recovered)
                  Amber (<50% of advance recovered) | Recovering (≥50% recovered)
  Filters: Region dropdown + Employment status (Active/Exited via _ateCases lookup)
  Dates: DD/MM/YYYY format via fmtDMY() helper
  Employment status badge: Sep / FTE badge on exited ATEs
  Global filter state: window._dbtRegionFilter, window._dbtEmpFilter
- Bench-to-Billable Pipeline (rmg + hrops)
- processATEAdvance(wb): ✅ LIVE — two-sheet upload
  Sheet 1 "Advance Paid": Emp Code, Employee Name, Region, Month,
           Advance Date, Amount, Voucher No, Notes
  Sheet 2 "Recovery": Emp Code, Employee Name, Region, Month Recovered,
           Recovery Date, Amount Recovered, DBT Intimation Ref, Payroll Ref, Notes
  File detection: ate_advance or ate_dbt in filename
  Header auto-detect: scans first 5 rows for "Emp Code" (handles blank row 1)
  Column aliases cover all confirmed naming variants (Voucher No, Month Recovered, Amount Recovered)
- Super Emp parser additions:
  Auto-classify deployment_status from training_end_date on upload
  Parse l_level from Designation field: L1/L2/DL1/DL2 keywords

### PHASE 7 — Workforce Planning Tab
- Bench Register: Field Services + Backup Pool, all emp types
- ATE bench separate from FTE bench
- Region capacity: L1+L2 demand vs ATE + FTE bench available
- RMG Action Log: CRUD on bench_deployments table

### PHASE 8 — Tab Redesigns (TA, Resignation, Prospective Joiners)

#### TA Tab (Phase 8A)
- Remove WoW chips as primary nav (always 0 from W13 onwards)
- Panel 1: Pipeline Health Strip (5 chips from req_tracker)
- Panel 2: Plat/Gold Critical Aging (reqs >45 days, T&M flagged) — TOP PRIORITY
- Panel 3: Req aging table with T&M badge + tier/region filter
- Panel 4: Recruiter Scorecard — filter to function_type='ta' ONLY
  (HRBPs were appearing due to role='hrbp' on TA users — FIXED in Supabase)
- Panel 5: Offer→Join Funnel (from weekly_reports + candidate upload)
- Unlinked candidates chip: "N candidates without req linkage"

#### Resignation Tab (Phase 8B)
- Past LWD Escalation panel: TOP section, 121 employees, T&M billing impact shown
- backfill_req_id inline editable field for RMG
- Position Lifecycle indicator per case
- Absconding → auto-link to resignation_tracker
- Attrition by account cluster (2+ resignations in 90 days)

#### Prospective Joiners Tab (Phase 8C)
- Auto-create from candidate upload when stage = "Offer Accepted"
  Manual entry only for edge cases
- Account-wise joining pipeline (links to open positions)
- At-Risk Joiners: DOJ >30 days from offer date (dropout risk flag)
- DNJ (Did Not Join) tracker
- Auto-close linked req when joiner's DOJ passes

Current data state (25 May 2026):
- Schema: 3 new cols added — emp_code, is_duplicate, matched_by
- 6 duplicates marked, 3 auto-resolved to Joined
- 84 Joined / 32 Offer Accepted / 3 Dropped Out
- Display filter: WHERE is_duplicate = false (shows 35 not 41)
- Process fix needed: TA to update ZingHR stage to "Joined" on joining day
  (Neha Kaur Sammi to enforce with team)

#### OD/WFH Tab (Phase 8D)
- OD Backlog strip by region
- Manager compliance league (most pending approvals)
- Account-wise OD pattern (T&M scope creep signal)
- OD Not Regularised → absent_cases connection
- WFH compliance when data flows from ZingHR

#### Absenteeism Tab (Phase 8E)
- Triage strip: 4 urgency buckets
  🔴 Uncontacted >7 days | 🟠 Contacted awaiting response
  🟡 Monitoring | 🟢 Closing (eSep/resigned)
- HRBP action queue: personal to-do sorted by days absent
- Account impact panel: Plat/Gold absent cases with billing impact
- Absconding pipeline with FnF status
- Pattern analysis: recurring accounts/regions month over month
- Days-to-first-contact metric (feeds HRBP D1 scorecard)

### PHASE 9 — Attrition History + Prediction
- attrition_history population from 2-year eSep data (Alex to upload)
- computeSeasonalIndex() from YYYY-MM keys
- computeBackfillForecast(): base rate × seasonal + (uncontacted absent × 0.7)
  + eSep initiated. 0.7 factor confirmed by Alex.
- Show as range not point estimate
- Prediction panel: TA tab + Account Health card

---

## Supabase Tables — Complete List

### HR CC Owns
- weekly_reports: weekly aggregates by week_key (W05-W23)
- weekly_submissions: individual form submissions per user
- user_profiles: 27 rows — see User Profiles section
- data_cache: key-value store
  Active keys: fnf, resignation, od, summary, account_hc,
  ta_summary, elah_deployment, elah_expiry_risk, elah_ramp_down,
  ate_dbt_summary
- absent_cases: 323 rows (215 Uncontacted, 25 Absconding, 27 Resolved)
- prospective_joiners: upcoming joiners per account
- ate_tracker: ATE cases with 14 new columns added 24 May
- ate_advance_log: Sheetal's monthly advance + recovery log (0 rows — pending first upload)
- hrbp_site_visits: visit details per submitter per account (18 cols after 05 Jun ALTER)
  Added cols: hrbp_name, hrbp_user_id, key_discussion, issues_flagged, issue_details, updated_at
  hrbp_name backfilled from submitter_name. RLS: hsv_read/insert/update_auth (authenticated)
- hrbp_visit_plans: Plat/Gold visit planning by HRBP (0 rows — new)
  Key cols: account_name, account_tier (Platinum/Gold), region, hrbp_name, planned_date,
  status (Planned/Completed/Cancelled/Rescheduled), site_visit_id FK→hrbp_site_visits
  RLS: hvp_read/insert/update_auth (authenticated)
- hrbp_er_cases: structured ER cases per HRBP per week (0 rows — new)
  Key cols: employee_name/code, case_type, status, opened_date, resolved_date,
  days_open + sla_breached (auto-computed by trigger _hec_compute_days, SLA=7 days)
  RLS: hec_read/insert/update_auth (authenticated)
- hrbp_retention_cases: at-risk employee tracking per HRBP (0 rows — new)
  Key cols: employee_name/code, account_name, risk_level (High/Medium/Watch),
  conversation_summary, action_taken, followup_date, followup_done, resolved
  RLS: hrc_read/insert/update_auth (authenticated)
- hrbp_discipline_cases: disciplinary actions per HRBP (0 rows — new)
  Key cols: employee_name/code, account_name, incident_type, incident_date,
  action_taken (Verbal Warning/Written Warning/SCN/Suspension/Termination/Counselling/Other),
  status (Open/In Progress/Closed/Appealed)
  RLS: hdc_read/insert/update_auth (authenticated)
- hrbp_photos: existing — added site_visit_id FK→hrbp_site_visits (05 Jun)
- workforce_intel: HC, attrition, wage bill (id='current')
- resignation_tracker: resignation cases
- req_tracker: open requisitions
- res_summary: resignation summary aggregates
- action_items: action centre tasks
- customer_accounts: 150 accounts, 4 new cols (billing_type, contracted_hc, elah_hc, elah_name)
- attrition_history: empty — pending Alex's 2-year eSep upload
- bench_deployments: empty — pending RMG setup
- elah_deployment: empty — pending Elah parser build (Phase 6)
- planned_leaves: empty — pending filename prefix from Ramesh
- account_positions_log: 23 cols, 6 indexes (0 rows — new table)
- training_sessions: trainer, training_name, month_key, week_key, region, mode, participants,
  category, count_type, duration_hours, is_mandatory, account, notes, created_by
  31 rows seeded (9 Maaz Khan, 22 Deepak Kumar Shetty) across Feb–May 2026
  Also written by: HRBP form (f_hrbp_training_entries → submitWeeklyForm)
  RLS: ts_read_all (SELECT authenticated), ts_insert_ld, ts_update_ld
- training_pipeline: training_name, planned_date, trainer, region, mode, category,
  expected_pax, duration_hours, is_mandatory, account, priority, dependency,
  status (Planned/Confirmed/Conducted/Postponed/Cancelled), postponed_to,
  cancel_reason, session_id (FK→training_sessions), created_week_key, updated_week_key
  Written by: LD form (f_ld_pipeline) + HRBP form (f_hrbp_pipeline) → submitWeeklyForm
  RLS: tp_read_all, tp_insert_auth, tp_update_auth (all authenticated)

### Shared With OPS360 — FLAG BEFORE TOUCHING
- workforce_intel, absent_cases, resignation_tracker
- prospective_joiners, planned_leaves, employee_account_mapping
- account_positions_log: candidate for OPS360 read access
  Add ops360_read policy when OPS360 service role is known

### Postgres Functions
- normalise_account_name(raw_name text) → text
  Uses pg_trgm similarity >0.4. Exact → elah_name → fuzzy → raw.
  Shared between HR CC and OPS360. Call instead of reimplementing.

---

## User Profiles — 27 Users (corrected 24 May 2026)

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

NOTE: TA users previously had role='hrbp' — FIXED 24 May 2026.
All TA users now correctly tagged role='ta', function_type='ta'.
Recruiter scorecard must filter by function_type='ta' (not role='ta').

---

## HRBP Region Assignments
- Abhishek Singh: North
- Surajit Sen: East
- Priya Paul: South
- Shambhavi Prathamesh Joshi: West
- Somasri Sukumar Samanta: West

HRBP_REGION lookup objects (two exist — keep in sync):
1. HRBP_REGION const (full name keys, line ~1654)
2. HRBP_REGION_MAP const (line ~4336, first-name based)
Both must have Somasri Sukumar Samanta → West.

---

## Workforce Intelligence — Current Numbers
- Active HC: 2,763 (Super Emp Master, 21 May 2026)
- active_hc_current = 2,763 | active_hc_mar = 2,979 (stale March)
- DO NOT use active_hc_mar for display anywhere
- Region HC: North 502 / South 660 / East 502 / West 1,088
- Total: 2,752 (from workforce_intel.regions)

### Monthly Trend Data (seeded 24 May 2026)
Joiners seeded in joiners_by_month: Jan=90, Feb=80, Mar=93, Apr=2
Monthly HC pending: Ramesh to confirm scope (FTE only or all types?)
Discrepancy: monthly_hc_snapshot shows Jan=2,979 → Apr=3,104 but
current HC is 2,763 — 341 gap unexplained. Ramesh to clarify.
monthly_hc set to NULL until confirmed — trend table uses 2,763 consistently.

### Attrition Rates (corrected denominators)
Jan: 34.6% | Feb: 20.5% | Mar: 19.9% | Apr: 1.2% | May: 19.1%
Apr shows only 3 exits in snapshot (partial capture) vs 32 in eSep.
Ramesh to upload complete May eSep to reconcile April.

---

## Upload Parsers

### ZingHR Super Employee Master (Ramesh — weekly)
- File detection: cmsitn prefix (precise — must NOT catch TA files)
- Populates: _acctHC, workforce_intel, ate_tracker (ATE rows)
- ATE auto-classification: training_end_date < today → Billable else Training
- L-level parsing: detect L1/L2/DL1/DL2 in Designation field
- After next upload: verify 31 ZingHR tagging fixes closed the gap

### ZingHR eSep Transaction (Ramesh — monthly)
- Populates: resignation_tracker, workforce_intel attrition slice
- HC denominator: hc_snap[mon] || active_hc_current || 2830
- Scoped to 2026 exits only

### ZingHR Req TAT (Ramesh — weekly)
- normaliseCustomer() applied to every customer field
- billing_type derived from customer_accounts.billing_type
- emp_category: FTE/ATE/Retainer (ZingHR attribute pending Alex)
- After next upload: all reqs will show correct tier (FIX 4 active)

### ZingHR Candidate Transactional (Ramesh — weekly)
- _reqMap from TA_ACTIVE_REQS before row loop
- customer from linked req when req_id matches
- _unlinked flag on unmatched candidates
- When stage = "Offer Accepted": auto-create prospective_joiner record

### Elah Resource Deployment (Ramesh — weekly, until July 1 sunset)
- File detection: resource_deployment / elah / deployment_detail
- Parser: processElahDeployment(wb) — NOT YET BUILT (Phase 6)
- Key Elah facts: 2,433 deployed / 88 accounts / 293 Retainer employees
  Available On = contract end date | Category "Ramp Down" = 10 employees
  31 employees in Elah with no ZingHR account tag (SAP risk before June 30)

### ATE Advance Upload (Sheetal — monthly from June 2026)
- File detection: ate_advance or ate_dbt in filename
- Parser: processATEAdvance(wb) — LIVE (built 25 May 2026)
- Confirmed column names from ate_advance_25052026.xlsx:
  Sheet "Advance Paid" (row 1 blank, row 2 headers):
    Emp Code | Employee Name | Region | Month | Advance Date | Amount | Voucher No | Notes
  Sheet "Recovery" (row 1 blank, row 2 headers):
    Emp Code | Employee Name | Region | Month Recovered | Recovery Date |
    Amount Recovered | DBT Intimation Ref | Payroll Ref | Notes
- Parser scans first 5 rows to auto-detect header (handles blank row 1)
- Column aliases handle all naming variants (Voucher No, Month Recovered, Amount Recovered)
- Upload card visible to: hrops + payroll function types
- hasFnF now also includes isHROps — Sheetal (hrops) can upload FnF files

### Planned Leaves (Ramesh — weekly)
- Parser: NOT YET BUILT | Filename prefix: TBC from Ramesh

---

## ATE Management — Confirmed Decisions

### NATS Formula
ROUND(region_fte_hc × 0.10) per region — dynamic from workforce_intel
Current headroom: West 78 / South 62 / North 41 / East 30 / Total 211

### DBT Flow
DBT goes directly govt → employee bank. CMS never receives DBT.
CMS exposure = salary advance paid − advance recovered.
Sheetal logs advance paid (Sheet 1) and recovery after DBT intimation (Sheet 2).

### ATE Deployment Status (auto-classified 24 May 2026)
West: 4 Billable / 27 Training
East: 2 Billable / 18 Training
North: 0 Billable / 9 Training
South: 0 Billable / 4 Training

---

## Account Positions Log — Workflow

Table: account_positions_log (live, 0 rows — RMG to populate)

RMG workflow (until ZingHR position code goes live June 30):
1. RMG sees vacancy (resignation confirmed or new position approved)
2. RMG raises req in ZingHR → gets req_id (e.g. REQ-2026-1847)
3. RMG opens HR CC Resignation tab (for backfills) or
   Account Health tab (for new positions)
4. RMG enters zinghr_req_id in the inline field
5. Dashboard joins: resignation ↔ req ↔ candidates ↔ offer ↔ joiner
6. Position Lifecycle panel on Account Health shows the full chain

Key columns:
- zinghr_req_id: links to req_tracker.req_id on next TAT upload
- linked_resignation_id: links to resignation_tracker (backfills)
- linked_absent_case_id: links to absent_cases (absconding)
- days_vacant: auto-computed (GENERATED ALWAYS AS STORED)
- status: Open / Sourcing / Offer / Filled / Cancelled / On Hold

---

## Billing Types — Key Reference
32 T&M accounts / 113 MS / 2 AMC in customer_accounts.
T&M accounts (deployment gap = direct billing loss per vacant day):
IHCL, Reliance Corporate IT Park, ONGC, Torrent Power,
Poonawalla Fincorp, Sutherland, Religare, Titan, Cadence,
Bajaj Finance, Syngene, Dixon, SMS India, Vitech, Clix Capital.
Deployment gap calc applies ONLY to T&M accounts.

---

## Manage Customer Accounts Tab — Status Decision Pending
Current: Admin UI to set region, tier, active per account.
Issue: Manual maintenance creates drift when ZingHR is source of truth.

Recommended (pending Alex's ZingHR discussion):
- Add Tier, Billing Type, Contract End Date, Contracted HC as
  ZingHR customer master attributes
- When live: tab becomes read-only display + computed columns only
- contracted_hc remains manually editable until ZingHR has that field
- Kesavan to fill contracted_hc for Plat/Gold accounts this week

---

## TA Summary — Seeded Data
data_cache key 'ta_summary' (seeded 24 May 2026):
  offer_join_pct: 85% (conservative — raw 101.4% normalised)
  total_offers: 141 | total_joined: 143 | total_dropouts: 12
  period: W05-W12 Feb-Mar 2026 (8 weeks with actual TA submissions)
  Updates when candidate data flows from ZingHR req TAT upload.

---

## OPS360 Relationship
OPS360 is a separate dashboard (not yet in production) sharing Supabase.
HR CC tables OPS360 will read (no write):
- customer_accounts, account_positions_log, ate_tracker
- bench_deployments, elah_deployment, attrition_history
- normalise_account_name() function

Shared (both read/write):
- workforce_intel, absent_cases, resignation_tracker, planned_leaves

OPS360 owns (HR CC may read in future):
- normalized_tickets, sla_contracts, delivery_hierarchy

When OPS360 is ready: add ops360_read RLS policy to HR CC tables.
No schema changes needed — just policy additions.

Key principle: OPS360 reads from HR CC data, never rebuilds parallel stores.
billing_type, tier, elah_name in customer_accounts serve both dashboards.

---

## Absent Cases — Status Values (exact as stored)
"Uncontacted" | "OD Not Regularised" | "Absconding — eSep Initiated"
"Resolved — Resigned" | "Resolved — Returned" | "Missed Punch"
"On Leave (Unplanned)" | "Resignation — HR Chain Pending"
"Under Investigation" | "Warning Letter Issued" | "Medical / Emergency"

Resolution filter must use indexOf('resolved') not includes() —
em-dash (—) in status values may cause Unicode mismatch with includes().

---

## Weekly Reports — Data Status
W05-W12 (Feb-Mar 2026): real submissions, TA data populated
W13-W17 (Apr-May): declining, TA opened/offers = 0 (team switched to ZingHR upload)
W18, W20, W23: zero respondents — ghost weeks
Decision confirmed: Remove week selector from Overview.
Replace with data freshness strip (Claude Code prompt written, pending commit).

---

## Pending People Actions
| Person | Action | When |
|---|---|---|
| Alex | Add Tier + Billing Type attributes in ZingHR customer master | Discuss with ZingHR team |
| Alex | Add Billing Type dropdown in ZingHR recruitment module | Before June 30 |
| Alex | Add Employee Category (FTE/ATE/Retainer) in ZingHR recruitment | Before June 30 |
| Alex | Upload 2-year eSep data for attrition_history | Phase 9 |
| Kesavan | Fill contracted_hc for Plat/Gold via Admin UI | This week |
| Kesavan | Decision on Manage Customer Accounts tab scope | Pending discussion |
| Ramesh | Upload corrected Super Emp Master (31 ZingHR tagging fixes done) | Tomorrow |
| Ramesh | Clarify HC scope: FTE only or all types in Super Emp extract | Tomorrow |
| Ramesh | Upload complete May eSep to reconcile April exits | Tomorrow |
| Ramesh | Confirm Retainer employees (293 in Elah) in Super Emp extract | This week |
| Ramesh | Confirm planned_leaves filename prefix | Pending |
| Mohit | Confirm RMG team member names for user_profiles | Tomorrow |
| Sheetal | Use ATE_Advance_Upload_Template.xlsx from June 2026 | June 2026 |
| RMG team | Enter zinghr_req_id in HR CC for each backfill/new position | After Phase 6 builds |
| RMG team | Tag emp_category=ATE on L1/L2 reqs in ZingHR | After ZingHR attribute added |

---

## ZingHR Attributes to Add
Recruitment module (Alex to configure):
1. Billing Type — dropdown: T&M / Managed Services / AMC / Hybrid (mandatory)
2. Employee Category — dropdown: FTE / ATE / Retainer (mandatory)

Customer master module (discuss with ZingHR team):
1. Account Tier — dropdown: Platinum / Gold / Silver / Bronze / Regular
2. Billing Type — dropdown: T&M / Managed Services / AMC / Hybrid
3. Contract End Date — date field
4. Contracted HC — numeric field

---

## Key Team Members
- Alex Augustine: admin, project owner
- Kesavan: Head of HR — admin user in dashboard
- Shambhavi Prathamesh Joshi: HRBP West
- Somasri Sukumar Samanta: HRBP West
- Surajit Sen: HRBP East
- Priya Paul: HRBP South
- Abhishek Singh: HRBP North
- Mohit Kumar: HR Ops / Payroll (hrops role)
- Sheetal Sadashiv Pachangane: Payroll — ATE advance uploads
- R Ramesh: Payroll — all ZingHR weekly uploads + Elah report
- Deepak Kumar Shetty: L&D
- Ashok B C: Facilities
- Neha Kaur Sammi: TA Lead (Pan India)
- Ajith Inguva: TA (Pan India)
- Mousumi Priyadarsini Behera: TA South
- Caral Anitha Dsouza: TA South
- Pijush Dutta: TA East
- Salma Saifi: TA North
- Dhananjay Kumar Singh, Juee Nilesh Patil, Kumari Puja,
  Nikita Yadav, Nilam Sunil Patil: TA West
- Pruthvi SS: RMG South
- Chander Mohan: RMG (region TBC)
- Amit Dasgupta: Delivery leadership (ATE initiative owner)

---

## Action Centre
- Tasks from action_items table
- due_date renders inline, overdue = red
- owner field: confirm renders inline (known gap — verify on live)
- Status: Open / In Progress / Closed / Done
- Admin/hrops: can add and edit tasks

---

## Known Debt — Carry Forward
- owner field in Action Centre task cards: confirm renders inline
- WI hardcoded fallback line ~845: stale values. Self-corrects on uploads.
- monthly_hc_snapshot gaps: Ramesh to upload eSep Jan-May 2026
- CEAT double-space duplicate in ZingHR: "CEAT LIMITED" + "CEAT  LIMITED"
- LIC naming: "LIC (EAST)" vs "LIC ( EAST )" — standardise in ZingHR
- dbt_months_received + advance_recovered on ate_tracker: redundant
  (ate_advance_log is source of truth). Keep schema, don't display.
- planned_leaves parser: filename prefix TBC from Ramesh
- Somasri Sukumar Samanta: confirm in both HRBP_REGION lookup objects
- Manage Customer Accounts tab: decision pending on ZingHR attributes
- Duplicate switchTab: line ~3564 is dead code, line ~4213 is live. Clean up in future session.
- _monKeys future-month guard in code (idx ≤ getMonth()). WI fallback line ~845 may still
  contain future-month entries — self-corrects on next Ramesh upload.

---

## When to Update This File
UPDATE AFTER (not during):
- Schema change in Supabase → table list + columns
- Phase completes in Claude Code → line count + commit hash
- Design decision confirmed → relevant section
- New upload parser added → file detection + what it writes
- Role access changes → user roles section
- Bug fixed → remove from Known Debt
- New pending item → Pending People Actions

DO NOT update for: work in progress, items being debated,
analysis outputs (Excel reports, recon files).

---

## Build Sequence — What's Next

NEXT SESSION (Phase 4 + 5):
  Phase 4: Account Health tab — billing type badge, composite score (100pts),
           Position Lifecycle panel, T&M priority flag in TA aging
  Phase 5: HRBP scorecard redesign — 5 dimensions, 100pts, calendar month cadence

AFTER RAMESH UPLOADS:
  Verify 31 ZingHR tagging fixes close the account HC gap
  Confirm HC scope for monthly trend (FTE only vs all employee types)
  Upload complete May eSep to reconcile April exits

FOLLOWING SESSIONS:
  Phase 6:   Elah parser (processElahDeployment) + RMG tab + RMG role setup
             Target: before Elah sunset July 1 2026
  Phase 6B:  Bench-to-Billable Pipeline (rmg + hrops) — only remaining Phase 6B item
  Phase 7:   Workforce Planning tab
  Phase 8:   TA / Resignation / Prospective Joiners / OD / Absenteeism redesigns
  Phase 9:   Attrition history + prediction (Alex to upload 2-year eSep data first)
