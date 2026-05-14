---
name: knowledge-prune
description: Audit, clean, and optimize the .bugs/ knowledge base produced by the investigate skill. Use when the user says "정리해줘", "최적화해줘", "prune", "clean up bugs", "consolidate patterns", or after many bugs have accumulated and the knowledge base has grown stale or inconsistent.
---

# Knowledge Prune

Maintain the `.bugs/` knowledge base so it stays accurate, deduplicated, and useful.
A stale knowledge base misleads the next investigation.

See [CHECKS.md](CHECKS.md) for detailed check procedures and report/apply formats.

## Operating Principles

- **Read before writing.** Fully audit before making any changes.
- **Never delete, always archive.** Move stale records to `.bugs/archive/`, never hard-delete.
- **Report first, apply second.** Show the full plan and get confirmation before modifying files.
- **Preserve traceability.** Every rule in a pattern file must keep its `<!-- BUG-NNN -->` source tag.

## Workflow

**Phase 1 — Full audit** *(collect all issues before fixing anything)*

Run five checks across `.bugs/`:

1. **Index integrity** — INDEX.md rows vs actual files: missing files, orphan files, field mismatches
2. **Duplicate detection** — same `file:line` in two or more records; near-duplicate titles
3. **Stale records** — `investigating` >14 days, `reproduced` >7 days, `patched` >3 days
4. **Pattern consolidation** — duplicate rules in pattern files; missing pattern files for categories with 2+ verified bugs; rules from abandoned bugs
5. **Adjacent risk follow-up** — closed bug cards' `Adjacent risk:` fields not yet investigated

**Phase 2 — Report**
Present all findings grouped by check. Propose one action per issue. Do not modify any file until the user authorizes. See [CHECKS.md](CHECKS.md) for the exact report format.

**Phase 3 — Apply** *(authorized actions only)*
- **Index fix**: add missing rows, update mismatched fields
- **Archive**: move to `.bugs/archive/`, update INDEX.md status to `(archived)` — keep the row
- **Reopen**: update status to `investigating`, refresh `last-touched`
- **Pattern merge**: combine duplicate rules into one canonical sentence, keep all source tags
- **Create pattern file**: populate from root causes of contributing verified bugs
- **Adjacent risk**: add note to the original bug card, do not open a new card without user confirmation

**Phase 4 — Confirm**
Report a summary of all changes applied and the updated knowledge base stats (N cards, N verified, N archived, M pattern categories, P total rules).

## Anti-patterns

- Hard-deleting any bug card or pattern rule
- Modifying files before showing the report
- Removing a `<!-- BUG-NNN -->` source tag
- Merging bugs whose symptoms differ — same file:line ≠ same bug
- Treating an adjacent risk as a confirmed bug without investigation
