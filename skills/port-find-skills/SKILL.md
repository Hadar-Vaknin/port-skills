---
name: port-find-skills
description: "Search for and load Port skills before calling any Port MCP tool. Use at the start of Port work — before list_blueprints, list_entities, upsert_*, run_action, search_port_knowledge_sources, or any other Port MCP tool. Check the load_skill catalog first; for org-specific skills not listed there, discover the org's skill catalog blueprint and list its entities. Skip only when the user explicitly says to skip skill discovery."
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

Skills provide step-by-step guidance for specialized Port tasks. Before calling any Port MCP tool, search for and load the relevant skill. Follow loaded skill instructions over generic habits.

`load_skill` and skill-catalog discovery (`list_blueprints`, then `list_entities` on the skill catalog blueprint) may run during this search. Skip only when the user explicitly says to skip skill discovery.

## 1. Search for the relevant skill

1. Check the `load_skill` tool description — it lists available skills with trigger guidance.
2. For org-specific skills not listed there, use `list_blueprints` to find your organization's skill catalog blueprint, then `list_entities` on that blueprint.

When listing blueprints, pick the catalog whose identifier or title indicates skills. Exclude supporting blueprints (`*_file`, `*_version`, `*_group`). If several candidates remain, fetch their schemas with `list_blueprints({ identifiers: [...] })`.

## 2. Load it

```json
load_skill({ name: "<skill-name>" })
```

Load referenced resources when the skill points to them:

```json
load_skill({ name: "<skill-name>", resource: "references/REFERENCE.md" })
```

Then call other Port MCP tools. Loaded skill instructions override generic habits when they conflict.
