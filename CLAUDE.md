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
- File: index.html | Lines: 10,890 | Latest commit: 98d6fad (+ res.error fix pending)
- Phases complete: 1, 2, 3, 6B-DBT, Trainer Activity (L&D tab)
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
- switchTab() LIVE: line 4213 (line 3564 is dead code)
- computeHRBPScore(): search "function computeHRBPScore"
- CUSTOMER_LIST: global, populated at boot from customer_accounts
- TA_ACTIVE_REQS: global, populated by processReqTAT
- _absentCases: global, populated by loadAbsentCases()
- WI: hardcoded const line ~845, overridden by Supabase at boot

## Architecture Notes
- Absent resolution chip: reads window._absResolvedCount and
  window._absTotalCount — set in sbBoot() separately.
  Do NOT modify loadAbsentCases() query.
- Duplicate switchTab: line 3564 is dead code, 4213 is live.

## Supabase
- Project ID: mzyrcrkwgbqgwajkjdnp
- pg_trgm extension: enabled
- execute_sql for DML | apply_migration for DDL
