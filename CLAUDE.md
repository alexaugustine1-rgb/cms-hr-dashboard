# CMS HR Ops Command Centre — Claude Code Rules

## Standing Rules — Non-Negotiable
- str_replace ONLY. Never rewrite the full file.
- node --check after every single edit. Extract scripts to temp .js first. No output = clean pass.
- If node --check errors: print 3 lines above and 3 below before fixing.
- Single HTML file. No splitting. No build toolchain. No npm.
- All persistent data in Supabase. No hardcoded arrays with real data.
- Git commits: email alexaugustine1@gmail.com
- After every phase: explicit PASS/FAIL checklist before stopping.

## Known Crash Patterns
Any of these crashes login with "ReferenceError: sbLogin is not defined":
- Orphaned em-dash in string concatenation
- await inside a non-async function
- Block-scoped function declarations inside try blocks
- const or let declared inside try blocks
node --check catches all of these.

## Shared Tables — FLAG Before Touching
Shared with OPS360. Additive only. No drops or type changes.
- workforce_intel, absent_cases, resignation_tracker
- planned_leaves, employee_account_mapping

## Current File State
- File: index.html | Lines: 11,250 | Latest commit: c67b8b8
- Phases complete: 1, 2, 3, 6B-DBT, L&D Tab Redesign (full)
- Next: Phase 4 — Account Health tab (read KNOWLEDGE.md for spec)

## Key Functions
- sbBoot(): search "async function sbBoot"
- renderOverview(): search "function renderOverview"
- renderAccountHealth(): search "function renderAccountHealth"
- nameMatch() LOCAL: inside renderAccountHealth() line ~6291
- normaliseCustomer(): search "function normaliseCustomer"
- processReqTAT(): search "async function processReqTAT"
- processCandidate(): search "async function processCandidate"
- handleUploadFile(): search "function handleUploadFile"
- switchTab() LIVE: search "function switchTab" — second occurrence is live
- computeHRBPScore(): search "function computeHRBPScore"
- loadLDTab(): search "async function loadLDTab" — drives full L&D tab
- _loadLDSessions(): search "async function _loadLDSessions"
- _loadLDPipeline(): search "async function _loadLDPipeline"
- CUSTOMER_LIST: global, populated at boot from customer_accounts
- TA_ACTIVE_REQS: global, populated by processReqTAT
- _absentCases: global, populated by loadAbsentCases()
- WI: hardcoded const line ~845, overridden by Supabase at boot

## Architecture Notes
- Absent resolution chip: reads window._absResolvedCount and
  window._absTotalCount — set in sbBoot() separately.
  Do NOT modify loadAbsentCases() query.
- Duplicate switchTab: BOTH now call loadLDTab. Second definition is live.
- L&D tab: renderLD() returns static shell only. loadLDTab() loads all
  data async (training_sessions + training_pipeline). Fires on render()
  and switchTab('ld'). Month selector id = ldMonthSel.
- HRBP form has TWO training sections:
  1. "Training Conducted This Week" → f_hrbp_training_entries → training_sessions
  2. "Training Programs Planned / Conducted" → f_hrbp_pipeline → training_pipeline
- LD form: f_ld_pipeline (structured grid) replaces old f_ld_upcoming free-text.
  f_ld_upcoming hidden input kept for backward compat.

## Supabase
- Project ID: mzyrcrkwgbqgwajkjdnp
- pg_trgm extension: enabled
- execute_sql for DML | apply_migration for DDL

## OVERWRITE PREVENTION — MANDATORY

Features get silently lost when sessions rewrite the full file.
These rules are non-negotiable:

1. str_replace ONLY. Never write the full file. If a task
   feels like it needs a full rewrite, STOP and tell Alex
   instead of proceeding.

2. Before every session: note the current line count of
   index.html. If the output file differs by more than 50
   lines from the input, STOP — do not commit. A change of
   50+ lines likely means a full rewrite happened silently.

3. Features known to have been lost in past rewrites and
   must be preserved:
   - HRBP action dropdown on OD/WFH tab (wfh_od_actions)
   - _wfhOdActions cache + saveODAction() function
   - loadWFHODActions() function
   - multi-instance badge in renderOD rows
   These must be present in index.html at every commit.

4. Before committing, grep for these function names:
   grep -n "saveODAction\|loadWFHODActions\|_wfhOdActions\|multiBadge" index.html
   If any return 0 results, DO NOT commit.
