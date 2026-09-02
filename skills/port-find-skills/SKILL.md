---
name: port-find-skills
description: "Before any Port MCP tool call, find and load skills that match the user's prompt. Use whenever Port MCP will be used. Always inspect the load_skill tool description AND list the org skill catalog via list_blueprints + list_entities, then load matching skills with load_skill. Do not skip catalog lookup. Skip only when the user explicitly says to skip skill discovery."
license: MIT
compatibility: "Claude Code, Cursor, Codex CLI, GitHub Copilot, VS Code"
metadata:
  version: "1.0.0"
  author: port-labs
  repository: https://github.com/port-labs/port-skills
  tags: port,skills,discovery,mcp,onboarding
  summary: Find and load Port skills before any MCP tool call
---

# Find skills

Before any Port MCP tool call, find skills that match the user's prompt and load them with `load_skill`. Follow loaded skill instructions over generic habits.

The only Port MCP tools allowed before that load are the ones in this skill: reading the `load_skill` description, `list_blueprints`, `list_entities` on the skill catalog blueprint, and `load_skill` itself.

## Always do both

1. **`load_skill` tool description** — it lists available skills with trigger guidance. Match those against the prompt.
2. **Skill catalog** — `list_blueprints` to find the organization's skill catalog blueprint, then `list_entities` on that blueprint. Match those against the prompt too.

Do not treat the catalog as a fallback. Skills can exist in either place; miss one and you may skip a skill the prompt needs.

The catalog blueprint varies by org. Choose the blueprint whose identifier or title indicates skills. Skip supporting blueprints (`*_file`, `*_version`, `*_group`).

## Load matches

For each skill that fits the prompt:

```json
load_skill({ name: "<skill-name>" })
```

Then make other Port MCP tool calls.
