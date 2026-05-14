# Stack-Specific Detection Signals

Use these signals during Step 4 (Trace) to identify issues specific to each stack.
Map every finding back to one of the six canonical types: `dead` `stub` `half-wired` `orphaned` `temporary` `lifecycle-leak`

## Flutter / Mobile

| Signal | Type |
|--------|------|
| Route declared but never pushed via `Navigator` / `go_router` | orphaned |
| Provider/Riverpod registered but no `ref.watch`/`ref.read`/`Consumer` | orphaned (drop confidence if `family`/`autoDispose`) |
| Bloc/Cubit with no `BlocProvider`/`BlocBuilder`/`BlocListener` | orphaned |
| MethodChannel handler with no native call site (grep both sides) | orphaned |
| async function without `FutureBuilder` or loading state | half-wired |
| Widget class referenced in no `build()` | orphaned |
| `StreamSubscription` / listener with no matching `dispose()` | lifecycle-leak |
| `print` / `debugPrint` not gated by `kDebugMode` | temporary |

## Web (React / Vue / Next)

| Signal | Type |
|--------|------|
| Route added but not in router config (`createBrowserRouter`, routes array) | orphaned |
| Mutation with no `invalidateQueries`, refetch, or optimistic update | half-wired |
| Component exported but no import references it | orphaned |
| New API endpoint not registered in the router/handler map | orphaned |
| `useEffect` with subscriptions/timers but no cleanup return | lifecycle-leak |

## Backend / API

| Signal | Type |
|--------|------|
| New route handler not mounted on the router | orphaned |
| New DB migration not in the migration index | orphaned |
| New service/repository not bound in DI container | orphaned |
| External call with no timeout, retry, or circuit breaker | half-wired |
