---
name: port-find-skills
description: "Searches for and loads Port skills before any Port MCP tool call. Use at the start of every Port MCP session or task — before list_blueprints, list_entities, upsert_*, run_action, search_port_knowledge_sources, or any other Port MCP tool. Scans the load_skill catalog and org skill entities via MCP, then loads matches. Skip only when the user explicitly says to skip skill discovery or when only load_skill / skill-catalog discovery tools are needed."
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

Port skills encode org-specific workflows and platform knowledge. Before calling **any** Port MCP tool, find and load the relevant skill(s). Do not call Port MCP tools until skill discovery completes.

**Exception:** `load_skill` and skill-catalog discovery (`list_blueprints`, `list_entities` on the skill catalog blueprint) may run during discovery itself.

**Skip** when the user explicitly says to skip skill discovery.

## Workflow

```text
Task Progress:
- [ ] Step 1: Search the skill catalog
- [ ] Step 2: Match the task to available skills
- [ ] Step 3: Load relevant skills
- [ ] Step 4: Proceed with Port MCP tool calls
```

### Step 1 — Search the skill catalog

1. Confirm Port MCP is connected. Authenticate if needed.
2. Read the `load_skill` tool description — it lists available skills with trigger guidance. This is the primary catalog.
3. For org-specific skills not listed there:
   - `list_blueprints` (no identifiers) to find your organization's **skill catalog blueprint** (identifier or title suggests skills; properties like `description`, `instructions`, or MCP exposure flags are clues).
   - Exclude supporting blueprints (`*_file`, `*_version`, `*_group` suffixes) — those are not the catalog.
   - `list_blueprints({ identifiers: [...] })` for full schema when multiple candidates exist.
   - `list_entities` on the chosen catalog blueprint with `$identifier`, `$title`, and description-like properties from that blueprint's schema. Paginate while `hasMoreEntities` is true.

Merge both sources; either can surface a match the other misses.

### Step 2 — Match

Compare the user's request against skill identifiers, titles, and descriptions. Look for:

- Domain keywords (integration name, product area, blueprint type)
- Task verbs (create, troubleshoot, configure, scaffold, list, search)
- Overlapping scope (e.g. "dashboard widget" → dashboard or plugin skills)

Pick the **most specific** match(es). When unsure between two, load both only if they add non-overlapping guidance.

### Step 3 — Load matches

For each selected skill:

```json
load_skill({ name: "<skill-identifier>" })
```

Load referenced resources when the skill points to them:

```json
load_skill({ name: "<skill-identifier>", resource: "references/REFERENCE.md" })
```

Also check local project skills when relevant — read `.cursor/skills/<name>/SKILL.md` or `.claude/skills/<name>/SKILL.md` directly if they cover the task and aren't in Port yet.

### Step 4 — Proceed

One short line on which skills loaded and why, then call Port MCP tools. Loaded skill instructions override generic habits when they conflict.

## Tools reference

| Tool | Purpose |
|------|---------|
| `load_skill` | Primary skill catalog and loader |
| `list_blueprints` | Discover which blueprint stores skills in this org |
| `list_entities` | List skill entities from the org's skill catalog blueprint |
| `search_port_knowledge_sources` | Port product docs and platform knowledge |

## Rules

- Skill discovery is **silent and fast** — no long preamble; one line after loading is enough.
- Never assume the skill blueprint identifier — discover it via `list_blueprints` when entity lookup is needed.
- Never assume no skill exists without completing Step 1.
- Never call Port MCP tools before completing skill discovery (except discovery tools listed above).
- If Port MCP is unavailable, note the gap and check local `.cursor/skills/` or `.claude/skills/` before proceeding.
