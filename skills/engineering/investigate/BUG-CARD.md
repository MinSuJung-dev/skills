# Bug Card & Index Schemas

## .bugs/ Directory Structure

```
.bugs/
├── INDEX.md
├── bugs/
│   └── BUG-NNN-slug.md
└── patterns/
    ├── lifecycle-leak.md
    ├── half-wired.md
    ├── orphaned.md
    ├── stub.md
    ├── schema-mismatch.md
    └── pattern-violation.md
```

## INDEX.md

```markdown
# Bug Index

| ID | Title | Category | Severity | Status | Affected |
|----|-------|----------|----------|--------|---------|
| BUG-001 | Login button noop | stub | critical | verified | lib/auth/login_screen.dart:47 |
```

**Duplicate prevention:** Before creating a new record, check `Affected` for the same file:line.

## Bug Card Schema

```markdown
---
id: BUG-NNN
title:
category: lifecycle-leak | half-wired | orphaned | stub | schema-mismatch | pattern-violation
severity: critical | high | medium | low
status: investigating | reproduced | patched | verified | abandoned
tier: small | large
created:
last-touched:
affected: file:line
---

## Symptom
<observable facts only>

## Feedback loop
Type:
Command:
Latency:
Determinism: always | N/M | flaky

## Reproduction
Steps:
Environment:

## Expected vs Actual

## Scope

## Hypotheses (ranked)
- [ ] H1:
      Prediction:
      Cost: low|med|high   Prior: low|med|high

## Attempts
### Attempt 1 — <date>
Probe: file:line
Tag: [DEBUG-xxxx]
Result: pass | fail | unclear
Evidence:
Learned:

## Falsified

## Current best guess

## Post-mortem
Root cause:
Fix:
What would have prevented this:
Adjacent risk:
Pattern propagated: patterns/<category>.md | none
```
