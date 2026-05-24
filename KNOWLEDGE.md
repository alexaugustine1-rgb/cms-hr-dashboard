# CMS HR Ops Command Centre — Project Knowledge
**Version:** 2.0 | **Last updated:** 24 May 2026 — end of session

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

## Known Crash Patterns — Check Every Edit
Any of these crashes login with "ReferenceError: sbLogin is not defined":
- Orphaned em-dash in string concatenation
- await inside a non-async function
- Block-scoped function declarations inside try blocks
- const or let declared inside try blocks
When node --check reports an error: print 3 lines above and 3 below
before attempting any fix.

---

## Current File State — 24 May 2026
- Line count: 10,964 lines (latest confirmed after commit 1ac9369)
- Latest commit: 1ac9369
- Local path: D:\Dropbox\CMS_IT_Services\Claude_HR_Automation\
  HR_OPS_COMMAND\cms-hr-dashboard\index.html

### Commits This Session
- 13222f4: Overview rework (WH strip removed, trend table, region cards)
- c739d46: Second KPI row (4 chips added)
- 1ac9369: Trend table HC/joiners fix + week resolution to latest week

### Pending Claude Code Fixes (not yet committed)
These prompts are written and ready — paste into Claude Code:
1. Absent resolution chip: indexOf filter (Unicode safe for em-dash)
2. Offer→Join chip: reads from _taSummaryCache (data_cache.ta_summary)
3. Week selector removal from Overview + data freshness strip
After these three: single commit with all three fixes.

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

### PHASE 3 — Overview Rework ✅ (partial — 3 fixes pending)
Completed:
- HC source: active_hc_mar removed entirely from display
- Workforce Health Indicators strip: REMOVED
- HRBP Engagement scorecard: REMOVED from Overview
- Recruiter Leaderboard: REMOVED from Overview (stays on TA tab)
- Monthly Trend Table: ADDED (6 months, all roles)
- Region cards: absent count added, 3-col grid
- Second KPI row (cmdStrip2): 4 chips added
  Chip 1: Attrition MoM (▲/▼ pp vs prior month)
  Chip 2: Offer→Join Rate % (pending fix — shows 0% until fixed)
  Chip 3: Plat/Gold At Risk (absent burden >5%)
  Chip 4: Absent Resolution % (pending fix — shows 0% until fixed)
- Week resolution: fixed to latest available week (sbBoot line 9115)

Pending (prompts written, not yet committed):
- Absent resolution chip filter: indexOf('resolved') Unicode safe
- Offer→Join chip: read from _taSummaryCache not TA_SUMMARY
- Week selector: remove from Overview, add data freshness strip

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
- DBT Recovery Centre (hrops/payroll only):
  Data from ate_advance_log aggregated per emp_code
  Net outstanding = total advanced − total recovered
  Traffic light: green=0, amber=1-2 months, red=3+ months
- Bench-to-Billable Pipeline (rmg + hrops)
- processATEAdvance(wb): two-sheet upload
  Sheet 1 "Advance Paid": Emp Code, Name, Region, Month, Date, Amount, Voucher
  Sheet 2 "Recovery": Emp Code, Name, Region, Month Recovered For,
           Recovery Date, Amount, DBT Intimation Ref, Payroll Ref, Method
  File detection: ate_advance or ate_dbt in filename
  Template: ATE_Advance_Upload_Template.xlsx (built, shared with Sheetal)
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
- hrbp_site_visits: visit details per submitter per account
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
- Parser: processATEAdvance(wb) — NOT YET BUILT (Phase 6B)
- Template: ATE_Advance_Upload_Template.xlsx (ready, shared with Sheetal)

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
```
IMMEDIATE (today — pending Claude Code commit):
  Bugfix 1: Absent resolution chip filter (indexOf, Unicode safe)
  Bugfix 2: Offer→Join chip (_taSummaryCache from data_cache)
  Bugfix 3: Week selector removed from Overview + freshness strip
  Single commit after all three pass node --check

TOMORROW (after Ramesh uploads):
  Verify 31 tagging fix closes account HC gap
  Verify ATE deployment status auto-classified correctly
  Confirm HC scope (FTE only vs all types)
  Phase 4: Account Health — billing badge + score + lifecycle panel
  Phase 5: HRBP scorecard redesign

NEXT SESSIONS:
  Phase 6: Elah parser + RMG tab + RMG role setup
  Phase 6B: ATE tab — mix dashboard + DBT recovery + Sheetal parser
  Phase 7: Workforce Planning
  Phase 8: TA / Resignation / Prospective Joiners / OD / Absenteeism redesigns
  Phase 9: Attrition history + prediction

## File Details — update to:
Line count: 11,005 lines
Latest commit: 90d1d1b

## Commits This Session — add:
- 90d1d1b: Absent resolution fix (indexOf), Offer-Join rate 
  from _taSummaryCache, week selector hidden on Overview via 
  switchTab display:none, data freshness strip added

## Phase 3 — change to: ✅ COMPLETE
All fixes deployed. No pending Claude Code items for Phase 3.
Dead code note: duplicate switchTab at line 3564 — 
line 4213 is the live one. Flag for future cleanup.

## Prospective Joiners — add:
Schema: 3 new cols (emp_code, is_duplicate, matched_by)
Data: 6 duplicates marked, 3 auto-resolved to Joined
Current: 84 Joined / 32 Offer Accepted / 3 Dropped Out
Display: WHERE is_duplicate = false (35 shown, not 41)
Process fix needed: TA to update ZingHR stage to "Joined" 
on joining day — Neha Kaur Sammi to enforce

## Additional commits — 24 May 2026 (evening)

7407319: MoM future-month guard (_nowMonIdx filter),
         MTD Joiners card (reads joiners_by_month),
         Absent deferred re-render timing fix
7520369: Absent resolution chip — separate count query
         (_absResolvedCount + _absTotalCount in sbBoot).
         loadAbsentCases() query UNCHANGED.
Final line count: 11,016

## Known debt — add:
- Hardcoded WI fallback (line ~845) contained Jun/Jul
  future-month entries — removed from Supabase today.
  _monKeys now has idx <= getMonth() guard in code.
  Permanent fix regardless of WI content.
- Duplicate switchTab function: line 3564 is dead code,
  line 4213 is live. Clean up in future session.

## Absent resolution chip — architecture note:
Uses window._absResolvedCount / window._absTotalCount
set by two separate boot queries (ILIKE resolved% + count head).
NOT derived from _absentCases (which excludes resolved by design).
Graceful fallback: _absentCases.length + _resolvedAbs if counts not ready.
```
