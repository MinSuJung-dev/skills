# Skills

AI-agnostic coding agent skills. Works with Claude Code, Codex, Cursor, and any agent that can read markdown.

## Install (Claude Code)

```bash
claude plugin install github:MinSuJung-dev/skills
```

## Skills

Coming soon.

## Contributing

Each skill lives in `skills/<category>/<skill-name>/SKILL.md` with a YAML frontmatter block:

```markdown
---
name: skill-name
description: When to trigger this skill (used by AI agents to decide relevance)
---

# Skill content...
```
