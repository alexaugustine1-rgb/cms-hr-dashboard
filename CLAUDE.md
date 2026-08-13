# CMS HR Ops Command Centre — Claude Code Rules

> **Read `KNOWLEDGE.md` first.** It is the single source of truth for this project
> (architecture, schema, parsers, users, phases, known debt, history). This file
> holds only the non-negotiable guardrails that must load every session. Keep it
> short — put project knowledge in KNOWLEDGE.md, not here.

## Non-Negotiable Rules
- **str_replace ONLY.** Never rewrite the full index.html. If a task seems to need a
  full rewrite, STOP and tell Alex.
- **node --check after EVERY edit.** Extract scripts to a temp .js first (Node v24
  won't check .html directly). No output = clean pass. On error, print 3 lines above
  and below before fixing.
- Single HTML file. No splitting, no build toolchain, no npm/webpack/React.
- All persistent data in Supabase. No hardcoded arrays with real data.
- Git commits authored as alexaugustine1@gmail.com.
- After every phase: explicit PASS/FAIL checklist before stopping.

## Overwrite Prevention (silent full-rewrites have lost features before)
- Note index.html line count before the session. If output differs by >~50 lines
  from input, STOP — do not commit (likely a silent rewrite). New-function additions
  over 50 lines are OK but must be documented in the commit report.
- Before committing, grep — if any return 0 results, DO NOT commit:
  `grep -n "saveODAction\|loadWFHODActions\|_wfhOdActions\|multiBadge" index.html`

## Known Crash Patterns (all caught by node --check)
Crashes login with "ReferenceError: sbLogin is not defined":
orphaned em-dash in string concatenation · await in a non-async function ·
block-scoped function declarations inside try blocks · const/let inside try blocks.

## Cowork note
index.html lives in Dropbox; the bash sandbox may show a stale/truncated copy. The
Read/Edit file tools see the true file. To node --check in Cowork, extract edited
regions via Read into a temp .js in the outputs dir and check that.

## End of session
Update `KNOWLEDGE.md` per its update protocol (schema/phase/commit/role/parser/bug
changes). Do not create new "knowledge" files — keep it all in KNOWLEDGE.md.
