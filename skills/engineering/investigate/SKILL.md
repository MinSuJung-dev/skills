---
name: investigate
description: Investigate hard bugs and performance regressions with a disciplined loop — build a feedback signal, reproduce, hypothesize falsifiably, instrument one variable at a time, fix, regression-test, and persist findings across sessions. Use when the user reports a bug, asks to debug, diagnose, or investigate something, says "it still doesn't work", "I tried that already", "same error", "it's broken again", or reports a performance regression. Maintains a persistent bug card file under .bugs/ so prior attempts and falsified hypotheses survive across sessions and context resets. Adapts strictness: lenient on small isolated bugs, strict on large/intermittent/repeated-failure bugs.
---

# Investigate

A discipline for hard bugs. Designed against the AI failure mode of *fixing the same bug the same way twice and calling it different.*

Most of debugging is mechanical once you have a fast deterministic signal. The skill is **building that signal** and **not losing what you've already ruled out**.

## Operating Principles

- **The feedback loop is the skill.** Phase 1 deserves disproportionate effort. Everything downstream is mechanical given a good loop.
- **Falsify, don't confirm.** State the prediction a hypothesis makes. If you can't, it's a vibe.
- **One variable per attempt.** Causation is unrecoverable otherwise.
- **Persist state.** Bug card lives on disk under `.bugs/`. Memory across sessions is non-negotiable for repeated-failure bugs.
- **Scope lock.** Stay within the scope identified at intake. Expanding requires explicit user approval.
- **Escalate on repetition.** Three failed attempts on related hypotheses → stop guessing, re-diagnose from root.

## Bug Card — single source of truth across sessions

The bug card is a markdown file at `.bugs/<slug>.md` (repo root, slug like `2026-05-13-login-button-noop.md`).

**On every invocation, before doing anything else**: list `.bugs/`. If a card with overlapping keywords or matching symptom exists, read it first and ask the user *"Found `.bugs/<slug>.md` with similar symptoms — same bug?"* before proposing new hypotheses. The whole point of persistence is to not re-explore falsified ground.

### Schema

```markdown
# Bug: <one-line title>

Status: investigating | reproduced | patched | verified | abandoned
Tier: small | large
Created: <date>   Last touched: <date>

## Symptom
<observable facts only — no interpretation>

## Feedback loop
Type: <failing-test | curl-script | cli-diff | headless-browser | trace-replay | harness | fuzz | bisection | differential | hitl>
Command: <exact command to run>
Latency: <seconds>
Determinism: <always | N/M | flaky>
Notes: <what makes it fast/sharp/deterministic, or why it isn't yet>

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
- [ ] H2: ...

## Attempts
### Attempt 1 — <date> — testing H1
Probe: <one specific change at one specific seam — file:line>
Tag: [DEBUG-a4f2]
Result: pass | fail | unclear
Evidence: <log output / repro outcome>
Learned: <ruled in or out>

### Attempt 2 ...

## Falsified
- H1 — ruled out because <evidence>

## Current best guess
<one or two sentences>

## Post-mortem (filled at close)
Root cause:
Fix:
What would have prevented this:
Adjacent risk:
```

Update the card **before and after every action**. If it gets stale, the loop trap reopens.

## Severity tiering — chooses game-mode

Tier the bug at intake. Determines gate strictness.

**Small bug → lenient**: scope is 1–2 files, error message is clear and specific, reproduction takes under a minute, no data loss / security / production impact, user hasn't reported repeated failures.

**Large bug → strict**: scope crosses layers or files, symptom is intermittent or hard to reproduce, user said any of "I tried that already" / "still broken" / "same error" / "again" / "third time", data loss / security / production / live impact.

When in doubt, strict. User can override at any time: "treat as small" / "go strict".

## Workflow

### Phase 1 — Build a feedback loop *(disproportionate effort here)*

**This is the skill.** A fast, deterministic, agent-runnable pass/fail signal lets bisection, hypothesis testing, and instrumentation all work cheaply. Without it, no amount of staring at code helps.

#### Ways to construct one, in roughly this order

1. **Failing test** at whatever seam reaches the bug — unit, integration, e2e.
2. **Curl / HTTP script** against a running dev server.
3. **CLI invocation** with a fixture input, diffing stdout against a known-good snapshot.
4. **Headless browser** (Playwright / Puppeteer) — drives UI, asserts on DOM / console / network.
5. **Replay a captured trace.** Save a real request / payload / event log to disk; replay through the bug code path in isolation.
6. **Throwaway harness.** Minimal subset of the system (one service, mocked deps) that exercises the bug path with a single function call.
7. **Property / fuzz loop.** For "sometimes wrong output" — run 1000 random inputs, look for the failure.
8. **Bisection harness.** If the bug appeared between two known states, automate "boot at state X, check, repeat" for `git bisect run`.
9. **Differential loop.** Same input through old vs new (or two configs), diff outputs.
10. **HITL bash script.** Last resort. Drive a human through a structured loop; captured output feeds back. Don't lean on this unless 1–9 are exhausted.

#### Iterate on the loop itself

Treat the loop as a product. Once you have *a* loop, ask:
- Faster? (Skip unrelated init, narrow test scope, cache setup.)
- Sharper? (Assert the specific symptom, not just "didn't crash".)
- More deterministic? (Pin time, seed RNG, isolate filesystem, freeze network.)

A 30-second flaky loop is barely better than no loop. A 2-second deterministic loop is a debugging superpower.

#### Non-deterministic bugs

Goal is not a clean repro but a **higher reproduction rate**. Loop the trigger ×100, parallelise, add stress, narrow timing windows, inject sleeps at suspect async boundaries. **50% flake is debuggable; 1% is not.** Raise the rate first.

#### When you genuinely cannot build a loop

Stop and say so explicitly. List what was tried. Mark card `Status: abandoned: cannot reproduce` or ask the user for: (a) access to the environment that reproduces, (b) a captured artifact (HAR, log dump, core dump, screen recording with timestamps), or (c) permission to add temporary production instrumentation. **Do not proceed to hypothesise without a loop** unless tier is small and the user explicitly says "skip repro, just try X" — record the override on the card.

Record the chosen loop in the bug card with its latency and determinism before moving on.

### Phase 2 — Reproduce

Run the loop. Watch the bug appear. Confirm:

- The loop produces the failure mode the **user** described — not a different failure that happens nearby. Wrong bug = wrong fix.
- Reproducible across multiple runs, or at a high enough rate to debug against.
- Exact symptom captured (error message, wrong output, timing) so verification later is unambiguous.

Update card `Status: reproduced`. Do not proceed without reproduction (except for the small-tier override above).

### Phase 3 — Hypothesize (3–5 ranked, falsifiable)

Generate **3 to 5 ranked hypotheses** *before testing any of them*. Single-hypothesis generation anchors on the first plausible idea and burns the next two hours.

Each hypothesis must be **falsifiable** — state the prediction:

> "If <H> is the cause, then <X> will eliminate the bug / <Y> will worsen it."

If you can't state the prediction, the hypothesis is a vibe — discard or sharpen.

If only one hypothesis comes to mind, **force a second**. Common cheap second hypotheses:
- The symptom location is not the cause location
- Data shape differs from assumed
- A previous fix in the same area introduced this
- Timing / async ordering
- Environment / config differs from expected

**Show the ranked list to the user before testing.** They often re-rank instantly ("we just deployed a change to #3") or know hypotheses they already ruled out. Cheap checkpoint, big time saver. Don't block — proceed with your ranking if user is AFK.

**Before adding any new hypothesis**, check the card's Falsified section. Don't re-explore ruled-out ground from prior sessions.

### Phase 4 — Instrument (one variable at a time)

Each probe maps to a specific prediction from Phase 3.

Tool preference:
1. **Debugger / REPL inspection** if the env supports it. One breakpoint beats ten logs.
2. **Targeted logs** at the boundaries that distinguish hypotheses.
3. Never "log everything and grep".

**Tag every debug log** with a unique prefix per session, e.g. `[DEBUG-a4f2]`. Cleanup is then a single grep. Record the tag on the card. Untagged logs survive; tagged logs die.

**Never combine isolation with the fix in one step.** Adding a log + changing the function is two variables. Isolate first, then fix.

**Perf branch.** For performance regressions, logs are usually the wrong tool. Establish a baseline measurement (timing harness, `performance.now()`, profiler, query plan), then bisect. Measure first, fix second.

After each attempt:
1. Record in the card under Attempts (probe, tag, result, evidence, what's learned).
2. Classify result: **falsified** (cross H off, move to next) / **confirmed** (proceed to fix) / **unclear** (refine the probe, don't advance).

### Phase 5 — Fix + regression test

Only after a hypothesis is confirmed.

Write the regression test **before the fix**, but only if there's a **correct seam** for it.

A correct seam exercises the **real bug pattern as it occurs at the call site**. If the only available seam is too shallow (single-caller test when the bug needs multiple callers, unit test that can't replicate the trigger chain), a regression test there gives false confidence.

**If no correct seam exists, that itself is a finding.** Note it on the card. The architecture is preventing the bug from being locked down — flag for post-mortem.

If a correct seam exists:
1. Turn the minimised repro into a failing test at that seam.
2. Watch it fail.
3. Apply the fix.
4. Watch it pass.
5. Re-run the Phase 1 loop against the original (un-minimised) scenario.

**Scope lock.** If the fix requires expanding scope (touching files outside Phase 1 intake), **stop and ask the user** before continuing. Don't quietly broaden.

### Phase 6 — Verify (binary)

Re-run the exact reproduction steps. Result must be **binary**:
- ✅ Original symptom no longer occurs AND no new symptom introduced
- ❌ Otherwise → fix is not verified

For non-deterministic bugs, run the repro at the **same elevated rate** as Phase 2 — if Phase 2 ran ×100, Phase 6 runs ×100. A passing single run doesn't verify a flake fix.

Run any existing tests that touch modified files.

"Looks better" / "should be fixed now" / "probably works" are not verification.

### Phase 7 — Failure accumulation and escalation

If verification fails:

1. Append the failed attempt to the card under Attempts with full evidence.
2. **Do not retry the same hypothesis with a cosmetic variation.** That is the loop trap.
3. Falsify the current hypothesis explicitly on the card. Move to next.

**Escalation gate**: after **3 failed attempts** on related hypotheses:
- Stop attempting fixes.
- Re-read the card from the top.
- Question assumptions: Is the repro correct? Is the symptom what we think? Is the scope right? Is the loop actually sharp?
- Often the answer: the bug is not where we've been looking. Reset hypothesis list.

Optional escalation tools:
- **Bisection** — "it used to work, now doesn't" → `git bisect run` with the Phase 1 loop.
- **Sharper loop** — go back to Phase 1, make the loop faster/sharper/more deterministic. Often the real fix.

### Phase 8 — Close and post-mortem

When verified:

1. Card `Status: verified`.
2. Fill Post-mortem section:
   - Root cause (one line)
   - Fix (one line)
   - What would have prevented this
   - Adjacent risk — places the same class of bug might live
3. Remove all `[DEBUG-...]` instrumentation (`grep` the prefix).
4. Delete throwaway prototypes or move them to a clearly-marked debug location.
5. Note the correct hypothesis in commit / PR message so the next debugger learns.

If post-mortem reveals architectural cause (no good test seam, tangled callers, hidden coupling), note it on the card. Make the architectural recommendation **after** the fix is in — you have more information now.

## Anti-loop heuristics (red flags during the run)

If any of these fire mid-investigation, stop and re-evaluate:

- Same file edited three times for the same bug → wrong file
- Each "fix" requires another "fix" → wrong root cause
- "Just one more change" said twice → escalate to Phase 7
- Scope expanded without explicit user approval → revert and ask
- Loop latency creeping up → go fix the loop, not the bug
- Hypothesis count dropped to 1 → force a second per Phase 3