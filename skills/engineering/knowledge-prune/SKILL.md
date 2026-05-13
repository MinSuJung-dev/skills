---
name: knowledge-prune
description: Audit, clean, and optimize the .bugs/ knowledge base produced by the investigate skill. Use when the user says "정리해줘", "최적화해줘", "지식 베이스 정리", "prune", "clean up bugs", "consolidate patterns", or after many bugs have been investigated and the knowledge base has grown stale or inconsistent.
---

# Knowledge Prune

Maintain the `.bugs/` knowledge base so it stays accurate, deduplicated, and useful.
A stale knowledge base is worse than no knowledge base — it misleads the next investigation.

## Operating Principles

- **Read before writing.** Fully audit the knowledge base before making any changes.
- **Never delete, always archive.** Move stale records to `.bugs/archive/`, never hard-delete.
- **One pass per run.** Complete the full audit before applying any changes.
- **Report first, apply second.** Show the full plan to the user and get confirmation before modifying files.
- **Preserve traceability.** Every rule in a pattern file must keep its `<!-- BUG-NNN -->` source tag.

---

## Workflow

### Phase 1 — Full audit

Scan the entire `.bugs/` directory. Produce an inventory:

```bash
ls .bugs/bugs/
ls .bugs/patterns/
cat .bugs/INDEX.md
```

Collect every issue found across the five checks below. Do not fix anything yet.

---

#### Check 1 — Index integrity

Compare INDEX.md rows against actual files in `.bugs/bugs/`.

Flag:
- **Missing file** — INDEX.md row exists but `.bugs/bugs/BUG-NNN-*.md` does not
- **Orphan file** — `.bugs/bugs/BUG-NNN-*.md` exists but has no INDEX.md row
- **Field mismatch** — Status, Category, or Severity in INDEX.md differs from the bug card's YAML header

---

#### Check 2 — Duplicate detection

Scan the `Affected` column in INDEX.md and the `affected:` field in each bug card.

Flag:
- **Same file:line in two or more records** — likely the same bug tracked twice
- **Near-duplicate titles** — titles that differ only in wording but describe the same symptom

For each duplicate pair, recommend which to keep (prefer `verified` over `investigating`, newer over older).

---

#### Check 3 — Stale record detection

Flag:
- **`investigating` status older than 14 days** — bug card abandoned without closure
- **`reproduced` status older than 7 days** — investigation stalled after reproduction
- **`patched` status older than 3 days** — fix applied but never verified

For each stale record, propose one action: `archive` (no longer relevant) or `reopen` (still worth pursuing).

---

#### Check 4 — Pattern consolidation

For each pattern file in `.bugs/patterns/`:

1. List all prevention rules and their source tags (`<!-- BUG-NNN -->`).
2. Identify rules that express the same constraint in different words.
3. Propose merging them into one canonical rule (keeping all source tags).
4. Identify rules whose source bug was later marked `abandoned` — flag for removal.
5. Identify bug categories (from INDEX.md) that have 2+ verified bugs but no pattern file — flag as a **missing pattern**.

---

#### Check 5 — Adjacent risk follow-up

Read every closed bug card's `Adjacent risk:` field.

For each adjacent risk that has not yet been investigated:
- Check whether the named file:line has been touched since the bug was closed (via `git log`)
- If untouched: flag as **uninvestigated adjacent risk** — worth a quick check before it becomes a bug

---

### Phase 2 — Report

Present the full audit result before changing anything.

```
Knowledge Base Audit Report

Scanned: N bug cards, M pattern files
Last bug: BUG-NNN (<date>)

────────────────────────────────────────

INDEX INTEGRITY [N issues]
- MISSING FILE: BUG-003 — file .bugs/bugs/BUG-003-*.md not found
- ORPHAN FILE: BUG-007-orphaned-screen.md — not in INDEX.md
- FIELD MISMATCH: BUG-005 INDEX says "investigating", card says "verified"

DUPLICATES [N pairs]
- BUG-002 and BUG-006 share affected file lib/routes.dart:23
  → Recommend: keep BUG-002 (verified), archive BUG-006

STALE RECORDS [N]
- BUG-008 — status: investigating, last touched 21 days ago
  → Propose: archive (no recent activity)
- BUG-010 — status: patched, last touched 5 days ago
  → Propose: reopen for verification

PATTERN CONSOLIDATION [N]
- lifecycle-leak.md: rules 2 and 4 express the same constraint
  → Propose: merge into one rule, keep both source tags
- 3 verified bugs in category "schema-mismatch" but no patterns/schema-mismatch.md
  → Propose: create missing pattern file from BUG-004, BUG-009, BUG-011

ADJACENT RISK FOLLOW-UP [N]
- BUG-001 flagged lib/auth/token_store.dart as adjacent risk (closed 2026-03-10)
  → File untouched since — worth a quick audit

────────────────────────────────────────

PROPOSED ACTIONS
1. Fix index integrity (3 items)
2. Archive BUG-006 (duplicate of BUG-002)
3. Archive BUG-008 (stale, 21 days)
4. Reopen BUG-010 for verification
5. Merge lifecycle-leak.md rules 2+4
6. Create patterns/schema-mismatch.md from BUG-004, BUG-009, BUG-011
7. Flag adjacent risk in lib/auth/token_store.dart

Authorize all with "apply" or cherry-pick: "apply 1,2,5"
```

**Do not modify any file until the user authorizes.**

---

### Phase 3 — Apply (after authorization)

Apply only the authorized actions. For each:

**Index integrity fix**
- Add missing rows to INDEX.md from bug card YAML headers
- Remove rows whose files are confirmed deleted (not just missing)
- Update mismatched fields to match the bug card

**Archive**
- Move `.bugs/bugs/BUG-NNN-*.md` to `.bugs/archive/BUG-NNN-*.md`
- Update INDEX.md row: append `(archived)` to status
- Do not remove the INDEX.md row — it stays for traceability

**Reopen**
- Update bug card `status:` to `investigating`
- Update `last-touched:` date
- Update INDEX.md row

**Pattern consolidation**
- Merge duplicate rules into one canonical sentence
- Keep all `<!-- BUG-NNN -->` source tags on the merged rule
- Remove rules from abandoned bugs only if the bug is in `archive/`

**Create missing pattern file**
- Use the pattern file schema from the investigate skill
- Populate `Prevention rules` from the root cause of each contributing bug
- Populate `Known instances` table

**Adjacent risk**
- Add a note to the relevant bug card under a new `## Adjacent Risk Follow-up` section
- Do not open a new bug card unless the user confirms the risk is real

---

### Phase 4 — Confirm

After all changes are applied:

```
Knowledge Base Prune Complete

Changes applied:
✅ Fixed 3 index integrity issues
✅ Archived BUG-006 (duplicate)
✅ Archived BUG-008 (stale)
✅ Reopened BUG-010 for verification
✅ Merged lifecycle-leak.md rules 2+4
✅ Created patterns/schema-mismatch.md (3 rules from BUG-004, BUG-009, BUG-011)
⚠️  Adjacent risk noted: lib/auth/token_store.dart (not yet investigated)

Knowledge base: N bug cards (N verified, N investigating, N archived)
Pattern library: M categories, P total rules
```

---

## Anti-patterns (never do these)

- Hard-deleting any bug card or pattern rule
- Modifying files before showing the report to the user
- Removing a `<!-- BUG-NNN -->` source tag
- Merging bugs whose symptoms differ — similar file:line does not mean same bug
- Treating an adjacent risk as a confirmed bug without investigation
