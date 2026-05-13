---
name: integration-audit
description: Audit whether a recently implemented feature is actually complete — find dead handlers, stub functions, orphaned routes, half-wired mutations, placeholder UI, temporary debug logic, and lifecycle leaks. Use when the user says "audit", "verify", "double-check", "review before PR/merge", "should work but doesn't", "why isn't this working", or mentions a button/route/feature that looks done but isn't behaving. Reports findings with severity and confidence; never modifies code without explicit user authorization.
---

# Integration Audit

Verify that a recently implemented feature is actually finished.
Find the gap between what looks done and what works.

## Operating Principles

- **Verify, don't modify.** Find and report. Do not change code until the user explicitly authorizes a specific fix.
- **Local, not global.** Audit only the user's stated scope or recent changes. Never drift into repo-wide cleanup.
- **Prove, don't assume.** Trace every execution path to its end. "Looks fine" is not a verdict.
- **Conservative on runtime.** Code registered at runtime (DI, codegen, reflection, platform channels) may have no static callers. Lower confidence before flagging it as dead.

## Workflow

### Step 1 — Confirm scope

Ask the user only if the target is vague (a feature name, a description, "the new thing").
Skip the question if the user already named a specific file, PR number, commit hash, or directory.

When asking:
- What is the audit target? (a PR, last N commits, a feature directory, a single screen)
- What is the base reference? (usually `main` or `develop`)

Scope discovery commands:

```bash
git diff --name-only <base>..HEAD          # changed files
git log --oneline <base>..HEAD             # change history
git diff <base>..HEAD -- '**/*.dart'       # filtered by extension
```

**Never scan the whole repo unless the user explicitly asks.**

### Step 2 — Inventory interactive and integration points

Within scope, extract and list every element in these four layers:

**UI layer**
- Event handlers: `onClick`, `onTap`, `onPressed`, `onChanged`, `onSubmit`, `onLongPress`
- Form elements: `<form>`, `TextField`, `Form`, validators
- Navigation triggers: `<Link>`, `Navigator.push`, `context.go`, route calls

**Routing layer**
- New route definitions
- Route → component/screen mappings
- Route guards and redirects

**State / data layer**
- New Provider / Bloc / ViewModel / Store registrations
- Mutation functions (`useMutation`, `dispatch`, repository methods)
- API calls (`fetch`, `http.get`, `dio`, repository wrappers)

**Platform / integration layer (when applicable)**
- MethodChannel handlers
- Native plugin registrations
- DI container registrations (`get_it`, `injectable`, Riverpod providers)
- Generated code references (`.g.dart`, `.freezed.dart`)

### Step 3 — Runtime-registration pre-check

Before tracing any element, verify it is not runtime-registered. The following are easy to misjudge:

- Classes registered in a DI container
- Annotation or reflection targets
- Generated code (`build_runner`, codegen) references
- Platform channel handlers (both Dart and native side)
- Framework lifecycle callbacks

If runtime registration is plausible, **drop confidence to low** and move the element to "Needs verification". Do not trace further.

### Step 4 — Trace each element

For every element that passed the runtime pre-check, follow this decision path:

**A. Does the handler do real work?**
- Empty `() => {}` → **dead**
- Only `console.log` / `print` / `debugPrint` → **dead**
- Only `// TODO`, `// FIXME`, `throw UnimplementedError()` → **stub**
- Calls an external function → go to B

**B. Does the called function reach a real side effect?**
- Follow the definition through every layer
- Does it reach DB, API, store, URL, localStorage, or a native channel?
- Breaks before reaching one → **half-wired**

**C. Does the UI reflect the change?**
- Refetch, invalidate, optimistic update, store subscription, or `setState` — at least one?
- For async work, are loading and error states exposed in the UI?
- None of the above → **half-wired** (missing feedback)

**D. Is the element reachable AND consumed?**
- New screens/modals/pages: where do users enter?
- New routes: registered in the router AND called from somewhere?
- New handlers: wired to a UI element that can trigger them?
- New Provider/Bloc/service: is there any consumer (`ref.watch`, `BlocBuilder`, injection site)?
- No entry path or no consumer → **orphaned**

### Step 5 — Classify

Apply three labels to every finding. **Use only the labels below.** If a stack-specific signal suggests another word, map it back to one of these six.

**Type**
- `dead` — empty handler or trivial-only action
- `stub` — TODO / FIXME / UnimplementedError, intentional placeholder
- `half-wired` — runs but has no feedback, no UI update, or no error handling
- `orphaned` — exists but has no entry path, no consumer, or no integration registration
- `temporary` — debug logs, hardcoded test values, leftover mocks
- `lifecycle-leak` — missing dispose / cleanup, duplicate listeners

**Severity**
- `critical` — user-facing action is broken; data loss possible
- `high` — user experience breaks (no feedback, infinite loading, silent failure)
- `medium` — works but incomplete (no error handling, mock still active)
- `low` — hygiene (debug logs, unused imports inside scope)

**Confidence**
- `high` — execution path traced end-to-end
- `medium` — likely, but one more check needed
- `low` — runtime registration suspected, holding back

### Step 6 — Report

Run the self-check below, then produce the report in this exact structure. Sort findings by severity, then confidence.

**Self-check before reporting**
- [ ] Did I confirm scope with the user (or skip because it was explicit)?
- [ ] Does every finding cite file:line?
- [ ] Did I drop confidence to low for any plausibly runtime-registered code?
- [ ] Are all findings classified using only the six canonical Types?
- [ ] Did I leave the codebase unmodified?

**Report format**

```
Audit Result

Scope: [explicit scope — e.g. PR #142, or last 5 commits on lib/features/auth/]
Inventory: [N handlers / N routes / N mutations / N DI registrations]

────────────────────────────────────────

CRITICAL [N]

1. lib/features/auth/login_screen.dart:47
   Type: dead
   Element: "Login button" onPressed
   Confidence: high
   Evidence: () => {} empty handler. No function called after form validation.
   Impact: Users cannot log in at all.
   Suggested fix: Call AuthBloc.add(LoginRequested(...))

2. ...

HIGH [N]
...

MEDIUM / LOW [N]
...

────────────────────────────────────────

NEEDS VERIFICATION (low confidence) [N]

Items below may be runtime-registered. Auto-classification withheld.
Confirm intent and they can be reclassified.

- lib/di/injection.dart:31 — UserRepository registered, no direct caller found
  (get_it-based DI; runtime injection plausible)

────────────────────────────────────────

RECOMMENDED NEXT STEPS

1. Resolve N Critical items first
2. ...

SAFE AUTO-FIX CANDIDATES

The following can be fixed safely on explicit request:
- Remove debug print statements (3)
- Remove unused imports inside scope (5)

Authorize with e.g. "fix Critical #1" or "remove all debug logs".
```

Every finding must cite **file:line**. Never report a vague directory.

## Anti-patterns (never do these)

- Modifying code without explicit authorization
- Reporting issues outside the stated scope
- Fixing things mid-audit — complete the full inventory first
- Skipping execution-path tracing because something "looks fine"
- Flagging runtime-registered code as dead with high confidence
- Inventing new Type labels (e.g. "disconnected", "incomplete") — always map to the canonical six
- Commenting on code style, naming, or formatting (out of scope)
- Commenting on test coverage (out of scope)
- Suggesting architectural rewrites unless asked

## Stack-specific signals

Each signal below maps to one of the six canonical Types. Use as starting hints, not exhaustive lists.

### Flutter / mobile

- Route declared but never pushed via `Navigator` / `go_router` / route table → **orphaned**
- Provider / Riverpod provider registered but no `ref.watch` / `ref.read` / `Consumer` → **orphaned** (but family / autoDispose may be runtime-resolved → drop confidence)
- Bloc / Cubit defined but no `BlocProvider` / `BlocBuilder` / `BlocListener` consuming it → **orphaned**
- MethodChannel handler registered but no native call site (grep both sides) → **orphaned**
- async function without `FutureBuilder` or loading state → **half-wired**
- Widget class defined but referenced in no `build()` → **orphaned**
- `StreamSubscription` / listener with no matching `dispose()` → **lifecycle-leak**
- `print` / `debugPrint` not gated by `kDebugMode` → **temporary**

### Web (React / Vue / Next)

- Route added but not registered in router config (`createBrowserRouter`, routes array) → **orphaned**
- Mutation called but no `invalidateQueries`, refetch, or optimistic update → **half-wired**
- Component exported but no import references it → **orphaned**
- New API endpoint not registered in the router/handler map → **orphaned**
- `useEffect` with subscriptions or timers but no cleanup return → **lifecycle-leak**
- Form submit handler that only logs / alerts → **dead**
- Optimistic update with no rollback path → **half-wired**

### Backend / API

- New route handler not mounted on the router → **orphaned**
- New DB migration not registered in the migration index → **orphaned**
- New service or repository not bound in DI container → **orphaned**
- External call with no timeout, retry, or circuit breaker → **half-wired** (missing error handling)
- Endpoint with TODO/UnimplementedError body → **stub**