---
title: "Restructuring a standalone plugin repo as a marketplace"
date: 2026-02-12
category: integration-issues
module: marketplace
tags: [marketplace, plugin, restructuring, claude-code]
symptoms:
  - plugin not discoverable via `/plugin marketplace add`
  - repo has plugin.json at root but no marketplace.json
severity: low
---

# Restructuring a standalone plugin repo as a marketplace

## Problem

A Claude Code plugin repo with `.claude-plugin/plugin.json` at root is a standalone plugin. To make it installable via `/plugin marketplace add`, it needs to be a **marketplace** — a repo with `marketplace.json` and plugins nested under `plugins/`.

## Solution

### Directory transformation

**Before:**
```
.claude-plugin/plugin.json
commands/workflows/pipeline.md
```

**After:**
```
.claude-plugin/marketplace.json
plugins/<plugin-name>/.claude-plugin/plugin.json
plugins/<plugin-name>/commands/workflows/pipeline.md
```

### Key steps

1. Create `plugins/<plugin-name>/` directory
2. Move `.claude-plugin/plugin.json` into `plugins/<plugin-name>/.claude-plugin/`
3. Move `commands/` into `plugins/<plugin-name>/`
4. Replace root `.claude-plugin/plugin.json` with `marketplace.json`
5. Keep `docs/`, `.gitignore`, `LICENSE` at root (they belong to the repo, not the plugin)

### marketplace.json structure

```json
{
  "name": "<repo-name>-marketplace",
  "owner": { "name": "Org", "url": "https://github.com/Org" },
  "metadata": { "description": "...", "version": "1.0.0" },
  "plugins": [
    {
      "name": "<plugin-name>",
      "description": "...",
      "version": "0.1.0",
      "author": { "name": "...", "url": "..." },
      "homepage": "https://github.com/...",
      "tags": [],
      "source": "./plugins/<plugin-name>"
    }
  ]
}
```

### Installation commands (for README)

```
/plugin marketplace add https://github.com/Org/repo
/plugin install <plugin-name>
```

## Git workflow gotchas

- **Pull before committing** when collaborating — remote may have diverged
- **Tags survive rebase** pointing at old commits. After `git pull --rebase`, re-tag:
  ```bash
  git tag -d v0.1.0
  git tag -a v0.1.0 -m "message"
  git push origin --delete v0.1.0
  git push origin v0.1.0
  ```

## Prevention

- Follow the `every-marketplace` pattern (hosts compound-engineering + coding-tutor) as the reference implementation
- Validate plugin.json fields against the real schema — don't invent fields like `prerequisites`
