# CMS HR Ops Command Centre — Project Knowledge
**Version:** 3.19.1 | **Last updated:** 13 Jul 2026 — Account Health deployment% + revised score live (+ blank-tab runtime fix). Remaining: Elah parser, inline owner/WFH columns (#26); owners need assigning.

> **This is the single source of truth for the project.** It replaces the older
> `HRCC_Project_knowledge.MD` and `cms_hr_cc_knowledge_v2.md` files. Update this
> file (and only this file) at the end of every session. See the update protocol
> at the very bottom.

---

## Recent Updates (Session Log)
> Newest first. Add a dated entry here at the end of every session.

### 13 Jul 2026 — Account Health deployment% + revised health score (commit a6f45e3)
- **CUSTOMER_LIST cache now carries budget + billing (fixes audit gap):** sbBoot load + updateAcct re-sync fetch contracted_hc,billing_type and push `budget`/`billing` onto each CUSTOMER_LIST entry (previously dropped — per-account billing_type now available in the cache).
- **Deployment% on Account Health cards:** deployPct = activeHC/budget (activeHC from _acctHC; budget=contracted_hc). Shown with sub `activeHC / budget`, green≥90 / amber 70-89 (or >110 'over') / red<70. T&M gets a purple T&M badge; MS/AMC show billing faintly + deployment is informational.
- **REVISED health score (Alex-directed): FnF REMOVED from score (shown as info only), open-critical-reqs weighted UP.** Weights via quality-fraction × weight: **T&M (with budget):** Deployment 25 + TA/critical-reqs 25 + Absent 20 + Attrition 20 + Visit 10. **MS/AMC or T&M-no-budget:** TA 30 + Absent 25 + Attrition 25 + Visit 20. deployF tiers: ≥95→1, ≥85→.7, ≥70→.35, else 0. Thresholds unchanged (≥80 HEALTHY / ≥55 AT RISK / else CRITICAL). Old score was Absent25/FnF25/Visit20/Attr20/TA10. **budget_hc (150/152 filled) is now a live, scored signal — the master pays off.**
- **Blank-tab bug fixed (8d06a08):** the deploy change renamed `hcCol`→`dpCol` but left a stale `hcCol` reference in the `segs` array (~line 8039) → runtime ReferenceError inside `cards.map` → Account Health rendered blank. node --check passed (valid syntax). **RECURRING LESSON (3rd time today after RMG canView/_currentUser): node --check ≠ runtime. After any RMG/Account-Health/shared-render edit, OPEN THE TAB in the browser before calling it done. Renames are the classic trap — grep for the OLD name after renaming.**

### 13 Jul 2026 — Roster-driven HRBP validation + inline owner/WFH columns (1896433; #26 pending)
- **Accounts-upload validation roster-driven (1896433):** `handleCustomerAccountsUpload` no longer hardcodes the 6 HRBP names — fetches `user_profiles` (active, role='hrbp', function_type='hrbp') into `VALID_HRBP_SET` (lowercase display_name), blank always allowed, fetch-fail falls back to blank-only + console.warn. **Alex caught that I'd hardcoded the roster in a prompt — hold the line: NO hardcoded HRBP/TA/Ops names; always source from user_profiles.**
- **PENDING #26 — inline owner/WFH columns:** Manage Customer Accounts inline table (renderAccountsTab) currently shows only Region/Tier/HC/Active; hrbp_owner + wfh_approved are template-only (not visible inline — Alex flagged). Prompt written: add HRBP OWNER dropdown (options from user_profiles roster, NOT hardcoded) + WFH OK checkbox, reusing the existing per-field save.
- **Still hardcoded (migrate for full consistency, non-urgent):** HRBP_REGION_MAP (all-staff name→region), TA scorecard `_HRBP` exclusion list.

### 13 Jul 2026 — RMG 'Internal' elah + WFH-approved (ca874ac; migration; #25 pending)
- **'Internal' elah (ca874ac):** `_rmgReconFlag` now short-circuits to green "Internal" when elah_demand_id (any case) === 'internal', BEFORE the RED/no-elah-line check — so enablement/internal roles (no Elah demand line) reconcile clean. Set by typing "Internal" in the RMG grid's editable Elah ID cell. (Prompt indent note: FIND blocks must match the file's exact spacing — terminal read exact indentation before applying.)
- **WFH-approved DONE (331e95e):** `customer_accounts.wfh_approved boolean default false`. Template download/upload now includes wfh_approved (TRUE/FALSE; parses true/yes/1). `_loadWfhApprovedSet()` at boot → `_wfhApprovedSet` (normalized names of wfh_approved=true accounts). OD/WFH tab (`renderOD`) has "Hide WFH-approved accounts" toggle (default ON) — excludes those WFH rows from table + footer count with a hidden-count note. OD rows unaffected.

### 13 Jul 2026 — Ops Review FnF priority panel (commit 1a691ac)
- `_loadOpsRevFnf()` (#opsrev-fnf, synchronous — reads FNF_DATA in-memory). Pending = not paid & status !~ paid/settled; Overdue = pending & payment_date<today; High-Value = pending & net_salary≥threshold. KPI strip (Pending/Overdue/High-Value) + priority table (overdue-first then net_salary desc, Overdue/High-Value chips, ₹X.XL formatting). Empty-data → amber "upload FnF report" nudge. **OPS REVIEW TAB now 6 panels: submissions, absenteeism, visits, TA, ATE/contract, FnF — all in loadOpsReview().**

### 13 Jul 2026 — Ops Review ATE/Contract panel + oversight boot-select FIX (19deb97, 5b0bcbb)
- **ATE & Contract Expiry panel (19deb97):** `_loadOpsRevAteContract()` (#opsrev-atecontract). ATE Overdue = ate_tracker status='Active' AND training_end_date<today (≤14). Contract Overdue = contract_workforce past_superann OR lwd_crossed OR ftc_18m_risk (~7; lwd_crossed & ftc_18m_risk currently 0). No-End-Date = 420 (data gap, amber nudge, NOT overdue). Attribution by account/customer→ACCT_OWNER→regionSoleHRBP→Unassigned (mirrors absenteeism). KPI strip + By-HRBP table + critical list (cap 15).
- **OVERSIGHT WAS INERT — FIXED (5b0bcbb):** sbBoot's `user_profiles` select (~line 13099 / earlier ~byte 1096511) did NOT include `oversight_regions`, so `_sbRole.oversight_regions` was always undefined → `_myRegions()` fell back to single region → Surajit saw only East despite all 5 tabs being wired. Fix: added `oversight_regions` to the boot select. **LESSON: when adding a user_profiles column that runtime logic reads off `_sbRole`, it MUST be added to the sbBoot select or it's silently undefined.** `_sbRole` now carries oversight_regions (and confirm `active` if ever read off _sbRole — currently only queried directly).

### 13 Jul 2026 — Ops Review Visits module → dynamic roster (commit 32d4b32)
- `_loadOpsRevVisits()` (#opsrev-visits): KPI strip Planned/Done(period)/Overdue/Open ER(+SLA); By-HRBP table from user_profiles active field-HRBP roster (replaced hardcoded VISIT_HRBPS), exact display_name match to hrbp_name, "Unassigned/Other" row for non-roster visits, period-aware Done via _pStart/_pEnd; critical site issues list (issues_flagged, cap 15). **TA module DONE (8efffc6):** `_loadOpsRevTA()` — Open/Critical/Plat-Gold via `_taBuildRecCounts(TA_ACTIVE_REQS)`, Closed-in-period from req_tracker(status=Closed, filled_date in period), FnF pending/overdue from FNF_DATA global; KPI strip + recruiter table sorted critical→open, period toggle drives Closed only. **OPS REVIEW TAB COMPLETE** — 4 modules: submission status (1cf9e5d), absenteeism (04b5ed8), visits (32d4b32), TA (8efffc6). Caveats: Closed(period) under-counts until filled_date backfills via more TAT uploads; FnF reflects last upload (not real-time).

### 13 Jul 2026 — Ops Review HRBP weekly submission panel (commit 1cf9e5d)
- `_loadOpsRevSubmissions()` renders FIRST in Ops Review (#opsrev-submissions before #opsrev-absent). Current week = weekly_reports max(week_num) → week_key+label (W30 "06 Jul – 10 Jul"). Roster = active field HRBPs (user_profiles role='hrbp'+function_type='hrbp'). Matches weekly_submissions (week_key + function_type='hrbp') by submitter_email (fallback name). Chip "S of T submitted" (green all / amber any pending); table HRBP | Status, Pending sorted first. Confirmed: HRBP weekly form NOT discontinued — weekly_submissions active every week through W30; site visits logged via the weekly form (hrbp_site_visits insert ~byte 774856) through 10 Jul.
- **Oversight scoping (#16) — helper + Long Absenteeism DONE (bf2f5c6):** globals `_myRegions()` (oversight_regions comma-list → array; else [region]; null=pan-india), `_regionInScope(reg)`, `_isMultiRegion()`. Long Absenteeism now: single-region HRBP unchanged; multi-region (Surajit) unlocked + filtered to East+North + pills limited to "All my regions"/East/North; admin/hrops unchanged. Account Health + OD/WFH + Resignation converted (4931e25, same pattern — multi-region filtered via _regionInScope, pills "All my regions"/East/North). HRBP tab done (af29577): view-widen (_pjRegion, _regionInScope filter, "All my regions" pills) + `filterVisitDropdown` pool now iterates `_myRegions()` (getCustsByRegion per region) so Surajit can log North visits. **OVERSIGHT SCOPING COMPLETE — all 5 tabs (Absenteeism, Account Health, OD/WFH, Resignation, HRBP).** Pattern for any future senior with oversight_regions: set the field, done. Prior model text: Model: Surajit East+North = view both + enter North visits (via the weekly form's account picker); casework actions org-owned by North HRBP (not hard-blocked). Region-lock sites to convert (bytes): 650232 HRBP tab _autoRegion, 558523 absenteeism, 841282 Account Health, 886579 Resignation, 898647 OD/WFH (TA 921617 / RMG 976752,1002793 / FnF 864672 are non-HRBP, skip).

### 13 Jul 2026 — Foundation: dynamic roster + owner attribution (migration + commit 04b5ed8)
- **Migration `add_user_profiles_active_20260713`:** `user_profiles.active boolean default true`. Onboard = add profile; **exit = set active=false** (keeps history; rosters exclude inactive).
- **Ops Review absenteeism rewired (04b5ed8):** `_loadOpsRevAbsent` no longer hardcodes HRBP names. fieldHRBPs = `user_profiles` where role='hrbp' AND function_type='hrbp' AND active (so Prapti auto-included, Kesavan/executive auto-excluded). Attribution `ownerOf(case)`: customer_accounts.hrbp_owner for the case's customer (normCust = trim/upper/collapse-space) if it's a field HRBP → else region-with-exactly-one-HRBP fallback (North/South/East resolve; West's 3 → no sole owner) → else Unassigned. Actioned/Resolved matched by `status_updated_by` (email) === HRBP email (exact, no more first-name guessing — user_profiles has email). ONE consolidated table: HRBP | Region | Pending | Actioned | Resolved | Overdue + amber Unassigned row + "N accounts have no owner" nudge linking to Manage Customer Accounts. Period toggle retained.
- **This is the reuse PATTERN** for all future by-HRBP views (visits, ATE/contract, TA): roster from user_profiles, attribution via customer_accounts.hrbp_owner. **Still hardcoded elsewhere (migrate later):** HRBP_REGION_MAP (all-staff name→region), TA scorecard HRBP-exclusion `_HRBP` list. **Owners must be assigned via the account template before attribution fully resolves — until then expect a large Unassigned bucket (by design).**

### 13 Jul 2026 — HRBP ownership model + customer_accounts as master (migration + commit 0fb46d9)
- **Org reality:** region ≠ HRBP. West has THREE HRBPs (Prapti Patel, Shambhavi Joshi, Somasri Samanta); North=Abhishek, East=Surajit, South=Priya. Surajit is senior manager overseeing **North + East** (full rights). Kesavan=executive/hrbp/Pan India.
- **Migration `add_hrbp_ownership_and_oversight_20260713`:** `customer_accounts.hrbp_owner text` (ONE HRBP owner per account — the attribution source of truth) + `user_profiles.oversight_regions text` (comma list; overrides single region when set). Surajit set `oversight_regions='East,North'`.
- **Account template (0fb46d9):** Manage Customer Accounts tab now has ⬇ Download Template + ⬆ Upload Accounts. `exportCustomerAccountsTemplate()` writes xlsx (account_id | name | region | tier | billing_type | budget_hc | elah_name | hrbp_owner | active | headcount_readonly; budget_hc↔contracted_hc; headcount read-only). `handleCustomerAccountsUpload()` validates region/hrbp_owner/billing_type(T&M/MS/AMC)/budget_hc, classifies by account_id (present+exists=UPDATE, blank=INSERT new via crypto.randomUUID, present-not-found=invalid), shows preview modal (N new/M updated/K invalid) before `_applyAccountsUpload()`. Never deletes; never writes headcount/elah_hc. **customer_accounts is now the intended single source of truth for billing_type/tier/budget_hc/region/hrbp_owner.**
- **Data:** billing_type set on all 152 (118 MS / 32 T&M / 2 AMC); budget_hc on 150/152.
- **PENDING:** (A ✅ template) → (B) owner-based attribution (Ops Review/absenteeism attribute via employee.customer→customer_accounts.hrbp_owner, not region; West splits) → (C) oversight scoping (honor oversight_regions; Surajit East+North). - **Reads-from-master AUDIT (done):** (1) `CUSTOMER_LIST` (hardcoded literal ~byte 279409) IS overwritten from customer_accounts at boot in sbBoot (~byte 1167900: `CUSTOMER_LIST.length=0` + repush name/region/tier/headcount — but NOT billing_type; active-only) and re-synced after admin edits. So it's a live cache, not stale (literal serves only if the boot fetch fails). (2) TIER + REGION: master-sourced (via CUSTOMER_LIST cache `c.t`/`c.r`, and `_lookupTier`/`_tierMap` built direct from customer_accounts in _loadHRCycleData). (3) BILLING_TYPE: NO hardcoded T&M list exists; processReqTAT writes billing_type=null then the DB RPC `enrich_req_tracker_from_accounts` fills tier+billing from customer_accounts → req_tracker → TA reads r.billing_type. Trust chain valid but depends on the RPC firing; CUSTOMER_LIST cache carries NO billing_type (per-account billing UI must use req_tracker or a fresh fetch). (4) **contracted_hc / budget_hc = WRITE-ONLY: maintained in customer_accounts + admin template but CONSUMED NOWHERE. renderAccountHealth has NO deployment-gap / budget-vs-actual calc — that feature is not implemented.** → biggest opportunity: build Account Health budget-vs-actual so budget_hc is actually used (Phase 4 deployment gap: T&M (contracted_hc − (active − absent))/contracted_hc).

### 13 Jul 2026 — NEW Ops Review tab — weekly team review (commits b630d8b, 9fac9f9)
- **Purpose:** leadership weekly-review cockpit — "supposed-to / done / pending" per module, "open queue = the plan" (no new data entry). Gated **admin + executive** (Kesavan=executive included). Tab id `opsreview`, sidebar `snav-opsreview` (🗓️), hidden for all non-admin/exec roles. Functions: `renderOpsReviewShell`, `loadOpsReview`, `_loadOpsRevAbsent` (all true globals). "This week" = since most-recent Monday. Module containers: `#opsrev-absent` (done), `#opsrev-visits` + `#opsrev-ta` (placeholders for next prompts).
- **Module 1 — HRBP Absenteeism (b630d8b + member-wise 9fac9f9):** from absent_cases. KPI strip: Total Pending / Resolved This Week / Overdue Follow-ups (past next_followup_date) / Absconding-eSep. isResolved = status lowercased contains 'resolved'. Table A "By HRBP — this week": 5 HRBPs, Actioned This Wk attributed via `status_updated_by` (it's the HRBP EMAIL — match on first-name substring: abhishek/surajit/priya/shambhavi/somasri), West = Shambhavi+Somasri shared; Resolved This Wk; Region Pending. Table B: pending by status bucket (Uncontacted/OD/Absconding/Other). **Caveat: pending backlog is REGION-level — absent_cases has no per-HRBP owner field; only `status_updated_by` (who last actioned) is person-level. Adding a true owner column is a pending option if Alex wants West split per person.**
- **Module 1 fixed (60dd44f):** first build mis-grouped everyone (HRBP+TA+L&D) by region because it used `HRBP_REGION_MAP` — **that const is a name→region lookup for ALL staff, NOT an HRBP roster; never use it for an HRBP-only list.** Now uses an explicit 5-entry roster (Abhishek/Surajit/Priya/Shambhavi/Somasri). Also: the module now **respects the global period toggle** — reads `_ovFrom`/`_ovTo` (set by the This Week/Last Week/Month/Custom pills, via `_setOvPreset`/`_initOvRange`) to window "Actioned/Resolved" (Total Pending/Overdue/Absconding stay point-in-time). `_setOvPreset` re-calls `loadOpsReview()` when `#tab-opsreview` is visible. Tables: A "By HRBP — activity this period" (5 rows, per-person via status_updated_by email first-name match), B "Pending backlog by region" (4 regions, point-in-time), C status buckets. **Modules 2 & 3 must also use `_ovFrom`/`_ovTo` (period window), not a hardcoded Monday.**
- **Modules pending:** 2 = HRBP Visits & Site Issues (hrbp_visit_plans planned vs hrbp_site_visits done + hrbp_er_cases open + issues_flagged; keys off hrbp_name = true per-HRBP). 3 = TA Performance (recruiter Open/Critical/Closed-this-week/Plat-Gold via _taBuildRecCounts + FnF pending from data_cache fnf). Report-compliance module deferred (weekly_submissions: latest week W30 had only 4 submitters).
- **Data landscape confirmed:** absent_cases 274 Uncontacted + 70 OD + 32 absconding (pending); visit_plans 20 Planned/5 Cancelled, 21 site visits last 30d; hrbp_er_cases 17 Open; FnF cache has status/region/net_salary/payment_date.

### 13 Jul 2026 — RMG recon Excel button "invisible" = CSS clipping, not a gate (commit 84442c7)
- **Symptom:** RMG team could not see the ⬇ Excel button on the Reconciliation tab. **Two commits chased the wrong cause first** — b67bb8f + 459491f added `role.role==='rmg'` to every RMG gate and even removed the button's `canView` guard entirely. Neither helped, because all 4 RMG users already have `function_type='rmg'` (verified in DB) so the gate was always TRUE for them.
- **Real cause (84442c7):** the recon summary header was a `display:flex; justify-content:space-between` row with **no flex-wrap**; the KPI chips pushed the "+ Add Position" and "⬇ Excel" buttons off the right edge on normal laptop widths (same clipping visible in the very first RMG screenshot where "+ Add Positio…" was cut off). Fix: added `flex-wrap:wrap;gap:8px` to the header row and wrapped the two buttons in their own `flex-wrap` group so they drop to a visible second line instead of clipping.
- **LESSON:** When a control is "not visible" but the access gate demonstrably passes, suspect CSS/layout (overflow/clip/z-index), not the role gate. Quick zero-deploy test: zoom the browser out (Ctrl-minus) or widen the window — if the control appears, it's clipping. node --check and even role-gate audits will never surface this.
- Side effect: 459491f left the Excel button ungated (renders for anyone reaching _renderRMGGrid, which is already tab-gated) — matches original intent (export available to all RMG viewers). Left as-is.

### 13 Jul 2026 — Ramesh incremental export → Phase 1 complete (commit 8d820af)
- **Export for ZingHR (8d820af):** In the RMG Deployment header (gated `canExportMoves = admin || function_type==='rmg' || role==='hrops'`): a chip "<N> changes pending for ZingHR" (`_rmgLoadPendingCount()` uses `{count:'exact',head:true}` — count only, no rows) and a "⬇ Export for ZingHR" button. `exportRMGMovementsForZingHR()` fetches rmg_movements WHERE exported_at IS NULL, confirms with user, writes the xlsx (rmg_changes_for_zinghr_YYYY-MM-DD.xlsx, ZingHR-ready cols), THEN stamps exported_at=now()/exported_by via `WHERE id IN (fetched ids)` (never blanket; file-write-before-stamp so a failed download won't mark rows). Chip refreshes after export. hrbp/ta see neither.
- **Phase 1 COMPLETE.** The full ELAH-parity loop is live: region/customer employee mapping → Move (customer transfer) / Bench→Billable → Ramesh pulls only new changes into ZingHR. Remaining: bench-gating refinement (below) + Phase 2 Elah parser for real demand IDs/billability.
- **Bench-gating done (commit d54744f):** Deploy row actions gated by customer tag — deployed→Move only, BACKUP POOL→Bench→Bill only, Enablement→neither. employee_master has NO billability signal (all 2727 customer-tagged; emp_status only Existing/NewJoinee). Interim decided with Alex: gate by customer tag — deployed→Move only, BACKUP POOL→Bench→Bill only, Enablement→neither. Helpers `RMG_BENCH_BUCKETS=['BACKUP POOL']` + `RMG_ENABLEMENT_TAGS=['ENABLEMENT']` + `_rmgIsBench`/`_rmgIsEnablement`. Real fix needs Phase 2 billability. (Only weak bench signal today: customer tags BACKUP POOL 17, CMSIT/CMS IT 98, Enablement 1 — Alex confirmed ONLY BACKUP POOL counts as bench.)

### 13 Jul 2026 — RMG Deployment view + Move/Bench actions (commits 62451d4, a3b6c64)
- **Deployment sub-view (62451d4):** RMG Workspace now has a view toggle — `Reconciliation` (existing grid) and `Deployment` (new). `_rmgView` global ('recon'|'deploy'). 8 new globals: `_rmgSwitchView`, `loadRMGDeployment`, `_renderRMGDeploy`, `renderRMGDeployView`, `openRMGMoveModal`, `submitRMGMove`, `openRMGBenchModal`, `submitRMGBench`. Deployment view lists `employee_master` (emp_code, name, designation, customer, region, emp_status; ~2727 rows fetched via paged `.range()` batches), filterable by region/customer/search, Demand ID column is '—' placeholder until Phase 2 (Elah parser). Two actions write `rmg_movements`: Move (move_type='Customer Transfer', To Customer + Effective Date required) and Bench→Billable (move_type='Bench→Billable', Effective Date required). made_by=display_name, made_by_id=_sbUser.id.
- **Access:** `canViewDeploy = admin || function_type==='rmg' || role==='hrops' || function_type==='hrbp'`. `canEditDeploy = admin || function_type==='rmg'` (Move/Bench buttons). TA excluded from Deployment. HRBP region-locked (region pills disabled, `_rmgDepRegion` forced to their region). Kesavan (executive/hrbp/Pan India) gets pan-India read-only via the hrbp clause (region!=='Pan India' guard leaves him unscoped). Added `role==='hrops'` to RMG canView in BOTH renderRMGShell + loadRMGTab AND unhid `snav-rmg` for hrops in showSidebarAfterLogin — needed because Ramesh/Mohit are role=hrops/function_type=**payroll** (would be missed by a function_type-only gate).
- **Two bugs fixed same day (a3b6c64):** (1) `canView is not defined` in `_renderRMGGrid` — the Excel button (1ba8f40) referenced canView but that function only declared canEdit → Reconciliation view threw on load (caught by loadRMGTab try/catch, surfaced as "Failed to load: canView is not defined"). **node --check did NOT catch it — runtime ReferenceError, not syntax. Lesson: manually open any view whose shared render fn was touched.** Fixed by declaring canView in _renderRMGGrid. (2) Duplicate customers in all three dropdowns (Deploy filter, Move, Bench) because `customer_accounts` has multiple rows per name (one per region/tier) — fixed with seen-object dedup-by-name at point of use. Caveat: Move modal now takes the FIRST row per customer name for its `name|region` value, so `to_region` is arbitrary for multi-region customers — revisit when demand-level data lands.
- **Two more runtime bugs fixed (d5d4946):** (1) recon Excel button did nothing — `exportRMGWorkspaceXlsx()` read `_currentUser` (never declared anywhere; correct global is `_sbRole`) → ReferenceError on click. (2) Deployment customer dropdown listed ALL customers regardless of region — now pre-filters `_rmgAccounts` by `_rmgDepRegion` before dedup, and the region pill resets `_rmgDepCustomer='All'` on change. **Both were runtime errors node --check can't catch (undefined global; logic). Standing rule now: for any RMG/shared-render edit, manually open the view AND click the button before committing.**
- **Commit chain (13 Jul):** 5b6c442 Position col · 1ba8f40 Excel export · 62451d4 Deployment view + Move/Bench · a3b6c64 canView+dedup fix · d5d4946 _sbRole+region-filter fix. (6e8e142 = TA owner unification, earlier.)

### 13 Jul 2026 — RMG Workspace Excel export (commit 1ba8f40)
- **RMG grid Excel export:** "⬇ Excel" button in the RMG header, gated on **canView** (admin/rmg/hrbp/ta/hrops) so read-only reviewers can export too. `exportRMGWorkspaceXlsx()` exports the currently-filtered rows (same `_rmgRegion`/`_rmgStatus`/`_rmgFlag` chain as the grid) via the already-loaded XLSX/SheetJS lib (no new dependency). Columns match the grid incl. Position/designation. Filename `rmg_workspace_YYYY-MM-DD.xlsx`. Empty set → toast. (54 insertions.)
- **In planning — RMG deployment/movement module (ELAH-parity):** Bringing ELAH's Supply/Team/Move views into HR CC. Decisions locked with Alex: (1) transfers = **direct RMG action + full audit log** (not a request→approve workflow); (2) mapping built on **employee_master now** (has emp_code, name, customer, region, designation, emp_status — 2727 rows), demand IDs + billability enriched **later** via the Elah parser (elah_deployment currently empty); (3) Ramesh export = **incremental** (only un-exported changes, then stamp `exported_at`). Roadmap: P0 Excel export ✅ + Chander gate check; P1 `rmg_movements` table + region/customer mapping view on employee_master + Move + Bench→Billable actions + Ramesh incremental export; P2 `processElahDeployment` parser to enrich with demand IDs/billability/band/SL/rate. `rmg_movements` schema drafted (emp_code, move_type, from/to customer+demand+status+region, requested_by, request_channel, request_ref, reason, effective_date, made_by/at, exported_at/by, notes); pending Alex sign-off on request_channel + effective_date default.
- **Chander "Add Position" (open):** Gate is `canEdit = role==='admin' || function_type==='rmg'`; Chander's profile is rmg/rmg/all and login loads function_type — so on paper he qualifies. No code/data cause found → likely stale session. Action: re-login + hard refresh; if still missing, add a temporary role readout to see his loaded `_sbRole`.

### 13 Jul 2026 — TA scoreboard↔grid reconciliation + RMG Position column
- **TA single-owner attribution (commit 6e8e142):** Recruiter scoreboard was rendering from the frozen `data_cache.ta_scorecard` snapshot while the top req table read live `data_cache.ta_reqs` — they disagreed for nearly every recruiter (Ajith showed 16 open in the grid vs 13/11 on the scoreboard; Caral card 23 vs live 3; Salma 25 vs 45; plus a phantom `Unassigned` 67 and stray `Meenakshi`). Root cause: two sources never reconciled + exact-string grouping splintered names on inconsistent whitespace (stored key was `"Ajith  Inguva"` with a double space). Fix: new global `_taOwner(r)` (primary_recruiter preferred → else first non-HRBP on `ta_recruiter`; paren-strip + whitespace-collapse + trim) and `_taBuildRecCounts(rows)`. `renderTAPipeline` builds `window._taLiveCounts` from `data` **before** the recruiter filter; grid recruiter filter, scoreboard rows (active/critical/plat_gold/backlog_age overridden from live counts; union of TA_SCORECARD names + live owners; sorted active desc), and personal-KPI `_recData` all use `_taOwner`. `ta_scorecard` cache no longer trusted for active/critical/backlog (still used for pipeline/offers/joined/fill). Now clicking any recruiter yields a table whose row count == their scoreboard active.
- **Post-fix live distribution (active / crit>45d; 180 open):** Salma 45/44 · Kumari 25/15 · Nilam 18/15 · Nikita 18/12 · Juee 17/14 · Ajith 16/15 · Mousumi 10/7 · Dhananjay 8/2 · Neha 6/5 · Caral 3/2 · Pijush 1/0 · Unassigned 13/0.
- **Data flags (not display bugs):** Salma carries 45 reqs as `primary_recruiter` — real per Neha's mapping; confirm concentration is intended. `Unassigned` 13 = empty primary_recruiter + only HRBP names on the ZingHR requisition — need a recruiter assigned.
- **RMG grid Position/Designation column (commit 5b6c442):** `_renderRMGGrid` gained a `Position` column between Account and Region, sourced from `account_positions_log.designation` (507/507 populated, incl. tat_sync rows). Fixed 130px width, whitespace-normalized on display, truncated to 22 chars with full value in hover tooltip (same pattern as Account). Grid `grid-template-columns` widened 13→14 tracks in both header and data row; header label array gained `'Position'`. node --check: clean pass.
- **Ops note (recurring):** Cowork bash sandbox served a stale/truncated copy of index.html (and a stale KNOWLEDGE.md — read as 3.5.0 at session start vs real 3.10.0). Commits must be run from the local Claude Code terminal; verify `git diff` is only the intended hunks first. See Cowork sync note.

### 10 Jul 2026 — Phase 3 gaps: taEffectiveStage, DB date, MTD, filled_date — commit 11ba4d9
- **taEffectiveStage rank guard:** Fixed overly broad rule — was returning `'Interview'` for ANY `interview_reached=true` candidate, including those already at Appointment Letter. New condition: `interview_reached && rank > 0 && rank < (TA_STAGE_RANK['Pre Offer Verification']||5)`. Rejected/blacklisted stages pass through unchanged (rank=0 effectively). Candidates at Offer Letter, Appointment Letter, etc. with interview dates now show their actual stage.
- **CAND_DATE from DB:** `loadTACandMap` now also selects `data_as_of` and computes `MAX → _taCandDbDate` (global). `renderTAPipeline` builds `_candDbFmt` from `_taCandDbDate` and uses it as `CAND_DATE` (falls back to `TA_SUMMARY.cand_as_of`). Amber warning appended if `cand_as_of_iso` (new field set by processCandidate, persisted to data_cache) is >1 day newer than DB — indicates a write that succeeded in the UI but didn't reach DB.
- **Employee_master refresh date:** `_emRefDate` global (locale string) set in `loadEMJoinerCounts` via `MAX(uploaded_at)` query (fetched once). Filled→Joined card as-of note now shows `_emRefDate||TAT_DATE` so the date reflects when employee_master was last uploaded, not the TAT file.
- **_emMTDJoiners emp_code guard:** MTD query now includes `.not('emp_code','is',null)` — explicit guard matching the confirmed definition (emp_code present AND doj in current month AND doj <= today). Functionally a no-op since employee_master upsert filters empty emp_codes, but satisfies spec intent.
- **Onboarding Tracker Joined MTD:** `joinedMTD` now uses `(_emMTDJoiners!==null) ? _emMTDJoiners : _joinedMTDFallback` where fallback is the prospective_joiners count. Both TA Pipeline card and Onboarding Tracker header now show the same authoritative figure.
- **processReqTAT filled_date:** Added `_tatParseIso(v)` helper (handles DD MMM YYYY, YYYY-MM-DD, ISO). Populates `filled_date` from 'ECode Creation Date' column for Closed rows; included in `_rtBatch` upsert payload. `filled_date` column already existed in req_tracker (migration pre-existed).
- **sbBoot _filled boot query:** New query after req_tracker load — counts req_tracker rows with `status='Closed'` AND `filled_date >= first-of-month` AND `filled_date <= today`. Sets `TA_SUMMARY.mar_fulfilled`. Filled→Joined card `_filled` now live from DB at boot. Shows 0 until next TAT upload backfills dates (known, acceptable).
- **Known Debt:** `mar_*` naming fossil — `mar_fulfilled`, `mar_joined`, `mar_closed` still named after March 2026 (the month the hardcoded constants were seeded). Rename to `mtd_*` in a future refactor. | Pre-Phase-3 bulk upload may have left pj historical noise (old Appointment Letter candidates from before rolling-window semantics were understood) — a one-time manual cleanup may be needed if Onboarding Tracker shows stale resolved rows.

### 10 Jul 2026 — Candidate parser overhaul (Phase 3) — commit 5f82c8c
- **Task 1 — Upsert by application_id:** `application_id: _tcVal(r,'ApplicationID')` added to mapped row. Rows with empty ApplicationID skipped (`_tcSkipped` count). `delete().neq('id',...)` removed. One-time legacy purge: `delete().is('application_id', null)` removes pre-upsert-era rows. Chunked `.insert()` replaced with `.upsert(chunk, {onConflict:'application_id'})` — upload is now idempotent; rolling-window uploads no longer destroy history.
- **Task 2 — Interview derivation + taEffectiveStage:** 7 round-date columns scanned per candidate row: `Client Round 3 Date`, `HR interview Date`, `HR Round Date`, `T1 Interview Date`, `T2 Interview Date`, `Techincal Round 1 Date` (ZingHR typo, preserved exactly), `Technical Round 2 Date`. Derived fields written to DB: `interview_reached` (bool), `first_interview_date`, `last_interview_date`. Global `TA_STAGE_RANK` map defined (Screening→1 … Appointment Letter→10). Global `taEffectiveStage(r)` returns `'Interview'` if `r.interview_reached`, else `r.application_stage`. `loadTACandMap` boot query now selects `interview_reached`; stage ranking routes through `taEffectiveStage`. TA Pipeline stage column and RMG stage column display effective stage.
- **Task 3 — Integrity guard:** Red persistent banner injected into `#uploadStatus` if 0 rows upserted or chunk errored (includes RLS failure message). Green `showToast` on success: `"{n} candidate rows written · {interviewCnt} at Interview · {skipped} skipped (no ApplicationID)"`. Both `_tcInserted===0` and `_tcErrMsg` paths covered.
- **Task 4 — One MTD source:** `_joined` on Filled→Joined card now `(_emMTDJoiners!==null)?_emMTDJoiners:(S.mar_joined||0)`. `_emMTDJoiners` verified correct: employee_master rows with `emp_status='NewJoinee'`, `doj >= first of month`, `doj <= today`.
- **Task 5 — pj rebuild guard:** `_syncRows` filter scoped to: stage in `_pjSyncStages` AND (`!employee_code OR (final_doj >= first day of prev month)`). Prevents full history re-rebuild from old uploaded data blowing away manually-set statuses on completed rows.
- **Line delta:** +74 insertions / -34 deletions (net +40 lines).
- **Known Debt:** DB needs `application_id` unique constraint on `ta_candidates` for upsert to be truly safe — currently conflicts silently succeed but duplicate-ID rows could exist from pre-upsert era. `filled_date` backfill: TAT re-upload needed to populate the column on existing Closed rows before `_filled` MTD count shows real data.

### 10 Jul 2026 — TA Pipeline: By Customer ageing matrix, customer filter, name normaliser (Phase 2)
- **New helpers:** `_CUST_ALIAS` (typo/alias map: COMAPANY→COMPANY, CMSIT→CMS IT), `_normCustomer(c)` (trim + collapse whitespace + uppercase + alias lookup), `_taExpandedCust` (row-expand state), `_taCustToggle(idx)` (global toggle fn), `_renderTACustMatrix(regionData, allScopeData)` (full matrix renderer).
- **Customer filter:** `_taCustomerFilter='All'` added to filter globals. Customer `<select>` dropdown after Recruiter in filter bar, keyed on `_normCustomer`. All filters (KPI cards, aging strip, grid) read the same `data` base — no e970c51 regression.
- **By Req / By Customer toggle:** `_taView` now takes values 'req' (was 'pipeline') | 'customer' | 'recruiter-map'. Sub-toggle rendered above filter row. Top-level Pipeline tab button active for both 'req' and 'customer' states.
- **Ageing matrix:** 10-col grid (Account, Total, <7, 7-15, 16-30, 31-45, 46-60, 61-90, 90+, Regions). Tier group headers Platinum → Gold → Silver → Regular, sorted by total desc within group. 61-90 cells amber-tinted, 90+ red-tinted. Zero cells render '·'. Region filter: Total column shows "X /Y" (region count / all-scope count); regions chip shows ALL regions with open reqs regardless of filter. Row click expands customer's req rows (sorted by tat desc, showing req_id, designation, tat, region, recruiter + stage). Phantom toggle recounts matrix cells. Matrix footer shows grand total.
- **`_taView='pipeline'` → `'req'`:** global init changed; all recruiter-map toggle buttons updated to use 'req' key with 'req'||'customer' active state.
- **Line delta:** +113 lines (new-function additions — `_renderTACustMatrix` ~75 lines + helpers).
- **Known Debt:** customer aliases hardcoded in `_CUST_ALIAS` — proper fix is alias column on `customer_accounts`; req with title leaked into customer field (RELIGAREBROKING…-Noida req) pending enrich fix.

### 10 Jul 2026 — RMG parity: effective status, Not-in-ZingHR badge, parity strip
- **Context:** `sync_positions_from_req_tracker()` RPC extended (migration `sync_positions_insert_and_orphan_steps_20260709`). After every TAT upload it now: (A) refreshes req_status/req_age_days on matched rows, (B) INSERTs any Open req missing from account_positions_log (raised_by='tat_sync', manual fields null), (C) marks orphan rows (zinghr_req_id absent from req_tracker) with req_status='Not in ZingHR'. Result: account_positions_log has 173 rows req_status='Open', 9 req_status='Not in ZingHR', 323 req_status='Closed' (manual status still 'Open' on many — two-status model locked, Close action not yet built).
- **Problem fixed:** RMG Workspace was reading manual `status` only → showed 391 open instead of 173.
- **Fix (commit this session):** Added `_rmgEffStatus(r)` helper (priority: manual Closed > Not in ZingHR > req_status Closed > null/empty req_status → Not in ZingHR > Open). All status filtering and display now use effective status. Four-pill status filter: All / Open / Closed / Not in ZingHR (violet). Not-in-ZingHR rows get violet left border. Rows auto-closed by ZingHR (req_status=Closed but manual status=Open) show Closed pill + grey "auto" suffix — manual `status` field NOT touched. Parity strip added above KPI cards: "RMG Open: 173 · TA Pipeline Open: 173 ✓" (green tick when match, red delta + message when not).
- **Open count corrected:** 391 (reading manual status) → 173 (effective status).
- **Known Debt:** Close action still pending — manual `status` flip not yet built. Until then, RMG cannot manually override a ZingHR-auto-closed req. The 323 rows with req_status=Closed/manual status=Open show as Closed (correct — ZingHR is authoritative for auto-close direction).

### 8 Jul 2026 — TA scoreboard vs grid reconciliation (single-owner attribution)
- **Reported gap:** Ajith's req grid (top table) showed ~16 open positions but his recruiter scoreboard read 13 active / 11 critical. Confirmed systemic — nearly every recruiter was off in both directions (e.g. Caral card 23 vs live 3, Nikita 43 vs 18, Salma 25 vs 45), plus phantom rows (bloated `Unassigned` 67, stray `Meenakshi`).
- **Root cause (two layers):** (1) The scoreboard rendered from the frozen `data_cache.ta_scorecard` snapshot (written once at the last TAT upload, 02:56), while the grid reads live `data_cache.ta_reqs`. Reqs re-owned after the snapshot showed in the grid but not the scoreboard. (2) The stored scoreboard key had a **double space** (`"Ajith  Inguva"`) — recruiter fields carry inconsistent whitespace/leading-trailing spaces, so exact-string grouping splintered a person across variants while the grid's loose `indexOf` did not.
- **Fix (commit 6e8e142):** Added one global `_taOwner(r)` (primary_recruiter preferred → else first non-HRBP on `ta_recruiter`, with paren-strip + whitespace-collapse + trim) and `_taBuildRecCounts(rows)`. `renderTAPipeline` now builds `window._taLiveCounts` from `data` **before** the recruiter filter, and three consumers all use `_taOwner`: the grid recruiter filter, the scoreboard rows (active/critical/plat_gold/backlog_age overridden from live counts; union of TA_SCORECARD names + live owners; sorted by active desc), and the personal-KPI `_recData`. `data_cache.ta_scorecard` is no longer trusted for active/critical/backlog — only for pipeline/offers/joined/fill (candidate-sourced). Result: clicking any recruiter yields a table whose row count equals their scoreboard number.
- **Post-fix live distribution (active / crit>45d, 180 open total):** Salma 45/44 · Kumari 25/15 · Nilam 18/15 · Nikita 18/12 · Juee 17/14 · Ajith 16/15 · Mousumi 10/7 · Dhananjay 8/2 · Neha 6/5 · Caral 3/2 · Pijush 1/0 · Unassigned 13/0.
- **Data flags (not display bugs):** Salma has 45 reqs with `primary_recruiter='Salma Saifi'` — real per Neha's mapping; confirm the concentration is intentional. `Unassigned` 13 = reqs with empty primary_recruiter and only HRBP names on the ZingHR requisition — need a recruiter assigned.
- **Ops note:** commit had to be run from the local Claude Code terminal, not the Cowork bash sandbox — the sandbox served a truncated copy of the 1.2 MB index.html (git saw stale metadata / zero changes). See Cowork sync note below. Edits themselves were fine via the Read/Edit file tools; only git-from-sandbox was unsafe.

### 7 Jul 2026 — TA Pipeline recruiter filter + scorecard, Hiring Report fixes, Suman login (session 1)
- **_lookupTier global scope fix (4ff6795):** `_tierMap`, `_tierList`, `_lookupTier` were defined inside a nested async function — `_loadHRCycleData` called them before that scope ran → ReferenceError → blank Hiring Report. Promoted all three to global scope. Added `customer_accounts` fetch at top of `_loadHRCycleData` to populate tier maps before any lookup.
- **closedReqs query fix (53d7993):** `req_tracker` closed reqs filter used chained `.not().neq()` (invalid) — replaced with `.gt('primary_recruiter','')`.
- **Hiring Report recruiter closures (a4a2f05):** `_loadHRCycleData` now fetches closed reqs within cycle window from `req_tracker` (primary_recruiter, tat). `renderHiringReport` shows recruiter performance table with Closed, Avg TAT, and % of cycle total.
- **TA Pipeline recruiter filter system (6137e7d, cdb597e, fb82687):** `_taRecruiterFilter` global (first-name string or 'All'). Clicking a scorecard row sets filter. Pill with clear button shown in filter bar. Personal KPI bar renders when filter active (Open Reqs, Critical, Plat/Gold, Closed MTD, Avg TAT, Dropouts). HRBP exclusion (`suhail,somasri,ayush,abhishek,priya paul,shambhavi`) applied to both scorecard rows and dropdown. Auto-scope: TA recruiter logins default to their own name.
- **loadTAClosedMTD (05c91ea):** New async function fetches `req_tracker` (status=Closed, current month, primary_recruiter not empty), builds `_taClosedMTDMap = {firstName: count, firstName_tat: avgTAT}`. Triggered at boot after ta_reqs loads (setTimeout 400ms). Re-renders TA Pipeline tab if active.
- **Recruiter dropdown (05c91ea):** `<select>` in TA Pipeline filter bar listing first names from TA_SCORECARD (HRBP-excluded). Highlights purple when filter active.
- **Scorecard 9-col layout (05c91ea):** Added `Closed MTD` column sourced from `_taClosedMTDMap`. Header: `Recruiter,Active,Crit,Backlog,P/G,Pipeline,Closed MTD,Avg TAT,Conv%`.
- **Scorecard active count source (0b57407):** Build loop now prefers `r.primary_recruiter` (Neha's manual mapping from req_tracker) over raw ZingHR `ta_recruiter`. Fallback splits `ta_recruiter` on comma, skips HRBPs, takes first. `primary_recruiter` is included in the TA_ACTIVE_REQS select (line ~11573).
- **Combined Filled→Joined KPI card (0b57407):** Replaced two separate "Roles Filled" + "People Joined" KPI cards with one combined card showing `filled → joined` side-by-side with "N pending joining" (amber) or "all joined ✓" (green). Grid now `repeat(4,1fr) 1.2fr`.
- **Suman Sharma login fix:** Account created via SQL → three GoTrue columns were NULL instead of `''`: `confirmation_token`, `email_change`. GoTrue crashed with `sql: Scan error on column index N, name "X": converting NULL to string is unsupported` → HTTP 500. Fixed via `UPDATE auth.users SET col = COALESCE(col,'')`. Password reset to `CMS2026` (no special chars). **Rule: any user created directly via SQL INSERT must have these columns set to `''`: `confirmation_token`, `recovery_token`, `email_change_token_new`, `email_change_token_current`, `email_change`, `phone`.**
- **Commits this session:** 4ff6795, 53d7993, a4a2f05, 7846abc, 6137e7d, cdb597e, fb82687, 05c91ea, ff9de01, 0b57407

### 6 Jul 2026 — Hiring Report tab, sidebar gating (session 3)
- **Hiring Report tab (7dc1cbe):** New tab `📈 Hiring Report` added after TA Pipeline in sidebar. Tab id: `hiringreport`. Functions: `loadHiringReport`, `_loadHRCycleData`, `renderHiringReport`, `saveHiringCycle`. Data sources: `hiring_cycles` table (named cycle periods), `employee_master` (DOJ-ranged joiners), `req_tracker` (open reqs + critical aging >100d), `prospective_joiners` (next cycle pipeline — Offer Accepted, DOJ after cycle end), `_taCandMap` (pipe/stage per req). Renders: KPI strip (total joined + region cards with realization %), account-wise closures table, critical aging grid, next cycle pipeline. New cycle form (admin/TA only). Access: admin, ta (function_type), executive — all others blocked.
- **DOJ date format (8b66bfa):** Account closures DOJ Range column now shows `1 Jul → 14 Jul` instead of raw ISO strings. IIFE pattern inline in acctRows map.
- **Sidebar gating (8b66bfa):** `snav-hiringreport` hidden for all HRBP roles (inserted in `showSidebarAfterLogin` HRBP block, line ~11254). RMG already excluded — `Hiring Report` absent from `_rmgAllowed` whitelist.
- **Pending:** `hiring_cycles` table migration needed in Supabase before tab renders data (will show "No cycle data available" until created).
- **Commits this session:** 7dc1cbe, 8b66bfa

### 6 Jul 2026 — Fuzzy+DOJ reconcile RPC, RMG Pipe+Stage columns, SQL cleanup (session 2)
- **Fuzzy+DOJ reconcile (b575f9c):** `processEmployeeMaster` now calls `reconcile_prospective_joiners()` RPC after the existing name-match auto-reconcile. Catches middle-name variants, extra spaces, and DOJ-bounded matches missed by the exact-match pass. Returns count of additional rows marked Joined.
- **RMG Pipe+Stage columns (fee8bbc):** `_renderRMGGrid` grid expanded from 11 to 13 columns. New cols: `Pipe` (active candidate count from `_taCandMap` keyed by `zinghr_req_id`) and `Stage` (abbreviated stage label with rank-based colour). IIFE pattern keeps the row builder self-contained. Header + data row `grid-template-columns` both updated.
- **Postgres function:** `reconcile_prospective_joiners()` created (migration: `reconcile_prospective_joiners_fuzzy_doj`). Uses pg_trgm similarity + DOJ window to fuzzy-match prospective_joiners against employee_master. Returns integer count of rows updated to Joined.
- **Manual SQL cleanup:** 13 prospective_joiners rows manually marked Joined via cross-reference with employee_master (direct SQL UPDATE). 4 duplicate rows deleted.
- **Commits this session:** b575f9c, fee8bbc

### 6 Jul 2026 — Long Absenteeism filter, Onboarding Tracker, RMG enhancements, login fixes (session 1)
- **Long Absenteeism (3e949dd):** `loadAbsentCases` now cross-references `employee_master` after fetch. Keeps only employees present in master (Existing/NewJoinee). Resigned/FnF Locked/FnF In Process employees are absent from the master and get filtered. Reduced from ~420 to ~251 actionable rows.
- **RMG Workspace (f008615):** Added "Added By" + "Added On" columns to grid; amber left-border + amber cell for rows added in last 48h. `submitRMGNewPosition` now populates `raised_by` + `raised_by_id`. Existing seed rows renamed "Elah Import" via SQL UPDATE.
- **Onboarding Tracker rewrite (ebdc980):** Renamed from "Prospective Joiners". Full `renderJoinersTab` rewrite — card pipeline by week (This Week / Next Week / 2-4 Weeks), HRBP auto-scope to region, overdue action buttons (Joined/No Show/Drop), Joined MTD in header, admin dropout rate by recruiter. RMG access added. `updatePJStatus` alias added.
- **ta_candidates → prospective_joiners sync (1e613bf):** On every candidate upload, syncs Appointment Letter + Pre Joining stage candidates to `prospective_joiners`. Tier enriched from `customer_accounts`. Manually-resolved rows (Joined/No Show/Dropped Out) protected from overwrite.
- **employee_master + mar_joined (8c1839e):** `processEmployeeMaster` upserts active employees to `employee_master` (conflict on emp_code). `mar_joined` KPI now sources from NewJoinee DOJ in Super Employee Master.
- **TA Pipeline (8ebea20):** Grid slice raised from 150 → 250.
- **Login fixes:** Sonal Rale + Pruthvi SS — `email_confirmed_at` was NULL; fixed via SQL UPDATE + temp password `CMS@2026!`. Neha's 404 was wrong URL (trailing `/and`). Prapti Patel — wrong password, reset needed via Supabase dashboard. Nehal + Chander — pending check.
- **Commits this session:** 8ebea20, 7d047db, 8c1839e, 732f602, 5335867, d6129e9, ebdc980, 1e613bf, f008615, 3e949dd

### 18 Jun 2026 — RMG status model + filter-responsive cards
- Session 18 Jun 2026 — RMG status model + filter-responsive cards. Commits: 312fb7a req_tracker
  full mirror (all 774 TAT rows retained, was truncating to 417; zinghr_status raw column added;
  status derived Open-only-if-zinghr_status='Approved', else Closed; enrich Step 3 closes balance≤0;
  verified 774 rows ≈189 Open / ≈215 stale), fccee3d TAT enrich (Supabase RPC
  sync_positions_from_req_tracker called after enrich in processReqTAT, writes ONLY
  req_status/req_age_days/audit cols — manual fields untouched; auto-fires every TAT upload; Add
  default + status pills → Open/Closed; verified via manual RPC run, req_status=rt.status and
  req_age=rt.tat exact), e970c51 filter-responsive RMG cards (cards now read cardBase = region+status
  filtered subset; flag pills filter grid rows only, cards keep RED/AMBER/GREEN breakdown; root cause
  was wrong-source bug — counts read full _rmgData not filtered data). RMG scope CONFIRMED: holds ALL
  reqs incl non-delivery/corp; only a few confidential searches excluded (confidential flag pending).
  Neha constraint: true open ≤100 vs system ~190 → ~90 phantom-open beyond the 217 stale; handle by
  surfacing phantom-signal flags (aged-no-movement / no recruiter / no candidate / balance-vs-joined),
  not by massaging the count; balance unreliable as real-open signal; candidate pipeline activity is the
  key real-vs-phantom discriminator. NEXT: Candidate enrich (pipeline stage into req_tracker +
  account_positions_log via req_id; serves TA Pipeline + RMG grid + phantom discriminator; start with
  diagnostic to read real stage vocabulary + req linkage before building the ladder). Then Close action
  (status = manual RMG override + reason), eSep surfacing, open-position-health view (phantom flags +
  RMG-vs-TA parity strip + confidential flag). Confirm what Neha's 100 counts (all active open vs
  TA-sourced active). Vercel MCP 403 on team scope recurring — needs connector re-auth.

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

### Session Discipline — End-of-Phase Gate
Every phase ends with:
1. `git push origin main`
2. `curl -sL "https://raw.githubusercontent.com/alexaugustine1-rgb/cms-hr-dashboard/main/index.html?cb=$(date +%s)" | grep -c "MARKER"` — must return ≥ expected count
3. A local commit is NOT a shipped phase — the push + CDN marker verification is the gate.
If the push rejects on auth, STOP and report — do not mark the phase complete until the marker is confirmed on origin/main. Vercel auto-deploys on push; the live site is only updated after a successful push.

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

## Current File State — 19 Jun 2026

SESSION 19 Jun 2026 — TA Pipeline & RMG recruitment build complete

COMMITS THIS SESSION (chronological):
- 58945e4: RMG sidebar scoping (7 tabs) + Long Absenteeism read-only gate
- 000ec2e: processCandidate → ta_candidates from candidate transaction report
- 23bd3ec: TA Pipeline 11-col grid (Pipe/Stage/ActvD) + phantom filter
- 8ebea20: Grid slice cap 150→250
- 7d047db: Prospective Joiners — joining strip + dropout rate
- 8c1839e: processEmployeeMaster → employee_master + auto-reconcile prospective_joiners
- 5335867: Dropout rate fix (_pjData) + TA Commitments retired
- 732f602: mar_joined KPI from NewJoinee DOJ in Super Employee Master

RMG GO-LIVE (19 Jun):
- 4 auth accounts: sonal.rale, pruthvi.ss, nehal.shaikh, chander1.mohan @cmsitservices.com
- user_profiles seeded via hardcoded UUID INSERT (JOIN-based SQL failed)
- All 4 confirmed in user_profiles with role=rmg

ADDITIONAL COMMITS — 19 Jun 2026 (session continued):
- ebdc980: Onboarding Tracker full redesign — card pipeline by week (This Week/Next Week/2-4 Weeks), HRBP auto-scope by region, overdue action buttons (Joined/No Show/Drop), Joined MTD header, RMG access added, updatePJStatus alias added, dropout rate admin-only
- 5335867: Dropout rate fix (_pjData not data) + TA Commitments retired from sidebar
- 732f602: mar_joined KPI sourced from NewJoinee DOJ in Super Employee Master (not ECode date)
- 8c1839e: processEmployeeMaster — persist to employee_master + auto-reconcile prospective_joiners by name match (24 auto-cleared on first run)
- d6129e9: Prospective Joiners ±60d DOJ window default (superseded by ebdc980 rewrite)
- 1e613bf: sync ta_candidates → prospective_joiners on every candidate transaction upload — Appointment Letter + Pre Joining stages, tier enriched from customer_accounts, manually-resolved rows protected, 39 rows synced on first run (28 Joined, 11 Offer Accepted)

ONBOARDING TRACKER — ARCHITECTURE (confirmed 19 Jun):
- Sidebar renamed: Prospective Joiners → Onboarding Tracker
- Data source: ta_candidates (candidate transaction upload) → syncs to prospective_joiners on every upload
- Upcoming joiners: Offer Accepted rows with future DOJ (no emp_code in ta_candidates)
- Confirmed joiners: Joined rows auto-set when emp_code populated in ta_candidates
- Auto-reconcile: Super Employee Master upload matches by name → marks Joined in prospective_joiners
- Tier enrichment: customer_accounts.tier joined by name during sync — correct Platinum/Gold/Silver/Regular
- HRBP auto-scope: role=hrbp users auto-filtered to their region on load
- 14 legacy overdue candidates (Mar-Apr DOJ) unresolvable by automation — need manual recruiter update
- Onboarding Tracker accessible to: admin, executive, hrops, hrbp, ta, rmg

PROSPECTIVE JOINERS DATA MODEL (corrected understanding):
- prospective_joiners is NOT a manual upload table anymore
- It is now a derived table — rebuilt from ta_candidates on every candidate transaction upload
- Manually resolved statuses (Joined/No Show/Dropped Out) are protected from overwrite
- employee_master (Super Emp Master) is the authoritative source for confirmed joiners and MTD count

KPI SOURCE MAP (updated):
- People Joined MTD → employee_master NewJoinee + DOJ current month (Super Emp Master upload)
- Upcoming Joiners → prospective_joiners Offer Accepted + future DOJ (candidate transaction upload)
- Pipe/Stage/ActvD per req → ta_candidates (candidate transaction upload)
- Open Reqs / Aging → data_cache.ta_reqs (TAT upload + enrich RPC)

PENDING NEXT SESSION:
- Designation extraction from req_title in prospective_joiners sync (currently raw req_title string)
- eSep surfacing
- Close action for RMG manual status override
- ~~RMG Workspace candidate visibility (Pipe/Stage per req in RMG grid)~~ ✅ done (fee8bbc, 6 Jul)
- ~~Hiring Report tab (cycle-based TA closure report)~~ ✅ done (7dc1cbe, 6 Jul)
- **hiring_cycles Supabase migration** — create table before Hiring Report tab goes live (cols: id uuid PK, name text, label text, cycle_from date, cycle_to date, notes text, created_by text, created_at timestamptz). Add RLS select policy for authenticated users; insert/update for admin+ta.
- KNOWLEDGE.md re-upload to Claude.ai project (current file is stale)

### Recent Commits (newest first)
- 7846abc (6 Jul): Hiring Report: remove notes from header, DOJ date format, hide from HRBP sidebar (cleanup — 2 of 3 changes were already live)
- 8b66bfa (6 Jul): Hiring Report: DOJ date format + hide from HRBP sidebar
- 7dc1cbe (6 Jul): Hiring Report tab: named cycles, region realization %, account closures, critical aging, next pipeline
- fee8bbc (6 Jul): RMG Workspace: Pipe + Stage columns from _taCandMap (13-col grid)
- b575f9c (6 Jul): processEmployeeMaster: fuzzy+DOJ reconcile via reconcile_prospective_joiners() RPC
- 3e949dd (6 Jul): Long Absenteeism: filter against employee_master; RMG added-by/added-on columns
- 1e613bf (19 Jun): sync ta_candidates → prospective_joiners: Appointment Letter + Pre Joining stages with tier enrichment
- ebdc980 (19 Jun): Onboarding Tracker: full redesign — pipeline by week, HRBP auto-scope, action needed, joined MTD
- d6129e9 (19 Jun): Prospective Joiners: default ±60d DOJ window filter + toggle to show all
- 732f602 (19 Jun): fix: mar_joined KPI sourced from NewJoinee DOJ in Super Employee Master
- 5335867 (19 Jun): fix dropout rate use full _pjData; retire TA Commitments sidebar
- 8c1839e (19 Jun): processEmployeeMaster: persist to employee_master + auto-reconcile prospective_joiners
- 7d047db (19 Jun): Prospective Joiners: joining this week/next week strip + dropout rate by recruiter
- 8ebea20 (19 Jun): TA Pipeline: raise req grid slice cap 150→250 to show all open reqs
- 23bd3ec (19 Jun): TA Pipeline 11-col grid (Pipe/Stage/ActvD) + phantom filter
- 000ec2e (19 Jun): processCandidate → ta_candidates from candidate transaction report
- 58945e4 (19 Jun): RMG sidebar scoping (7 tabs) + Long Absenteeism read-only gate
- ae3fc58 (18 Jun): KNOWLEDGE: 18 Jun — RMG status model, TAT enrich, filter-responsive cards, phantom-open principle
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
  Only touches those 4 cols (req_status, req_age_days, updated_at, updated_by). NEVER writes
  status, notes, demand_class, backfill_* or elah_demand_id — those are manual RMG fields.
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
3. **Candidate enrich** — pipeline stage into req_tracker + account_positions_log via req_id; serves TA Pipeline + RMG grid + phantom discriminator. Start with diagnostic: read real stage vocabulary + req linkage before building the ladder.
4. **Designation fix** — contract_workforce parser + eSep enrichment join (prompt written, not committed).
5. **Phase 6A — Resignation backfill trigger** — eSep upload flags resignations needing backfill; RMG review queue (Raise Req / Deploy from Bench / No Replacement). Designation enrichment: join contract_workforce by emp_code at eSep parse time. Design complete.
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
- hiring_cycles (migration PENDING — needed for Hiring Report tab; cols: id, name, label, cycle_from, cycle_to, notes, created_by, created_at)

### account_positions_log Schema (17 Jun 2026, updated 10 Jul 2026)
505 rows as of 09 Jul 2026 (173 Open, 9 Not in ZingHR, 323 Closed). Migration: rmg_positions_log_columns_and_rls (20260617) + sync_positions_insert_and_orphan_steps_20260709.
req_status and req_age_days auto-synced via sync_positions_from_req_tracker() RPC on every TAT upload. RPC also INSERTs missing Open reqs (raised_by='tat_sync') and marks orphans req_status='Not in ZingHR'.
RLS: 4 policies (aplog_select/insert/update/delete).
Key columns: zinghr_req_id, account_name, region, position_type, designation,
elah_demand_id (text — Elah demand line, entered by RMG), demand_region,
emp_category (FTE/ATE/Retainer), backfill_emp_code, backfill_emp_name,
req_status (Open/Closed/Not in ZingHR — auto-synced from req_tracker, do not hand-edit),
status (manual RMG override — Close action pending, not yet built),
req_age_days, in_elah (bool), days_vacant (GENERATED STORED), raised_by, updated_by, updated_at, created_at.
Two-status model: req_status=auto (ZingHR), status=manual (RMG). _rmgEffStatus() in UI resolves both.

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
- sync_positions_from_req_tracker() — extended 09 Jul 2026 (migration: sync_positions_insert_and_orphan_steps_20260709).
  Step A: UPDATE req_status/req_age_days on matched rows. Step B: INSERT Open reqs missing from account_positions_log (raised_by='tat_sync'). Step C: mark orphan rows req_status='Not in ZingHR'. Called in processReqTAT after enrich_req_tracker_from_accounts.
- reconcile_prospective_joiners() → integer (migration: reconcile_prospective_joiners_fuzzy_doj, 6 Jul 2026).
  Fuzzy-matches prospective_joiners against employee_master using pg_trgm similarity + DOJ window.
  Updates matched rows to status='Joined'. Returns count of rows updated.
  Called in processEmployeeMaster after the exact-name auto-reconcile pass.

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
