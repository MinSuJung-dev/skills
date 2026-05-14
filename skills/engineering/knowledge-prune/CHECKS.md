# Checks, Report Format, and Apply Procedures

## Check Details

### Check 1 — Index integrity
```bash
ls .bugs/bugs/
cat .bugs/INDEX.md
```
Flag:
- **Missing file** — INDEX.md row exists but `.bugs/bugs/BUG-NNN-*.md` does not
- **Orphan file** — file exists but has no INDEX.md row
- **Field mismatch** — Status/Category/Severity in INDEX.md differs from bug card YAML header

### Check 2 — Duplicate detection
Scan `Affected` column in INDEX.md and `affected:` in each card.
Flag: same file:line in 2+ records; near-duplicate titles.
Recommend: keep `verified` over `investigating`, newer over older.

### Check 3 — Stale records
- `investigating` older than 14 days → propose `archive` or `reopen`
- `reproduced` older than 7 days → propose `reopen`
- `patched` older than 3 days → propose `reopen` for verification

### Check 4 — Pattern consolidation
For each `.bugs/patterns/<category>.md`:
1. List all rules and their `<!-- BUG-NNN -->` source tags
2. Identify rules expressing the same constraint — propose merge
3. Identify rules from abandoned bugs — propose removal
4. Check INDEX.md for categories with 2+ verified bugs but no pattern file — propose creation

### Check 5 — Adjacent risk follow-up
Read every closed card's `Adjacent risk:` field. For each uninvestigated risk, check `git log` to see if the named file was touched since the bug closed.

## Report Format

```
Knowledge Base Audit Report

Scanned: N bug cards, M pattern files

────────────────────────────────────────

INDEX INTEGRITY [N]
- MISSING FILE: BUG-003
- ORPHAN FILE: BUG-007-orphaned-screen.md
- FIELD MISMATCH: BUG-005 index=investigating, card=verified

DUPLICATES [N]
- BUG-002 and BUG-006 share lib/routes.dart:23 → keep BUG-002 (verified), archive BUG-006

STALE RECORDS [N]
- BUG-008 — investigating, 21 days → propose archive
- BUG-010 — patched, 5 days → propose reopen

PATTERN CONSOLIDATION [N]
- lifecycle-leak.md rules 2+4 are equivalent → propose merge
- 3 verified bugs in schema-mismatch but no pattern file → propose create

ADJACENT RISK [N]
- BUG-001 flagged lib/auth/token_store.dart (untouched since close)

────────────────────────────────────────

PROPOSED ACTIONS
1. Fix index integrity (3 items)
2. Archive BUG-006
3. Archive BUG-008
4. Reopen BUG-010
5. Merge lifecycle-leak.md rules 2+4
6. Create patterns/schema-mismatch.md
7. Flag adjacent risk in lib/auth/token_store.dart

Authorize all with "apply" or cherry-pick: "apply 1,2,5"
```

## Confirm Format

```
Knowledge Base Prune Complete

✅ Fixed 3 index integrity issues
✅ Archived BUG-006 (duplicate)
✅ Archived BUG-008 (stale)
✅ Reopened BUG-010
✅ Merged lifecycle-leak.md rules 2+4
✅ Created patterns/schema-mismatch.md (3 rules)
⚠️  Adjacent risk noted: lib/auth/token_store.dart

Stats: N cards (N verified, N investigating, N archived) | M categories, P rules
```
