# Pattern Library

Pattern files accumulate prevention rules from closed bugs. They are the long-term memory of the knowledge base.

## Pattern File Schema

```markdown
# Pattern: <category>

## What it is
<one paragraph>

## How to spot it
- <signal>

## Prevention rules
- <rule derived from BUG-NNN>  <!-- BUG-NNN -->

## Known instances
| ID | File:Line | Summary |
|----|-----------|---------|
```

## Categories and Stack Signals

### lifecycle-leak
StreamSubscription / listener with no dispose, useEffect with no cleanup, duplicate subscriptions.
- Flutter: `StreamSubscription` with no `cancel()` in `dispose()`
- Flutter: `print` / `debugPrint` not gated by `kDebugMode`
- Web: `useEffect` with subscriptions/timers but no cleanup return

### half-wired
Handler runs but no UI feedback, async with no loading/error state, mutation with no invalidation.
- Flutter: async function without `FutureBuilder` or loading state
- Web: mutation called but no `invalidateQueries`, refetch, or optimistic update
- Backend: external call with no timeout, retry, or circuit breaker

### orphaned
Route defined but never pushed, component imported nowhere, service registered but no consumer.
- Flutter: route declared but never pushed via `Navigator` / `go_router`
- Flutter: widget class referenced in no `build()`
- Web: component exported but no import references it
- Backend: new route handler not mounted on the router

### stub
TODO / FIXME / UnimplementedError, empty handler body, function returning hardcoded null.

### schema-mismatch
API response shape differs from model, DB column type mismatch, serialization error.

### pattern-violation
Project-specific convention broken (naming, layer boundary, data flow direction).
