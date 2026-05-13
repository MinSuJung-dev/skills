# Skills

AI-agnostic coding agent skills. Works with Claude Code, Codex, Cursor, and any agent that can read markdown.

## Install (Claude Code)

```bash
claude plugin install github:MinSuJung-dev/skills
```

## Skills

### Engineering

| Skill | Description |
|-------|-------------|
| [integration-audit](./skills/engineering/integration-audit/SKILL.md) | Audit whether a recently implemented feature is actually complete — finds dead handlers, stubs, orphaned routes, half-wired mutations, and lifecycle leaks |
| [investigate](./skills/engineering/investigate/SKILL.md) | Disciplined bug investigation loop — build a feedback signal, reproduce, hypothesize falsifiably, instrument, fix, and persist findings across sessions |

## Usage (other AI agents)

For agents without a plugin system (Codex, Cursor Rules, etc.), paste the contents of the relevant `SKILL.md` directly into your system prompt or rules file.

## Contributing

Each skill lives in `skills/<category>/<skill-name>/SKILL.md` with a YAML frontmatter block:

```markdown
---
name: skill-name
description: When to trigger this skill (used by AI agents to decide relevance)
---

# Skill content...
```
