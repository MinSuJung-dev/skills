---
name: investigate
description: Investigate hard bugs and performance regressions with a disciplined loop — build a feedback signal, reproduce, hypothesize falsifiably, instrument one variable at a time, fix, regression-test, and persist findings across sessions. Use when the user reports a bug, asks to debug, diagnose, or investigate something, says "it still doesn't work", "I tried that already", "same error", "it's broken again", or reports a performance regression. Maintains a persistent knowledge base under .bugs/ — bug cards, a master index, and a pattern library — so prior attempts, falsified hypotheses, and accumulated lessons survive across sessions and context resets. Adapts strictness: lenient on small isolated bugs, strict on large/intermittent/repeated-failure bugs.
---

# Investigate

A discipline for hard bugs. Designed against the AI failure mode of *fixing the same bug the same way twice and calling it different.*

Most of debugging is mechanical once you have a fast deterministic signal. The skill is **building that signal**, **not losing what you've already ruled out**, and **turning every fix into a lesson that prevents the next bug**.

## Operating Principles

- **The feedback loop is the skill.** Phase 1 deserves disproportionate effort. Everything downstream is mechanical given a good loop.
- **Falsify, don't confirm.** State the prediction a hypothesis makes. If you can't, it's a vibe.
- **One variable per attempt.** Causation is unrecoverable otherwise.
- **Persist state.** The knowledge base lives on disk under `.bugs/`. Memory across sessions is non-negotiable for repeated-failure bugs.
- **Propagate lessons.** Every fix produces a pattern entry. The knowledge base gets smarter with each bug.
- **Scope lock.** Stay within the scope identified at intake. Expanding requires explicit user approval.
- **Escalate on repetition.** Three failed attempts on related hypotheses → stop guessing, re-diagnose from root.

---

## Knowledge Base Structure

All state lives under `.bugs/` at the repo root.

```
.bugs/
├── INDEX.md                        ← master index of all bugs (required)
├── bugs/
│   ├── BUG-001-login-button-noop.md
│   ├── BUG-002-route-orphaned.md
│   └── ...
└── patterns/
    ├── lifecycle-leak.md
    ├── half-wired.md
    ├── orphaned.md
    ├── stub.md
    ├── schema-mismatch.md
    └── pattern-violation.md
```

### INDEX.md schema

```markdown
# Bug Index

| ID | Title | Category | Severity | Status | Affected |
|----|-------|----------|----------|--------|---------|
| BUG-001 | Login button noop | stub | critical | verified | lib/auth/login_screen.dart:47 |
| BUG-002 | Settings route unreachable | orphaned | high | investigating | lib/routes.dart:23 |
```

**Duplicate prevention:** Before creating a new bug record, check the `Affected` column for the same file:line. If a match exists, reopen that record — do not create a duplicate.

### Bug card schema

```markdown
---
id: BUG-NNN
title: <one-line title>
category: lifecycle-leak | half-wired | orphaned | stub | schema-mismatch | pattern-violation
severity: critical | high | medium | low
status: investigating | reproduced | patched | verified | abandoned
tier: small | large
created: <date>
last-touched: <date>
affected: <file:line, file:line>
---

## Symptom
<observable facts only — no interpretation>

## Feedback loop
Type: <failing-test | curl-script | cli-diff | headless-browser | trace-replay | harness | fuzz | bisection | differential | hitl>
Command: <exact command to run>
Latency: <seconds>
Determinism: <always | N/M | flaky>

## Reproduction
Steps:
1.
2.
Environment: <OS, version, branch, commit>

## Expected vs Actual
Expected:
Actual:

## Scope
<files / modules / layers in play at intake>

## Hypotheses (ranked)
- [ ] H1: <hypothesis>
      Prediction: if H1 holds, <X> will eliminate the bug / <Y> will worsen it
      Cost: low|med|high   Prior: low|med|high

## Attempts
### Attempt 1 — <date> — testing H1
Probe: <one specific change at one specific seam — file:line>
Tag: [DEBUG-xxxx]
Result: pass | fail | unclear
Evidence: <log output / repro outcome>
Learned: <ruled in or out>

## Falsified
- H1 — ruled out because <evidence>

## Current best guess
<one or two sentences>

## Post-mortem (filled at close)
Root cause:
Fix:
What would have prevented this:
Adjacent risk: <places the same class of bug might live>
Pattern propagated: <patterns/lifecycle-leak.md | none>
```

### Pattern file schema

Pattern files accumulate lessons from closed bugs. They are the long-term memory of the knowledge base.

```markdown
# Pattern: <category name>

## What it is
<one paragraph — what this failure mode looks like>

## How to spot it
- <signal 1>
- <signal 2>

## Prevention rules
- <rule derived from BUG-NNN>  <!-- BUG-NNN -->
- <rule derived from BUG-MMM>  <!-- BUG-MMM -->

## Known instances
| ID | File:Line | Summary |
|----|-----------|---------|
| BUG-001 | lib/auth/login_screen.dart:47 | Login button empty handler |
```

---

## Workflow

### Phase 0 — Knowledge base lookup *(always first)*

**Before doing anything else:**

1. Check if `.bugs/INDEX.md` exists. If not, create the empty index and the directory structure.
2. Scan INDEX.md for any record with overlapping `Affected` file:line or matching keywords from the current symptom.
3. If a match exists: read that bug card and ask the user *"Found `.bugs/bugs/BUG-NNN-<slug>.md` with similar symptoms — same bug?"* before proposing new hypotheses.
4. Check the relevant pattern file (e.g. `.bugs/patterns/lifecycle-leak.md`) for known prevention rules that apply. If any match, surface them before starting investigation — the fix may already be documented.

**The whole point of persistence is to not re-explore falsified ground.**

If no match: assign the next BUG-NNN id, create the bug card file, and add a row to INDEX.md with `Status: investigating`.

---

### Phase 1 — Confirm scope and tier the bug

Ask the user (skip if already explicit):
- What is the scope? (screen, feature, file, module)
- Any prior attempts to fix this?

Tier the bug. Determines gate strictness.

**Small bug → lenient**: scope is 1–2 files, error is clear, reproduction takes under a minute, no data loss / security / production impact, user hasn't reported repeated failures.

**Large bug → strict**: scope crosses layers or files, symptom is intermittent, user said "I tried that already" / "still broken" / "same error" / "again", data loss / security / production impact.

When in doubt, strict. User can override: "treat as small" / "go strict".

---

### Phase 2 — Build a feedback loop *(disproportionate effort here)*

**This is the skill.** A fast, deterministic, agent-runnable pass/fail signal lets bisection, hypothesis testing, and instrumentation all work cheaply. Without it, no amount of staring at code helps.

Ways to construct one, in roughly this order:

1. **Failing test** at whatever seam reaches the bug — unit, integration, e2e.
2. **Curl / HTTP script** against a running dev server.
3. **CLI invocation** with a fixture input, diffing stdout against a known-good snapshot.
4. **Headless browser** (Playwright / Puppeteer) — drives UI, asserts on DOM / console / network.
5. **Replay a captured trace.** Save a real request / payload / event log; replay through the bug path in isolation.
6. **Throwaway harness.** Minimal subset of the system that exercises the bug path with a single function call.
7. **Property / fuzz loop.** For "sometimes wrong output" — run 1000 random inputs, look for the failure.
8. **Bisection harness.** Automate "boot at state X, check, repeat" for `git bisect run`.
9. **Differential loop.** Same input through old vs new (or two configs), diff outputs.
10. **HITL bash script.** Last resort. Drive a human through a structured loop.

Iterate on the loop: faster? sharper? more deterministic? A 30-second flaky loop is barely better than no loop. A 2-second deterministic loop is a debugging superpower.

**Non-deterministic bugs:** goal is a higher reproduction rate, not a clean repro. Loop ×100, parallelise, add stress. 50% flake is debuggable; 1% is not.

**If you genuinely cannot build a loop:** stop and say so. List what was tried. Mark card `Status: abandoned` or ask for a captured artifact (HAR, log, core dump). Do not proceed to hypothesise without a loop (except small-tier override — record it on the card).

Record the chosen loop in the bug card before moving on.

---

### Phase 3 — Reproduce

Run the loop. Watch the bug appear. Confirm:

- The loop produces the failure mode the **user** described — not a different failure nearby.
- Reproducible across multiple runs, or at a high enough rate to debug against.
- Exact symptom captured (error message, wrong output, timing).

Update card `Status: reproduced`.

---

### Phase 4 — Hypothesize (3–5 ranked, falsifiable)

Generate **3 to 5 ranked hypotheses** before testing any. Single-hypothesis generation anchors on the first plausible idea.

Each hypothesis must be **falsifiable**:

> "If <H> is the cause, then <X> will eliminate the bug / <Y> will worsen it."

If you can't state the prediction, the hypothesis is a vibe — discard or sharpen.

If only one hypothesis comes to mind, force a second. Common cheap second hypotheses:
- The symptom location is not the cause location
- Data shape differs from assumed
- A previous fix in the same area introduced this
- Timing / async ordering
- Environment / config differs from expected

**Check the bug card's Falsified section first.** Don't re-explore ruled-out ground from prior sessions.

**Show the ranked list to the user before testing.** Don't block — proceed if user is AFK.

---

### Phase 5 — Instrument (one variable at a time)

Each probe maps to a specific prediction from Phase 4.

Tool preference:
1. **Debugger / REPL inspection** — one breakpoint beats ten logs.
2. **Targeted logs** at the boundaries that distinguish hypotheses.
3. Never "log everything and grep".

**Tag every debug log** with a unique prefix, e.g. `[DEBUG-a4f2]`. Record the tag on the card. Cleanup is a single grep.

**Never combine isolation with the fix in one step.** Adding a log + changing the function is two variables.

After each attempt: record in the card under Attempts (probe, tag, result, evidence, what's learned). Classify: **falsified** / **confirmed** / **unclear**.

---

### Phase 6 — Fix + regression test

Only after a hypothesis is confirmed.

Write the regression test **before the fix**, but only if there is a **correct seam** for it. A correct seam exercises the real bug pattern at the actual call site.

If no correct seam exists, note it on the card. The architecture is preventing the bug from being locked down — flag for post-mortem.

If a correct seam exists:
1. Turn the minimised repro into a failing test.
2. Watch it fail.
3. Apply the fix.
4. Watch it pass.
5. Re-run the Phase 2 loop against the original scenario.

**Scope lock.** If the fix requires touching files outside Phase 1 intake, stop and ask the user.

---

### Phase 7 — Verify (binary)

Re-run the exact reproduction steps. Result must be binary:
- ✅ Original symptom gone AND no new symptom introduced
- ❌ Otherwise → fix is not verified

For non-deterministic bugs, run at the **same elevated rate** as Phase 3.

"Looks better" / "should be fixed" / "probably works" are not verification.

**If verification fails:** append the failed attempt to the card. Do not retry the same hypothesis with a cosmetic variation — that is the loop trap. Falsify the hypothesis, move to next.

**Escalation gate:** after 3 failed attempts on related hypotheses — stop. Re-read the card from the top. Question assumptions: Is the repro correct? Is the scope right? Is the loop sharp? Reset hypothesis list.

---

### Phase 8 — Close and propagate *(self-reinforcing loop)*

When verified:

**1. Close the bug card.**

```
Status: verified
Post-mortem:
  Root cause: <one line>
  Fix: <one line>
  What would have prevented this: <one line>
  Adjacent risk: <places the same class of bug might live>
  Pattern propagated: <patterns/lifecycle-leak.md | none>
```

**2. Propagate to the pattern library.**

Determine the bug's category (lifecycle-leak, half-wired, orphaned, stub, schema-mismatch, pattern-violation).

Open (or create) the matching file under `.bugs/patterns/`. Add:
- A new prevention rule derived from this bug's root cause, tagged `<!-- BUG-NNN -->`
- A row in the Known Instances table

Example entry in `patterns/lifecycle-leak.md`:
```
- StreamSubscription must be stored in a variable and cancelled in dispose() <!-- BUG-001 -->
```

**3. Update INDEX.md** — change the row's Status to `verified`.

**4. Remove all `[DEBUG-...]` instrumentation** — grep the tag prefix, remove every match.

**5. Note the root cause in the commit / PR message** so the next debugger learns.

---

This propagation step is the self-reinforcing loop:

```
Bug found
  ↓
BUG-NNN card created → INDEX.md updated
  ↓
Investigation → fix → verify
  ↓
Pattern propagated to .bugs/patterns/<category>.md
  ↓
Next investigation checks patterns first → same class of bug caught earlier
  ↓
New class of bug → loop repeats → pattern library grows
```

---

## Anti-loop heuristics

If any of these fire mid-investigation, stop and re-evaluate:

- Same file edited three times for the same bug → wrong file
- Each "fix" requires another "fix" → wrong root cause
- "Just one more change" said twice → escalate to Phase 7
- Scope expanded without explicit user approval → revert and ask
- Loop latency creeping up → fix the loop, not the bug
- Hypothesis count dropped to 1 → force a second

## Stack-specific pattern categories

When propagating to `.bugs/patterns/`, use these category signals:

**lifecycle-leak** — StreamSubscription / listener with no dispose, useEffect with no cleanup, duplicate subscriptions

**half-wired** — handler runs but no UI feedback, async with no loading/error state, mutation with no invalidation

**orphaned** — route defined but never pushed, component imported nowhere, service registered but no consumer

**stub** — TODO / FIXME / UnimplementedError, empty handler body, function returning hardcoded null

**schema-mismatch** — API response shape differs from model, DB column type mismatch, serialization error

**pattern-violation** — project-specific convention broken (naming, layer boundary, data flow direction)
