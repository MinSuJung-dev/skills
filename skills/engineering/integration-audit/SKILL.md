---
name: integration-audit
description: 최근 구현한 기능이 실제로 완성됐는지 감사한다. "감사해줘", "확인해줘", "PR 전에 검토", "동작해야 하는데 안 돼", "왜 안 되지", 또는 완성된 것처럼 보이는데 동작하지 않는 버튼/라우트/기능을 언급할 때 사용. 심각도와 신뢰도를 포함한 결과를 보고한다. 명시적 승인 없이는 코드를 수정하지 않는다.
---

# Integration Audit

Verify that a recently implemented feature is actually finished.
Find the gap between what looks done and what works.

See [STACK-SIGNALS.md](STACK-SIGNALS.md) for Flutter/Web/Backend specific detection patterns.

## Operating Principles

- **Verify, don't modify.** Find and report. Do not change code until the user explicitly authorizes a fix.
- **Local, not global.** Audit only the stated scope. Never drift into repo-wide cleanup.
- **Prove, don't assume.** Trace every execution path to its end. "Looks fine" is not a verdict.
- **Conservative on runtime.** DI, codegen, reflection, platform channels may have no static callers — lower confidence before flagging as dead.

## Workflow

**Step 1 — Confirm scope**
Ask if the target is vague (feature name, description). Skip if the user already named a specific file, PR number, commit hash, or directory.

**Step 2 — Inventory** *(extract every element in these layers)*
- **UI**: onClick/onTap/onPressed/onChanged/onSubmit, form elements, navigation triggers
- **Routing**: new route definitions, route→component mappings, guards
- **State/data**: Provider/Bloc/ViewModel registrations, mutations, API calls
- **Platform**: MethodChannel handlers, DI registrations, generated code references

**Step 3 — Runtime pre-check**
Before tracing, check if the element is runtime-registered (DI container, codegen, reflection, platform channels, lifecycle callbacks). If plausible → drop confidence to low, move to "Needs verification". Do not trace further.

**Step 4 — Trace each element**
- **A. Does the handler do real work?** Empty `() => {}` → dead. `console.log`/`print` only → dead. TODO/throw UnimplementedError → stub. Calls external function → go to B.
- **B. Does it reach a real side effect?** DB, API, store, URL, localStorage, native channel — if it breaks before → half-wired.
- **C. Does the UI reflect the change?** Refetch/invalidate/setState/subscription — at least one. Loading and error states for async — if none → half-wired.
- **D. Is the element reachable and consumed?** No entry path or no consumer → orphaned.

**Step 5 — Classify** *(use only these six types)*
`dead` | `stub` | `half-wired` | `orphaned` | `temporary` | `lifecycle-leak`
Severity: `critical` | `high` | `medium` | `low`
Confidence: `high` | `medium` | `low`

## Self-check before reporting
- [ ] Confirmed scope with the user (or skipped because explicit)?
- [ ] Every finding cites file:line?
- [ ] Confidence dropped to low for runtime-registered code?
- [ ] Codebase unmodified?

**Step 6 — Report**
Sort by severity, then confidence. Every finding must cite **file:line**.

```
Audit Result
Scope: [e.g. PR #142]
Inventory: [N handlers / N routes / N mutations / N DI registrations]

CRITICAL [N]
1. lib/features/auth/login_screen.dart:47
   Type: dead | Confidence: high
   Evidence: () => {} empty handler.
   Impact: Users cannot log in.
   Suggested fix: Call AuthBloc.add(LoginRequested(...))

NEEDS VERIFICATION (low confidence) [N]
- lib/di/injection.dart:31 — no direct caller (get_it DI; runtime injection plausible)

SAFE AUTO-FIX CANDIDATES
- Remove debug print statements (3)
Authorize with "fix Critical #1" or "remove all debug logs".
```

## Anti-patterns
- Modifying code without authorization
- Reporting issues outside stated scope
- Fixing mid-audit — complete inventory first
- Flagging runtime-registered code as dead with high confidence
- Commenting on style, naming, formatting, or test coverage
