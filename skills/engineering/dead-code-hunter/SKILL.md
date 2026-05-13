---
name: dead-code-hunter
description: Finds incomplete implementations in a codebase — empty handlers, stub functions, unconnected routes, TODO-blocked logic, and hardcoded fake data. Use when the user says "find what's not implemented", "check what's missing", "audit incomplete features", or "the UI is there but nothing works".
---

# Dead Code Hunter

Systematically locate every place where implementation is missing or faked. Report findings grouped by severity, then fix or hand off.

## What to look for

### 1. Empty or no-op event handlers

```ts
// Bad
onClick={() => {}}
onClick={() => console.log('clicked')}
onSubmit={handleSubmit} // where handleSubmit does nothing
```

Search patterns:
- Arrow functions with empty body `() => {}`
- Handlers that only call `console.log`, `console.warn`, or `alert`
- Handler functions whose body contains only a comment or a single `return`

### 2. Stub / not-implemented functions

```ts
function saveUser() {
  // TODO: implement
}

async function fetchDashboard() {
  throw new Error('Not implemented')
}

const getReport = () => null
```

Search patterns:
- Functions containing only `TODO`, `FIXME`, `HACK`, `XXX` comments
- Functions throwing `new Error('not implemented')` / `new Error('TODO')`
- Functions returning hardcoded `null`, `undefined`, `[]`, `{}` with no logic

### 3. Pages and components with no entry point

- Routes defined in a router that are never linked to from any `<Link>`, `<NavLink>`, `router.push()`, or `navigate()` call
- Components that exist but are never imported anywhere
- Menu items or nav entries pointing to `#` or `javascript:void(0)`

### 4. Hardcoded fake data masking missing API calls

```ts
const users = [{ id: 1, name: 'Test User' }] // never fetched
const revenue = 99999 // hardcoded placeholder
```

Search patterns:
- Variables named `mock`, `fake`, `dummy`, `placeholder`, `temp`, `test`
- Data arrays defined inline in component files that should come from an API
- `fetch`/`axios`/`useQuery` calls commented out next to hardcoded values

### 5. Disconnected form fields

- Form inputs with no `onChange`, no controlled value, and no name attribute for uncontrolled capture
- Submit handlers that don't read form values
- Validation logic that always passes (`return true`, empty validator)

### 6. Dead imports and unused exports

- Imported symbols never referenced in the file
- Exported functions/components never imported elsewhere in the project

---

## How to run the hunt

### Phase 1 — Automated scan

Run these searches across the codebase:

```bash
# Empty handlers
grep -rn "() => {}" src/
grep -rn "console\.log" src/ --include="*.ts" --include="*.tsx"

# Stubs
grep -rn "TODO\|FIXME\|not implemented\|throw new Error" src/

# Fake data signals
grep -rn "mock\|dummy\|placeholder\|fake" src/ -i --include="*.ts" --include="*.tsx"

# Disconnected routes: list all route paths, then check each appears in a Link/navigate call
```

Adapt the `src/` path and file extensions to the project's structure.

### Phase 2 — Route connectivity check

1. Extract every route path from the router config.
2. For each path, search for any navigation reference (`to=`, `href=`, `push(`, `navigate(`).
3. Flag routes with zero references — they are orphaned.

### Phase 3 — Form audit

For every `<form>` or form library usage (`useForm`, `Formik`, `react-hook-form`):
1. Confirm each field has a registered name and value binding.
2. Confirm the submit handler reads and uses those values.
3. Confirm validation is non-trivial.

---

## Output format

Report findings as a prioritised list:

```
## Dead Code Hunt Results

### 🔴 Critical — Broken user flows
- `src/pages/Checkout.tsx:42` — onClick does nothing; payment never initiated
- `src/api/orders.ts:18` — fetchOrders throws "not implemented"; order list always empty

### 🟡 High — Features exist in UI but are non-functional
- `src/components/FilterBar.tsx:67` — filter onChange handler is empty; filters have no effect
- Route `/settings/notifications` is defined but unreachable (no links)

### 🟠 Medium — Fake data / placeholders
- `src/pages/Dashboard.tsx:12` — revenue hardcoded to 99999; no API call
- `src/data/users.ts` — user list is a static mock array

### ⚪ Low — Cleanup items
- `src/utils/validate.ts:5` — validator always returns true
- `src/components/Avatar.tsx` — imported in 0 files
```

---

## Fix protocol

For each critical or high finding:

1. **Identify the intended behaviour** — check designs, PRD, or ask the user if unclear.
2. **Implement or scaffold** — wire the handler to a real action; connect the API; add navigation.
3. **If implementation scope is large** — replace the stub with a clearly-named placeholder that throws a visible error in dev, and file an issue. Do not leave silent no-ops.
4. **Delete** unused imports and unreachable dead components once confirmed safe.

Do not silently leave `console.log` handlers in place. A visible `alert('Not yet implemented')` in dev is better than silence — at least it signals to the user that something is missing.
