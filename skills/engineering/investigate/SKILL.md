---
name: investigate
description: Investigate hard bugs and performance regressions with a disciplined loop. Maintains a persistent knowledge base under .bugs/ — bug cards, master index, and pattern library — so findings survive across sessions. Use when the user reports a bug, says "it still doesn't work", "I tried that already", "same error", "it's broken again", or reports a performance regression.
---

# Investigate

Discipline for hard bugs. Designed against the AI failure mode of fixing the same bug twice.

See [BUG-CARD.md](BUG-CARD.md) for bug card and index schemas. See [PATTERNS.md](PATTERNS.md) for pattern library structure.

## Operating Principles

- **Feedback loop first.** Everything downstream is mechanical once you have a fast deterministic signal.
- **Falsify, don't confirm.** Every hypothesis must state its prediction. If you can't, it's a vibe.
- **One variable per attempt.** Causation is unrecoverable otherwise.
- **Persist state.** Bug card lives on disk. Memory across sessions is non-negotiable.
- **Propagate lessons.** Every fix produces a pattern entry. The knowledge base gets smarter.
- **Escalate on repetition.** 3 failed attempts on related hypotheses → stop, re-diagnose from root.

## Workflow

**Phase 0 — Knowledge base lookup** *(always first)*
Check `.bugs/INDEX.md` for overlapping file:line or matching keywords. If a match exists, read that card first and ask the user before proposing new hypotheses. Check the relevant `.bugs/patterns/<category>.md` for known prevention rules. If no match: assign next BUG-NNN id, create the card, add a row to INDEX.md.

**Phase 1 — Confirm scope and tier**
Small bug (1–2 files, clear error, under a minute to reproduce) → lenient. Large bug (crosses layers, intermittent, user said "tried that already") → strict. When in doubt, strict.

**Phase 2 — Build a feedback loop** *(disproportionate effort)*
Try in order: failing test → curl/HTTP script → CLI diff → headless browser → trace replay → throwaway harness → fuzz → bisection → differential → HITL script. Iterate until fast, sharp, and deterministic. Cannot build a loop? Stop and say so — do not proceed to hypothesise.

**Phase 3 — Reproduce**
Run the loop. Confirm it produces the failure the user described. Update card `Status: reproduced`.

**Phase 4 — Hypothesize (3–5 ranked, falsifiable)**
Each hypothesis: *"If H holds, then X will eliminate the bug / Y will worsen it."* Check the card's Falsified section first. Show ranked list to user before testing.

**Phase 5 — Instrument (one variable at a time)**
Each probe maps to a specific prediction. Tag every debug log `[DEBUG-xxxx]`. Record probe, tag, result, evidence on the card after each attempt.

**Phase 6 — Fix + regression test**
Write the failing test before the fix, but only if a correct seam exists. If not, note it on the card. Scope lock: touching files outside Phase 1 intake requires explicit user approval.

**Phase 7 — Verify (binary)**
Re-run the Phase 2 loop. ✅ original symptom gone, no new symptom. ❌ otherwise → fix not verified. For flaky bugs, run at the same elevated rate as Phase 3. 3 failed attempts → escalate: re-read card from top, reset hypothesis list.

**Phase 8 — Close and propagate**
1. Fill post-mortem on the card (root cause, fix, what would have prevented, adjacent risk).
2. Add a prevention rule to `.bugs/patterns/<category>.md` tagged `<!-- BUG-NNN -->`.
3. Update INDEX.md status to `verified`.
4. Remove all `[DEBUG-...]` instrumentation.

## Anti-loop heuristics

Same file edited 3× for same bug → wrong file. Each fix needs another fix → wrong root cause. Hypothesis count dropped to 1 → force a second. Scope expanded without approval → revert and ask.
