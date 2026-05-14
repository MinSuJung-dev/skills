---
name: diagnose
description: Disciplined diagnosis loop for hard bugs and performance regressions. Reproduce → minimise → hypothesise → instrument → fix → regression-test. Use when user says "diagnose this" / "debug this", reports a bug, says something is broken/throwing/failing, or describes a performance regression.
source: https://github.com/mattpocock/skills/blob/main/skills/engineering/diagnose/SKILL.md
---

# Diagnose

A discipline for hard bugs. Skip phases only when explicitly justified.

## Phase 1 — Build a feedback loop

**This is the skill.** Everything else is mechanical. Without a fast, deterministic, agent-runnable pass/fail signal, no amount of staring at code helps.

Ways to construct one (try in order): failing test → curl/HTTP script → CLI diff → headless browser → trace replay → throwaway harness → fuzz loop → bisection harness → differential loop → HITL script.

Iterate the loop: faster? sharper signal? more deterministic? A 2-second deterministic loop is a superpower. A 30-second flaky loop is barely better than nothing.

Non-deterministic bugs: raise the reproduction rate (loop ×100, add stress, narrow timing). 50% flake is debuggable; 1% is not.

Cannot build a loop? Stop. List what you tried. Ask for a captured artifact or env access. Do not proceed to hypothesise.

## Phase 2 — Reproduce

Run the loop. Confirm it produces the failure the **user** described — not a nearby different failure. Capture the exact symptom.

## Phase 3 — Hypothesise

Generate **3–5 ranked, falsifiable hypotheses** before testing any.

> "If X is the cause, then changing Y will make the bug disappear / changing Z will worsen it."

Can't state the prediction? It's a vibe — discard or sharpen. Show the ranked list to the user before testing.

## Phase 4 — Instrument

Each probe maps to one prediction. **One variable at a time.** Tag every debug log `[DEBUG-xxxx]` — cleanup becomes a single grep. For perf regressions: measure first, fix second.

## Phase 5 — Fix + regression test

Write the failing test **before** the fix — only if a correct seam exists (exercises the real bug pattern at the call site). If no correct seam: note it, flag for architecture review.

1. Failing test → watch it fail → apply fix → watch it pass → re-run Phase 1 loop.

## Phase 6 — Cleanup + post-mortem

- [ ] Original repro no longer reproduces
- [ ] Regression test passes (or absent seam documented)
- [ ] All `[DEBUG-...]` removed
- [ ] Throwaway prototypes deleted
- [ ] Correct hypothesis noted in commit/PR message

What would have prevented this bug? If architectural → hand off to `/improve-codebase-architecture` after the fix is in.
