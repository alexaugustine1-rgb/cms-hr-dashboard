# CMS HR Ops Command Centre — Project Knowledge
**Version:** 3.1.0 | **Last updated:** 18 Jun 2026 — two-status model locked, phantom-open principle, RMG users + sidebar scoping, session log added

> **This is the single source of truth for the project.** It replaces the older
> `HRCC_Project_knowledge.MD` and `cms_hr_cc_knowledge_v2.md` files. Update this
> file (and only this file) at the end of every session. See the update protocol
> at the very bottom.

---

## Recent Updates (Session Log)
> Newest first. Add a dated entry here at the end of every session.

### 18 Jun 2026 — RMG status model, TAT enrich, filter-responsive cards
- **req_tracker is now a FULL MIRROR of the TAT (~774 rows).** processReqTAT had been
  truncating to ~417 rows — the root cause of the 189-vs-217-vs-182 open-count
  confusion. Now retains all 774 rows, adds raw `zinghr_status` column, and derives
  `status` = Open only when zinghr_status='Approved', else Closed. Enrich Step 3
  closes anything with balance ≤ 0. Verified: 774 rows ≈ 189 Open / ≈ 215 stale.
  (312fb7a; full-mirror groundwork 1368754)
- **TAT enrich is live.** Supabase RPC `sync_positions_from_req_tracker` pushes
  req_status + age from req_tracker onto RMG positions, auto-fires after every TAT
  upload, and writes ONLY req_status / req_age_days / audit columns — manual fields
  cannot be touched. Status vocab aligned to Open/Closed across the Add form and
  filter pills. Verified by hand (req_status=rt.status, req_age=rt.tat exact). (fccee3d)
- **Filter-responsive RMG cards (fixed).** Root cause was a wrong-source bug: the five
  cards read the full array while the grid read the filtered copy. Cards now read a
  Region+Status subset (`cardBase`); flag pills filter grid rows only and the cards
  hold their RED/AMBER/GREEN breakdown. (e970c51)
- **RMG users + sidebar scoping (this Cowork session).** 4 RMG users (Sonal Rale,
  Pruthvi SS, Nehal Shaikh, Chander Mohan) via seed SQL; sidebar scoped to 7 tabs;
  Long Absenteeism gated read-only for RMG. node --check PASS. Pending commit + push
  and SQL run (after Alex creates the 4 Supabase Auth accounts).
- **Decisions locked:** RMG Workspace holds ALL reqs incl non-delivery/corporate;
  only a few confidential searches excluded (confidential flag still to build.)
  Two-status model locked (see Key Learnings + Current State).
- **Phantom-open principle:** system ~190 open vs Neha's real ≤100 ⇒ ~90 phantom-open
  ON TOP of the 217 stale. Surface with signal flags, let cleanup converge the count;
  do not redefine "open" down. (See Key Learnings.)
- **Still open:** visual confirm on cards → fork between Close action vs Candidate
  enrich → eSep surfacing → open-position-health view. Confirm what Neha's 100 counts
  (all active open vs TA-sourced active).
- **Recurring:** Vercel MCP 403 on team scope — needs connector re-auth (2nd session).

---

## What This Is
Single-file HTML HR Operations dashboard for CMS IT Services.
Used daily by HR team, HRBPs, Talent Acquisition, RMG, and executive leadership.

Covers: Overview (CXO command strip), TA Pipeline, TA Commitments, HRBP Connects
and scorecards, HR Ops / Payroll, L&D, Risks, Workforce Intelligence, Resignation
& Backfill, OD/WFH compliance, FnF tracker, Long Absenteeism, ATE & Contract,
Prospective Joiners, RMG Workspace, Action Centre, Admin/Settings.

---

## Live Deployment
- URL: cms-hr-dashboard.vercel.app
- Repo: alexaugustine1-rgb/cms-hr-dashboard
- Branch: main | File: index.html (always at repo root, exact name)
- Vercel auto-deploys on every push to main
- Git author: alexaugustine1@gmail.com
- PAT: [redacted — store in password manager / .git config, never in repo]
- vercel.json: no-cache headers on / and /index.html — keep committed at repo root

---

## Supabase
- Project ID: mzyrcrkwgbqgwajkjdnp
- Shared with OPS360 (different schemas, same project)
- execute_sql for DML/queries | apply_migration for DDL
- RLS with zero policies = 400 errors on every authenticated query. Always add at
  least one policy when creating a table.
- pg_trgm extension: ENABLED (for normalise_account_name function)

---

## Standing Rules — NON-NEGOTIABLE (operational)
These govern how every build session edits index.html.

- **str_replace ONLY.** Never rewrite the full file. If a task feels like it needs
  a full rewrite, STOP and tell Alex instead of proceeding.
- **node --check after EVERY edit.** Extract scripts to a temp .js first (Node v24
  won't syntax-check .html directly). No output = clean pass.
- If node --check errors: print 3 lines above and 3 below before attempting a fix.
  Quote-escaping in string concatenation is the most common cause.
- Single HTML file. No splitting. No build toolchain. No npm, webpack, React.
- All persistent data in Supabase. No hardcoded arrays with real data.
- Git commits authored as alexaugustine1@gmail.com.
- After every phase: explicit PASS/FAIL checklist before stopping.
- Node.js path (Windows): C:\Program Files\nodejs\node.exe (confirmed installed).

### Overwrite Prevention — MANDATORY
Features get silently lost when sessions rewrite the full file.
1. str_replace only. Never write the full file.
2. Note index.html line count before each session. If the output differs by more
   than ~50 lines from input, STOP — do not commit (likely a silent full rewrite).
   The 50-line threshold is a trip wire, not a hard cap; legitimate new-function
   additions over 50 lines must be documented in the commit report. Distinction:
   lines added = new feature code (OK); lines removed + replaced ≈ input count =
   silent rewrite (NOT OK).
3. Features known to have been lost in past rewrites — must exist at every commit:
   - HRBP action dropdown on OD/WFH tab (wfh_od_actions)
   - _wfhOdActions cache + saveODAction() function
   - loadWFHODActions() function
   - multi-instance badge in renderOD rows
4. Before committing, grep for these — if any return 0 results, DO NOT commit:
   `grep -n "saveODAction\|loadWFHODActions\|_wfhOdActions\|multiBadge" index.html`

### Known Crash Patterns — Check Every Edit
Any of these crashes the login flow with "ReferenceError: sbLogin is not defined":
- Orphaned em-dash in string concatenation
- await inside a non-async function
- Block-scoped function declarations inside try blocks
- const or let declared inside try blocks
- Any JS syntax violation before sbLogin is defined
node --check catches all of these reliably.

### Cowork Note — Dropbox sync vs bash sandbox
This repo lives in a Dropbox-synced folder. The Cowork Linux sandbox (bash) can see
a STALE / partial copy of index.html (wrong byte size, truncated tail) even after
edits. The Read/Edit file tools always see the true current file. To node --check
in Cowork, extract the edited regions via the Read tool into a temp .js in the
outputs directory (sandbox-writable) and check that, rather than trusting the bash
view of index.html.

### Workflow: Cowork + Claude.ai
- Planning / diagnosis / prompt writing → Claude.ai chat
- Build sessions → Cowork + Claude Code
- Review / KNOWLEDGE.md → Claude.ai chat
- "Cowork builds, Claude.ai steers"

---

## Architecture — Non-Negotiable (technical)
- Single-file HTML. Never split.
- No build toolchain. No npm/webpack/React.
- All persistent data in Supabase. No hardcoded arrays with real data.

### Single-File Architecture Assessment (17 Jun 2026)
Sound for the next 6 months. Risks are in specific large functions, not the
architecture. Vite/TypeScript migration deferred — revisit only if the file
becomes unmaintainable.

Function size watch list (split into orchestrator + sub-functions when next touched
if over 300 lines):
- renderHRBP: 523 lines — highest risk for accidental edits
- renderATETab: 370 lines
- renderTA_Commitments: 357 lines
- renderWorkforce: 363 lines

### Key Function Locations (search anchors)
- sbBoot(): "async function sbBoot"
- showSidebarAfterLogin(): "function showSidebarAfterLogin" (role/tab gating)
- renderOverview(): "function renderOverview"
- renderAccountHealth(): "function renderAccountHealth" (nameMatch() LOCAL inside, ~line 6291)
- renderAbsenteeism(): "function renderAbsenteeism"
- renderResignation(): "function renderResignation"
- renderTAPipeline(): "function renderTAPipeline"
- renderJoinersTab(): "function renderJoinersTab"
- renderRMGShell() / loadRMGTab(): RMG Workspace
- normaliseCustomer(): "function normaliseCustomer"
- processReqTAT(): "async function processReqTAT"
- processCandidate(): "async function processCandidate"
- handleUploadFile(): "function handleUploadFile"
- computeHRBPScore(): "function computeHRBPScore"
- loadLDTab() / _loadLDSessions() / _loadLDPipeline(): L&D tab data loaders
- switchTab(): TWO definitions — line ~3845 is DEAD; line ~4608/4668 is LIVE. New tab wiring goes in the live one.
- Globals: CUSTOMER_LIST, TA_ACTIVE_REQS, _absentCases, WI (hardcoded ~line 845, overridden by Supabase at boot)

---

## Architecture Notes

### Absent Resolution Chip
Reads `window._absResolvedCount` and `window._absTotalCount` — set by two separate
queries in sbBoot() (ILIKE 'resolved%' count + total head count). NOT derived from
`_absentCases` (that array excludes resolved cases by design). Do NOT modify
loadAbsentCases() query — it intentionally excludes resolved.

### innerHTML Does NOT Execute `<script>` Tags
Any function callable from onclick handlers in dynamically rendered HTML MUST be a
true global (top-level script scope), not inside an innerHTML string. Confirmed
crash: hscToggle was inside innerHTML `<script>` — moved to global (25 May 2026).

### Role Gate Pattern — Use Both Fields
`_sbRole.role` and `_sbRole.function_type` are DIFFERENT fields. Gate checks must
read both:
```javascript
var role   = _sbRole ? (_sbRole.role   || '') : '';
var fnType = _sbRole ? (_sbRole.function_type || '') : '';
```
Comparing only function_type blocks admin users (function_type='admin', not 'hrops').
NEVER gate on function_type alone. Always: `role==='admin' OR function_type==='X'`.

### Tab Visibility / Sidebar Scoping (showSidebarAfterLogin)
- Called as `showSidebarAfterLogin(_sbRole.role)`.
- Most sidebar items are visible by default; per-role blocks then hide what doesn't apply.
- HRBP function-type hiding runs only inside `if(role === 'hrbp')`.
- RMG scoping (added 18 Jun 2026): `if(role!=='admin' && function_type==='rmg')`
  whitelists only snav-overview, snav-rmg, snav-tapipeline,
  snav-prospective-joiners, snav-resignation, snav-acct-health, snav-absenteeism;
  hides all other sidebar items, the submit buttons, and empty groups.
- tab_scope field only used for the facilities-only restriction
  ('admin_facilities_only' / 'admin_facilities_overview_actions'). RMG leaves it null.

### L&D Tab Architecture (redesigned 05 Jun 2026)
`renderLD(w, pw)` returns a static shell only. `loadLDTab(monthKey)` drives all data
loading. Called by render() (100ms), switchTab('ld'), ldMonthSel onChange.
- `_loadLDSessions(mk)`: training_sessions → KPI chips + table
- `_loadLDPipeline(mk)`: training_pipeline (Planned/Confirmed) → pipeline table
- Month selector id: `ldMonthSel`. Constants: LD_TRAINERS (7) + LD_CATEGORIES (6).
- HRBP form has TWO training sections: "Training Conducted This Week" →
  f_hrbp_training_entries → training_sessions; "Training Programs Planned/Conducted"
  → f_hrbp_pipeline → training_pipeline.

### HRBP Site Visit Cache
`_hrbpSVCache` holds ALL site visits — not week-restricted. Journey Plan panel and
scorecard both read it. loadHRBPSiteVisitsForTab() triggers deferred re-render (50ms)
after populating the cache. No region field — last-visit date is pan-account.

### Postgres: No CURRENT_DATE in Stored Generated Columns
IMMUTABLE requirement — Postgres rejects CURRENT_DATE / now() in stored generated
columns. hrbp_er_cases.days_open + sla_breached computed by trigger
`_hec_compute_days`. account_positions_log.days_vacant is GENERATED STORED
(expression does not reference today).

---

## Key Learnings

### ATE Conversion Emp Code Problem
ATEs have 22xxxxxxx emp codes. On FTE conversion ZingHR issues a new 11xxxxxxx code.
Solution: HRBP enters new FTE emp code in `fte_emp_code` at Conversion Initiated
stage. processEmployeeMaster reconciliation auto-confirms when `fte_emp_code`
appears in active non-ATE rows.

### getEmpType() / ATE Detection
ATE identified by Grade (col CM) = 'ATE' via `isATE(r)` — NOT EmpType (col CK).
Corrects the count from 27 (EmpType) to 65 (Grade). FTC/Consultant/CONTRACT remain
EmpType-based.

### Account Health — Attrition Formula
Rate is annualised: `exits / activeHC × (365/90) × 100`. Thresholds 3% / 8% / 15%
are annual rates. Label shows '(90d) · ann.'. Exit count uses live LWD computation
from `r.lwd` date string — NOT the frozen `days_to_lwd` integer.

### Account Health — Region Filter Pattern
- `_absentCases`: region ✅ | `FNF_DATA`: region ✅ | `CMS_RES_DATA`: region ✅
- `_hrbpSVCache` (last visit): NO region field — pan-account only ⚠
Filter pattern: `if (acct.r && r.region && r.region !== acct.r) return false;`

### Frozen days_to_lwd — Root Cause
`days_to_lwd` is baked into CMS_RES_DATA at upload time. Always recompute from
`r.lwd` date string at render/compute time.

### HC Scope — KEY PRINCIPLES (confirmed 17 Jun 2026)
- Consultant and FTC are employment-type labels only.
- For all HC reporting, workforce planning, account deployment, and backfill —
  treated identically to permanent FTE.
- Contractual workforce (HC breakdown) = Consultant + FTC + Contract + ATE.
- ATE shown separately only in ATE Tracker module.
- FTC/Consultant ARE included in total HC (account_hc cache and WI regional totals).
- Total HC 2,621 includes all emp types.
- HC Budget from RMG covers total HC (FTE + FTC + Consultant). ATE grouped with
  contractual workforce for HC breakdown.

### Elah-as-Budget-Ceiling Principle (RMG rule)
Every billable req must trace to an Elah demand line. Net-new positions not in Elah
are exceptions. Only non-billable roles may sit outside Elah. A position is open
only if: Status != Closed AND Balance > 0 AND Billable = Yes.

### ZingHR Recruitment Data Issues (17 Jun 2026 analysis)
- TAT Customer Name blank on 758 of 774 rows — account name trapped in requisition title
- 217 reqs filled-but-never-closed in ZingHR
- Raw open count: 405 → Reconciled open count: 182 (Elah-as-budget-ceiling applied)
- 66 critical resignations with no backfill raised
- 73 seats above Elah demand line need verification (Indian Hotels, SBI General, Avenue Supermarts)

### RMG Reconciliation Flag Logic
- RED — elah_demand_id is null/empty OR in_elah = false
- AMBER — position_type contains 'Backfill' AND backfill_emp_code is null/empty
- GREEN — elah_demand_id set AND (not a backfill OR backfill_emp_code set)

### Open-Count Truth / Phantom-Open Principle (18 Jun 2026)
System shows ~190 open vs Neha's real ≤100 ⇒ ~90 phantom-open ON TOP of the 217
already-isolated stale reqs. Balance alone is an unreliable real-open signal (it
caught the 217 but misses the ~90). Surface the gap with phantom-signal flags —
aged-no-movement / no recruiter / no candidate / balance-vs-joined — and let cleanup
plus the Close action converge the number. Do NOT redefine "open" down to hit the
target; that hides the very problem the reconciliation exists to expose. Candidate
pipeline activity (not balance) is the real-vs-phantom discriminator for the next build.

---

## Current File State — 18 Jun 2026
- Line count: ~13,152 (index.html). True size only visible via file tools in Cowork (see Dropbox sync note).
- Latest committed HEAD: e970c51 — RMG filter-responsive cards
- Pending (NOT yet committed): RMG sidebar scoping in showSidebarAfterLogin + Long Absenteeism read-only gate for RMG. node --check PASS.

### Recent Commits (newest first)
- e970c51 (18 Jun): RMG filter-responsive cards — counts follow Region+Status, flag filters rows only
- fccee3d (18 Jun): RMG: TAT enrich live + Open/Closed pills + filter-responsive cards
- 312fb7a (18 Jun): fix derived status Open-only for Approved
- 1368754 (18 Jun): req_tracker: mirror full TAT — retain all reqs, add zinghr_status (raw), derive Open/Closed from status + balance
- daa1ab0: req_tracker as live source — TAT upserts with enrich RPC, candidate req_id linkage
- fb43e77 / fa1976d: Recruiter Mapping — TA recruiters only, strip emp codes from attribution
- 40fd550 (17 Jun): TA Pipeline — billing type filter, filled-unclosed banner, backfill emp column
- aa507d7 (17 Jun): RMG Workspace — reconciliation grid, Add Position form, role-gated view/edit
- 8f9edc6 (09 Jun): Account Health — region filter, live LWD, annualised attrition
- c355d6d (09 Jun): ATE conversion tracking — fte_emp_code, reconciliation, UI badges
- e48a8f0: Action Centre — clickable KPI chips, filter bar, table view
- 0d9450d: OD/WFH — wfh_od_actions, multi-instance badge, HRBP action dropdown

---

## Completed Phases / Work

- **PHASE 1 — Data Integrity ✅** nameMatch LOCAL (renderAccountHealth), attrition annualised, HC denominator, normaliseCustomer().
- **PHASE 2 — Schema ✅** all migrations applied (see Tables).
- **PHASE 3 — Overview Rework ✅** Monthly trend table, region cards, data freshness strip, MTD Joiners, Absent Resolution chip, Offer→Join chip.
- **HRBP Tab ✅ (08 Jun)** Schema: hrbp_visit_plans, hrbp_er_cases, hrbp_retention_cases, hrbp_discipline_cases. 4 sub-tabs.
- **OD/WFH Enhancements ✅ | ATE Advance Upload ✅ | Post-Upload Refresh ✅ | Action Centre Filter Bar ✅**
- **Resignation Tab Fixes ✅ (09 Jun)** days_to_lwd recomputed at render; location column removed; exit reason shown; no-notice badge; Plat/Gold chip fix.
- **ATE Detection Fix ✅ | ATE Conversion Tracking ✅ (09 Jun)**
- **Account Health Signal & Attrition Fixes ✅ (09 Jun)**
- **RMG Workspace Tab ✅ (aa507d7 — 17 Jun)** Tab id: rmg, icon 🔗. Access: admin/rmg = full edit; hrbp/ta/hrops = read-only; others hidden. KPI strip (Total/No Elah ID/RED/AMBER/GREEN), filter bar (Region/Status/Flag), 9-col grid (Req ID, Elah ID editable, Account, Region, Position Type dropdown, Backfill Emp editable, Status, Age, Flag), Add Position form (8 fields), saveRMGField optimistic update. Live switchTab wired at line ~4668. 10 functions: renderRMGShell, loadRMGTab, _renderRMGGrid, _renderRMGAddForm, toggleRMGAddPanel, rmgSyncRegion, submitRMGNewPosition, _rmgKpi, _rmgReconFlag, saveRMGField. showToast global at ~line 8795.
- **TA Pipeline Refresh ✅ (40fd550 — 17 Jun)** _taBilling filter (All/T&M/MS); filledUnclosed banner; req table 9 cols (added Backfill = separation_emp_code).
- **TA Recruiter Mapping + req_tracker live source ✅ (daa1ab0)** Neha sets primary owner per req; TAT upserts via enrich RPC; candidate req_id linkage; attribution cleanup.
- **req_tracker Full Mirror + ZingHR Status ✅ (18 Jun — 1368754 + 312fb7a)**
- **TAT Enrichment + account_positions_log Sync ✅ (18 Jun — fccee3d)**
- **RMG Filter-Responsive Cards ✅ (18 Jun — e970c51)**
- **RMG Users + Sidebar Scoping ⏳ (18 Jun — working, pending commit + push)**

### req_tracker Full Mirror (18 Jun 2026)
- `zinghr_status text` column added (migration: req_tracker_add_zinghr_status_20260618).
- processReqTAT now upserts ALL ~774 TAT rows via `allRows` array (not just Approved + current-month-Closed).
- `zinghr_status` = raw ZingHR status verbatim. `status` (derived) = 'Open' if zinghr_status==='Approved', else 'Closed'.
- `active` / `closed` arrays still maintained for TA_ACTIVE_REQS / TA_SUMMARY consumers.
- Verified: 774 rows ≈ 189 Open / ≈ 215 stale. (Previously truncated to ~417 — the root cause of the 189-vs-217-vs-182 open-count confusion.) Plus 1 blank header row leak (see Known Debt).
- Enrich Step 3 closes anything with balance ≤ 0.
- enrich RPC Step 4 rebuilds data_cache.ta_reqs WHERE status='Open' AND balance>0 — what sbBoot reads on page load.
- Bug fixed (1368754): prior broader status mapping inflated open reqs by ~400 (rows leaking into data_cache.ta_reqs). Now Open only when zinghr_status==='Approved', so ta_reqs reflects true open count.

### Two-Status Model — LOCK THIS IN (18 Jun 2026)
- `req_status` = AUTO. Owned by ZingHR/TAT-enrich. Synced by the RPC after every upload.
- `status` = MANUAL RMG override. The Close action (not yet built) will own it.
- This split is what prevents upload-clobber / bounce-back: a TAT re-upload refreshes
  req_status without ever overwriting an RMG manual decision in `status`.

### TAT Enrichment + sync RPC (18 Jun 2026)
- `sync_positions_from_req_tracker()` RPC created (migration: sync_positions_from_req_tracker_20260618).
  Updates account_positions_log.req_status = rt.status, req_age_days = rt.tat WHERE rt.req_id = apl.zinghr_req_id.
  Only touches those 4 cols (req_status, req_age_days, updated_at, updated_by). Manual fields untouched.
- Called in processReqTAT immediately after enrich_req_tracker_from_accounts RPC.
- Vocab alignment: status filter pills changed from ['All','Approved','Closed'] → ['All','Open','Closed'].
  submitRMGNewPosition default req_status changed from 'Approved' → 'Open'.
- STEP 5 verification (after next TAT upload): confirm req_status/req_age_days updated on matched rows;
  non-matching zinghr_req_id rows untouched; manual fields (status, notes, demand_class) untouched.

### RMG Filter-Responsive Cards (18 Jun 2026)
- `cardBase = data.slice()` inserted after Region+Status filters, before Flag filter (line 8715).
- Five KPI cards (Total / No-Elah / RED / AMBER / GREEN) now read `cardBase` (Region+Status filtered).
- Flag pill narrows grid rows only (via `data`); cards stay on Region+Status breakdown.
- Behaviour: All→full counts; North→North counts; North+Open→North open counts; Flag pill→rows narrow, cards unchanged.

### RMG Users + Sidebar Scoping (18 Jun 2026)
- Added 4 RMG users (SQL: `June Build/seed_rmg_users_18Jun2026.sql`): Sonal Rale,
  Pruthvi SS, Nehal Shaikh, Chander Mohan — all role='rmg', function_type='rmg',
  region='Pan India'. SQL matches profiles to auth.users by email; idempotent.
  PREREQUISITE: the 4 Supabase Auth logins must be created first (Dashboard → Authentication → Users).
- Code: RMG sidebar scoping block in showSidebarAfterLogin shows only Overview,
  RMG Workspace, TA Pipeline, Prospective Joiners, Resignation & Backfill, Long
  Absenteeism, Account Health; hides everything else + submit + empty groups.
- Code: Long Absenteeism "Update" button now gated — RMG sees "View only".
- Status: edits node --check PASS; PENDING commit + push and SQL run.

---

## Pending Phases / Build Sequence — What's Next

### NEXT (in order)
1. **STEP 5 verification** (after next TAT upload) — confirm req_status/req_age_days populated in account_positions_log; manual fields untouched; non-matching rows unchanged. Run 3 queries in KNOWLEDGE §TAT Enrichment above.
2. **Commit + push** 18 Jun RMG sidebar scoping; run RMG users SQL after auth accounts created.
2. **Designation fix** — contract_workforce parser + eSep enrichment join (prompt written, not committed).
3. **Phase 6A — Resignation backfill trigger** — eSep upload flags resignations needing backfill; RMG review queue (Raise Req / Deploy from Bench / No Replacement). Designation enrichment: join contract_workforce by emp_code at eSep parse time. Design complete.
4. **HC Budget module** — after Elah/ZingHR reconciliation (469 mapping exceptions). Elah total 2,529 vs ZingHR 2,621 (gap 92; 11 multi-account tagged; bench in Elah 146). Schema: hc_budget table when data is clean.
5. **ATE stale record cleanup** — after Ramesh confirms ZingHR headcount (61 active in tracker vs 27 last upload).
6. **Resignation Past LWD section** — 211 already-left employees; design pending (collapsed vs escalation panel). Phase 8B priority.
7. **Account Health Phase 4** — Position Lifecycle panel + composite health score (100pts: Deployment gap 35 / Attrition 25 / Absent burden 20 / HRBP visit recency 10 / Open critical reqs 10), RAG <50/<70/>70, billing type badge (T&M red, MS blue, AMC amber). Deployment gap (T&M only): (contracted_hc − (active_hc − absent)) / contracted_hc. HC source priority: Elah HC (<7d) → Super Emp → headcount.
8. **Phase 5 — HRBP Scorecard redesign** — 5 dimensions (Workforce Stability 25, Account Coverage 25, ER & Connect Quality 20, TA Partnership 15, Deployment Health 10). Cap ≤70 if regional attrition >25%. Leaderboard open to all roles; calendar-month visit cadence. Geo-tagging dropped.
9. **Attrition Prediction (Phase 9)** — attrition_history created, 0 rows. Blocked on Ramesh's Jan–May 2026 eSep history. computeSeasonalIndex() from YYYY-MM keys; computeBackfillForecast(): base rate × seasonal + (uncontacted absent × 0.7) + eSep initiated (0.7 factor confirmed). Show as range.

### Phase 8 — Tab Redesign Sub-Specs (detail carried from earlier planning)
- **8A TA tab rebuild:** pipeline health strip (5 chips from req_tracker); Plat/Gold critical aging (reqs >45d, T&M flagged) as top priority; aging table with T&M badge + tier/region filter; recruiter scorecard filtered to function_type='ta'; offer→join funnel; unlinked-candidates chip.
- **8B Resignation:** Past LWD escalation panel (211 employees, T&M billing impact); backfill_req_id inline editable for RMG; position lifecycle indicator; absconding auto-link; attrition by account cluster (2+ in 90d).
- **8C Prospective Joiners:** auto-create from candidate upload at "Offer Accepted"; account-wise joining pipeline; at-risk joiners (DOJ >30d from offer); DNJ tracker; auto-close linked req when DOJ passes.
- **8D OD/WFH:** OD backlog by region; manager compliance league; account-wise OD pattern (T&M scope-creep signal); OD-not-regularised → absent_cases link.
- **8E Absenteeism:** triage strip (4 urgency buckets); HRBP action queue; account impact (Plat/Gold billing); absconding pipeline with FnF status; pattern analysis; days-to-first-contact metric (feeds HRBP D1).

### On the Horizon
- Transfer Detection: week-on-week customer-tag change in Super Emp Master — low-effort parser, not built.
- Customer Contract Expiry: 3 new cols on customer_accounts (contract_end_date, renewal_status, contract_notes) — no data yet.
- Elah parser (processElahDeployment) — Phase 6, URGENT before Elah sunset (target was July 1 2026).

---

## Supabase Tables — Complete List

### HR CC Owns
- weekly_reports, weekly_submissions, user_profiles (31 rows — see below)
- data_cache: keys fnf, resignation, od, summary, account_hc, ta_summary, elah_deployment, elah_expiry_risk, elah_ramp_down, ate_dbt_summary, esep_codes
- absent_cases: 323 rows. Uses `.select('*')` — all cols including region.
- prospective_joiners, attrition_history (empty), bench_deployments (empty)
- ate_tracker: ATE cases. Cols include fte_emp_code, fte_confirmed_at, fte_confirmed_by (added 09 Jun)
- ate_advance_log: Sheetal's monthly advance + recovery log
- hrbp_site_visits (18 cols), hrbp_visit_plans, hrbp_er_cases, hrbp_retention_cases, hrbp_discipline_cases, hrbp_photos
- workforce_intel (id='current'), resignation_tracker, req_tracker, res_summary
- action_items, customer_accounts (150 rows, billing_type/contracted_hc/elah_hc/elah_name)
- account_positions_log (393 rows after June load — see schema below)
- elah_deployment (empty), planned_leaves (empty)
- training_sessions (31 rows seeded), training_pipeline

### account_positions_log Schema (17 Jun 2026, updated 18 Jun 2026)
393 rows after June load. Migration: rmg_positions_log_columns_and_rls (20260617).
req_status and req_age_days now synced from req_tracker via sync_positions_from_req_tracker() RPC on every TAT upload.
RLS: 4 policies (aplog_select/insert/update/delete).
Key columns: zinghr_req_id, account_name, region, position_type, designation,
elah_demand_id (text — Elah demand line, entered by RMG), demand_region,
emp_category (FTE/ATE/Retainer), backfill_emp_code, backfill_emp_name,
req_status (Open/Closed — vocab aligned 18 Jun; auto-synced from req_tracker, do not hand-edit), req_age_days, in_elah (bool), days_vacant
(GENERATED STORED), updated_by, updated_at, created_at.
Seed breakdown — position_type: Backfill—Resignation 188, New Position 67,
Backfill—Internal Movement 22, Redeployment 1. req_status: Approved 208, Closed 66,
null 4. regions: West 163, South 90, null 24, East 1. in_elah: true 249, false 29.

RMG workflow (until ZingHR position code goes live): RMG sees vacancy → raises req
in ZingHR (gets req_id) → enters zinghr_req_id in HR CC (Resignation tab for
backfills, Account Health for new positions) → dashboard joins resignation ↔ req ↔
candidates ↔ offer ↔ joiner → Position Lifecycle panel shows the chain.

### Shared With OPS360 — FLAG BEFORE TOUCHING (additive only, no drops/type changes)
workforce_intel, absent_cases, resignation_tracker, prospective_joiners,
planned_leaves, employee_account_mapping (not yet built), account_positions_log.

### Postgres Functions
- normalise_account_name(raw_name text) → text (pg_trgm, similarity >0.4; exact →
  elah_name → fuzzy → raw). Shared with OPS360 — call, don't reimplement.

---

## Upload Parsers

### ZingHR Super Employee Master (Ramesh — weekly)
- File detection: cmsitn prefix (must NOT catch TA files).
- Populates: _acctHC (composite ACCOUNTNAME|||REGION + flat keys), workforce_intel, ate_tracker.
- ATE identified by Grade (col CM) = 'ATE' via isATE(r) — NOT EmpType (col CK). FTC/Consultant/CONTRACT remain EmpType-based.
- ATE reconciliation pass: auto-confirms fte_emp_code against active FTE rows; flags missing ATEs.
- HC filter: uses col 3 "Employee Status" (Existing/NewJoinee/FnF), NOT col 7 "EmployeeStatus" (Active/Inactive).
- Attrition DOL: header is 'DOL' (added first in lookup). Without it attrition reads 0.
- Error handling: try-catch wrapper logs "Super Emp upload error: [message]" and exits cleanly.
- 09-Jun numbers: activeRows 2,709 · WI HC 2,749 · 380 HC cache keys · ATE 65 · FTC 106 · Consultant 324 · Contract 89.

### ZingHR eSep Transaction (Ramesh — monthly)
Populates CMS_RES_DATA, workforce_intel attrition. HC denominator: hc_snap[mon] || 2830.
Scoped to 2026 exits only. Excludes FnF Locked and revoked from CMS_RES_DATA (intentional).

### ZingHR Req TAT / Candidate Transactional (Ramesh — weekly)
normaliseCustomer() on every customer field. billing_type from customer_accounts.
_reqMap from TA_ACTIVE_REQS. _unlinked flag on unmatched candidates. processReqTAT
now upserts req_tracker directly (live source). Candidate upload at "Offer Accepted"
should auto-create a prospective_joiner record.

### ATE Advance Upload (Sheetal — monthly)
File detection: ate_advance or ate_dbt. Two sheets: "Advance Paid" + "Recovery".
Header auto-detect (scans first 5 rows). processATEAdvance(wb) — LIVE.
Template: ATE_Advance_Upload_Template.xlsx (shared with Sheetal).

### Elah Resource Deployment (Ramesh — weekly, until sunset)
Parser processElahDeployment(wb) — NOT YET BUILT (Phase 6 — URGENT). File detection:
resource_deployment / elah / deployment_detail. Reads "Billed Resource" sheet; builds
per-account HC, contract expiry register, ramp-down register.

### Planned Leaves (Ramesh — weekly)
Parser NOT YET BUILT. Filename prefix TBC from Ramesh.

---

## ATE Management
- **NATS formula:** ROUND(region_fte_hc × 0.10) per region — dynamic from workforce_intel.
- **DBT flow:** DBT goes directly govt → employee bank. CMS never receives it. CMS exposure = advance paid − recovered. Sheetal logs advance (Sheet 1) and recovery after DBT intimation (Sheet 2).
- **ate_tracker reconciliation (09 Jun):** 4 exited ATEs set Active → Separated. DO NOT TOUCH 22007479 Ashish Raj (Conversion Initiated ✓). 22003768 Divya Goel: tracker=Separated (13-Apr) but 09-Jun master=Existing+Grade ATE — confirm rejoin vs stale tag.

---

## Billing Types — Key Reference
32 T&M / 113 MS / 2 AMC. T&M accounts (deployment gap = direct billing loss):
IHCL, Reliance Corporate IT Park, ONGC, Torrent Power, Poonawalla Fincorp,
Sutherland, Religare, Titan, Cadence, Bajaj Finance, Syngene, Dixon, SMS India,
Vitech, Clix Capital. Deployment gap calc applies ONLY to T&M accounts.

---

## Workforce Intelligence — Current Numbers
- Active HC: 2,749 (Super Emp Master, 09 Jun 2026). Account HC cache (activeRows): 2,709. Gap of 40 = FnF Initiated/transitional.
- Region HC: North 502 / South 660 / East 502 / West 1,088.
- Attrition (annualised): Jan 34.6% · Feb 20.5% · Mar 19.9% · Apr 1.2% · May 19.1%.
- WI is a large hardcoded JS const (~line 845); Supabase patches it at boot. Attrition scoped to current year (2026).

---

## User Profiles — 31 Users

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
| Sonal Rale | rmg | rmg | Pan India |
| Pruthvi SS | rmg | rmg | Pan India |
| Nehal Shaikh | rmg | rmg | Pan India |
| Chander Mohan | rmg | rmg | Pan India |

Notes: TA users corrected from role='hrbp' → role='ta' on 24 May 2026. Recruiter
scorecard filters by function_type='ta'. RMG rows added 18 Jun 2026 (require auth
accounts; see seed SQL). RMG access = full edit on RMG Workspace; read-only on TA
Pipeline, Prospective Joiners, Resignation & Backfill, Long Absenteeism, Account
Health; all other tabs hidden.

---

## HRBP Region Assignments
Abhishek Singh: North · Surajit Sen: East · Priya Paul: South ·
Shambhavi Prathamesh Joshi: West · Somasri Sukumar Samanta: West.

HRBP_REGION lookup objects (two — keep in sync):
1. HRBP_REGION const (full-name keys, line ~1654)
2. HRBP_REGION_MAP const (first-name based, line ~4336)

---

## Absent Cases — Status Values (exact as stored)
"Uncontacted" | "OD Not Regularised" | "Absconding — eSep Initiated" |
"Resolved — Resigned" | "Resolved — Returned" | "Missed Punch" |
"On Leave (Unplanned)" | "Resignation — HR Chain Pending" |
"Under Investigation" | "Warning Letter Issued" | "Medical / Emergency".

Resolution filter must use indexOf('resolved') not includes() — em-dash may cause
Unicode mismatch.

---

## Pending People Actions

| Person | Action | When |
|---|---|---|
| Alex | Create 4 RMG Supabase Auth accounts, then run seed_rmg_users_18Jun2026.sql; commit + push RMG sidebar change | This week |
| Mohit | (Resolved 18 Jun — RMG emails received: sonal.rale, pruthvi.ss, nehal.shaikh, chander1.mohan) | Done |
| Ramesh | Confirm actual ATE headcount in ZingHR (61 active in tracker, last upload 27) | ASAP |
| Ramesh | Upload Jan–May 2026 eSep historical data for attrition_history | Pending |
| Alex/Team | Elah/ZingHR reconciled master (469 mapping exceptions) before HC Budget module | Ongoing |
| Alex | Add Tier + Billing Type + Contract End Date + Contracted HC in ZingHR customer master | Before June 30 |
| Alex | Add Billing Type + Employee Category (FTE/ATE/Retainer) in ZingHR recruitment module | Before June 30 |
| Kesavan | Fill contracted_hc for Plat/Gold via Admin UI | This week |
| Sheetal | Use ATE_Advance_Upload_Template.xlsx from June 2026 | June 2026 |
| RMG team | Enter zinghr_req_id in HR CC for each backfill/new position | After Phase 6A |

---

## ZingHR Attributes to Add
Recruitment module (Alex): 1) Billing Type — T&M / Managed Services / AMC / Hybrid
(mandatory); 2) Employee Category — FTE / ATE / Retainer (mandatory).
Customer master (discuss with ZingHR team): Account Tier; Billing Type; Contract End
Date; Contracted HC. When live: Manage Customer Accounts tab becomes read-only +
computed columns; contracted_hc stays manually editable until ZingHR has the field.

---

## Key Team Members
- Alex Augustine: admin, project owner
- Kesavan: Head of HR Ops / HRBP / Payroll / Compliance — executive (function_type hrbp)
- Shambhavi Prathamesh Joshi / Somasri Sukumar Samanta: HRBP West
- Surajit Sen: HRBP East · Priya Paul: HRBP South · Abhishek Singh: HRBP North
- Mohit Kumar: HR Ops / Payroll (hrops)
- Sheetal Sadashiv Pachangane: Payroll — ATE advance uploads
- R Ramesh: Payroll — all ZingHR weekly uploads + Elah report
- Deepak Kumar Shetty: L&D · Ashok B C: Facilities
- Neha Kaur Sammi: TA Lead (Pan India) · Ajith Inguva: TA (Pan India)
- TA: Mousumi/Caral (South), Pijush (East), Salma (North), Dhananjay/Juee/Kumari/Nikita/Nilam (West)
- RMG: Sonal Rale, Pruthvi SS, Nehal Shaikh, Chander Mohan (Pan India)
- Amit Dasgupta: Delivery leadership (ATE initiative owner)

---

## Action Centre
Tasks from action_items table. due_date renders inline (overdue red). Status: Open /
In Progress / Closed / Done. Admin/hrops can add and edit. Non-admin users see
My Tasks / All Tasks toggle. Known gap: confirm owner field renders inline on live.

---

## OPS360 Relationship
Supabase shared project. OPS360 reads: customer_accounts, account_positions_log,
ate_tracker, bench_deployments, elah_deployment, attrition_history,
normalise_account_name(). Shared read/write: workforce_intel, absent_cases,
resignation_tracker, planned_leaves. OPS360 owns (HR CC may read in future):
normalized_tickets, sla_contracts, delivery_hierarchy, itsm_adapters. When OPS360 is
ready: add ops360_read RLS policy to HR CC tables (no schema changes). Principle:
OPS360 reads HR CC data, never rebuilds parallel stores.

---

## Account Migration Plan
Currently on personal Vercel + Supabase + GitHub. Migrate to CMS enterprise accounts
after OPS360 prototype is validated. Sequence: Supabase org transfer first (keys
unchanged) → Vercel team setup with custom domain → GitHub repo transfer.

---

## Known Debt — Carry Forward
- Header row leak: req_tracker contains req_id = "Requisition Wise TAT Report" (ZingHR report title parsed as data row). Cosmetically harmless — no customer match, enrich ignores it. Fix: add `if (!rec.req_id || !/^\d+$/.test(rec.req_id)) continue;` in processReqTAT loop.
- ATE stale records: ate_tracker has 61 Active but Super Emp shows 27. Cleanup awaiting Ramesh headcount confirmation.
- contract_workforce: ~50 ATE rows that should not be there. Cleanup pending.
- Resignation Past LWD: 211 already-left employees in main table. Design pending (collapsed vs escalation panel, Phase 8B).
- Account Health last visit: _hrbpSVCache has no region field — last visit is pan-account.
- Duplicate switchTab: ~line 3845 dead, ~line 4668 live. Clean up in a future session.
- planned_leaves parser: filename prefix TBC from Ramesh.
- CEAT double-space duplicate in ZingHR: "CEAT LIMITED" + "CEAT  LIMITED".
- LIC naming: "LIC (EAST)" vs "LIC ( EAST )" — standardise in ZingHR.
- WI hardcoded fallback (~line 845): stale values; self-corrects on uploads.
- dbt_months_received + advance_recovered on ate_tracker: redundant (ate_advance_log is source of truth). Keep schema, don't display.
- Action Centre owner field: confirm renders inline on live.
- Confidential-search flag for RMG Workspace: not yet built (RMG holds all reqs incl non-delivery/corp; a few confidential searches need exclusion).
- Vercel MCP 403 on team scope: recurring (2 sessions) — needs connector re-auth.

---

## When to Update This File
UPDATE AFTER (not during): schema change in Supabase (table list + columns); phase
completes (line count + commit hash); design decision confirmed; new upload parser
added (file detection + what it writes); role/access change (user profiles section);
bug fixed (remove from Known Debt); new pending item (Pending People Actions).

DO NOT update for: work in progress, items being debated, analysis outputs (Excel
reports, reconciliation files).

This file (KNOWLEDGE.md) is the single living source of truth — keep it current and
do not fork new "knowledge" files.
