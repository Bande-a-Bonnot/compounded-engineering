---
title: "feat: Create public compounded-engineering plugin repo"
type: feat
date: 2026-02-11
---

## Enhancement Summary

**Deepened on:** 2026-02-11
**Sections enhanced:** 6
**Research agents used:** agent-native-architecture, create-agent-skills, architecture-strategist, code-simplicity-reviewer, agent-native-reviewer, plugin-readme-researcher, ankane-readme-researcher

### Key Improvements
1. Hybrid Ankane-style README: concise, ASCII flow, scannable tables
2. `prerequisites` field in plugin.json for forward-compatible dependency declaration
3. Interaction contract table in README for agent callers
4. Removed CHANGELOG.md and permissions section (YAGNI)
5. Added `completion-promise` and `requires-plugins` to command frontmatter

### New Considerations Discovered
- `--auto` mode for agent-to-agent composition (deferred to future iteration)
- Pipeline state file for resumability (deferred)
- Failure contract (minimal: don't output promise if failed)

# Create Public compounded-engineering Plugin Repo

## Overview

Create a dedicated **public** repository at `github.com/Bande-a-Bonnot/compounded-engineering` that packages the `workflows:pipeline` command as a proper Claude Code plugin, with a comprehensive README.md. This makes the pipeline command installable by anyone using Claude Code.

## Problem Statement / Motivation

The `workflows:pipeline` command currently lives as a project-local custom command at `.claude/commands/workflows/pipeline.md`. It cannot be shared, installed, or reused across projects. Publishing it as a Claude Code plugin in a public repo makes it:

- Installable by any Claude Code user
- Discoverable via GitHub
- Forkable and customizable
- A reference implementation for thin orchestrator plugins

## Proposed Solution

### 1. Restructure as a Claude Code plugin

Transform the project from a "project using plugins" to a "distributable plugin":

```
compounded-engineering/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── commands/
│   └── workflows/
│       └── pipeline.md          # The pipeline command (moved from .claude/)
├── docs/
│   ├── brainstorms/
│   │   └── 2026-02-11-workflows-pipeline-brainstorm.md
│   └── plans/
│       └── 2026-02-11-feat-workflows-pipeline-command-plan.md
│       └── 2026-02-11-feat-public-plugin-repo-plan.md
├── .gitignore
├── LICENSE                      # MIT
└── README.md                    # Comprehensive documentation
```

**Research Insight (simplicity reviewer):** Removed CHANGELOG.md — no change history exists at v0.1.0. Add it when there's a second version.

**Key structural changes:**
- Move `pipeline.md` from `.claude/commands/workflows/` to `commands/workflows/` (plugin convention)
- Create `.claude-plugin/plugin.json` manifest
- Do NOT publish `.claude/settings.local.json` or `.claude/ralph-loop.local.md`

### 2. Plugin manifest (`.claude-plugin/plugin.json`)

```json
{
  "name": "compounded-engineering",
  "version": "0.1.0",
  "description": "End-to-end development pipeline: brainstorm → plan → deepen → review → implement → compound. One command to rule them all.",
  "author": {
    "name": "Bande-a-Bonnot",
    "url": "https://github.com/Bande-a-Bonnot"
  },
  "repository": "https://github.com/Bande-a-Bonnot/compounded-engineering",
  "license": "MIT",
  "keywords": [
    "pipeline",
    "workflow-automation",
    "compound-engineering",
    "orchestrator"
  ],
  "prerequisites": [
    "compound-engineering (>=2.30.0) - https://github.com/EveryInc/compound-engineering-plugin",
    "ralph-loop (optional) - official Claude plugin"
  ]
}
```

### 3. Update pipeline.md command references

Fully-qualify all external command references to avoid namespace ambiguity:

| Current (bare)                           | Updated (fully-qualified)                                 |
|------------------------------------------|-----------------------------------------------------------|
| `/ralph-wiggum:ralph-loop`               | `/ralph-loop:ralph-loop`                                  |
| `/workflows:brainstorm $ARGUMENTS`       | `/compound-engineering:workflows:brainstorm $ARGUMENTS`   |
| `/workflows:plan`                        | `/compound-engineering:workflows:plan`                    |
| `/compound-engineering:deepen-plan`      | `/compound-engineering:deepen-plan` (already qualified)   |
| `/compound-engineering:plan_review`      | `/compound-engineering:plan_review` (already qualified)   |
| `/workflows:work`                        | `/compound-engineering:workflows:work`                    |
| `/workflows:review`                      | `/compound-engineering:workflows:review`                  |
| `/workflows:compound`                    | `/compound-engineering:workflows:compound`                |

### 4. Comprehensive README.md (Hybrid Ankane Style)

**Research Insight (ankane-readme-researcher + plugin-readme-researcher):** Use Ankane structural discipline (imperative voice, ≤15 words per sentence, minimal prose) with plugin-specific additions (prerequisites, flow visualization, interaction contract). ASCII flow diagram over Mermaid (renders everywhere). Every section under 10 lines.

Draft outline:

```markdown
# compounded-engineering

End-to-end development pipeline for Claude Code. One command from idea to merged PR.

## How It Works

  brainstorm → plan → deepen → review → implement → PR → compound
  |________ interactive ________|  |_______ autonomous _______|

## Components

| Component | Count |
|-----------|-------|
| Commands  | 1     |

| Command | Description |
|---------|-------------|
| `/workflows:pipeline` | Full development lifecycle orchestrator |

## Prerequisites

- [compound-engineering](https://github.com/EveryInc/compound-engineering-plugin) plugin (required)
- [ralph-loop](https://github.com/anthropics/claude-plugins-official) plugin (optional)

Install dependencies:

  claude /plugin install compound-engineering

## Installation

  git clone https://github.com/Bande-a-Bonnot/compounded-engineering.git
  claude --plugin-dir ./compounded-engineering

## Usage

Run the full pipeline:

  /workflows:pipeline

Seed with a feature idea:

  /workflows:pipeline "add rate limiting to the API"

### What Happens

Phase 1 (interactive):
  1. Brainstorm — explore the idea with you
  2. Plan — generate an implementation plan
  3. Deepen — enrich with parallel research
  4. Review — multi-agent plan review; approve or reject

Phase 2 (autonomous):
  5. Implement — fresh Task agent per milestone
  6. Open PR, request reviews (Codex, Gemini)
  7. Address feedback, monitor CI
  8. Compound learnings to branch

Phase 3:
  9. Report PR URL(s). You merge.

For milestone plans, one agent per milestone. Pipeline waits for
your merge between each.

### Interaction Points

| Step | Interactive? | Condition |
|------|-------------|-----------|
| Brainstorm | Yes (1-5 questions) | Always |
| Plan | Yes (0-3 questions) | If ambiguity exists |
| Deepen/Review | No | Autonomous |
| Apply suggestions | Yes (1 question) | Always |
| Implement | No | Autonomous per milestone |
| Merge approval | Yes (1 per milestone) | Multi-milestone plans only |

## For Agent Callers

Individual phases are available via compound-engineering:
- `/compound-engineering:workflows:brainstorm`
- `/compound-engineering:workflows:plan`
- `/compound-engineering:workflows:work`
- `/compound-engineering:workflows:review`
- `/compound-engineering:workflows:compound`

Use the pipeline for end-to-end. Use individual commands for targeted work.

## Customization

Fork and edit `commands/workflows/pipeline.md`. Common changes:

- Remove review bot comments (lines with @codex, /gemini)
- Remove ralph-loop steps (lines 1 and 10)
- Add your own review commands

## License

MIT
```

**Sections removed (simplicity reviewer):**
- Permissions section — not user-actionable
- Namespace collision discussion — internal concern
- Detailed per-phase descriptions — replaced by 9-line numbered list

**Sections added (agent-native reviewer):**
- Interaction Points table — documents the agent-drivability contract
- For Agent Callers — documents primitive commands for partial invocation

### 5. Create public GitHub repo

```bash
gh repo create Bande-a-Bonnot/compounded-engineering \
  --public \
  --description "End-to-end Claude Code development pipeline: brainstorm → plan → implement → compound" \
  --clone
```

### 6. Push initial commit

Initialize git, add all plugin files, push to origin.

## Technical Considerations

- **Namespace collision**: Both `compound-engineering` and `compounded-engineering` register commands. Using fully-qualified references in `pipeline.md` avoids ambiguity. Users invoke as `/compounded-engineering:workflows:pipeline` or just `/workflows:pipeline` if no collision.
- **Dependency chain**: Pipeline depends on `compound-engineering` + `ralph-loop`. No programmatic dependency declaration exists in plugin.json — must be documented in README.
- **ralph-loop optional**: Consider documenting that steps 1 and 10 (ralph-loop) are optional. Without them the pipeline still works, just without loop persistence.
- **Design docs inclusion**: Include `docs/brainstorms/` and `docs/plans/` for transparency — they document the design rationale with 8-agent deepening and 3-reviewer review.

### 4b. Update pipeline.md command frontmatter

**Research Insight (agent-native reviewer):** Add `completion-promise` and `requires-plugins` to frontmatter for agent discoverability:

```yaml
---
name: workflows:pipeline
description: End-to-end development pipeline from brainstorm to merged PR with compounded learnings
argument-hint: "[feature idea or problem to explore]"
completion-promise: "PIPELINE_DONE"
requires-plugins: ["compound-engineering", "ralph-loop"]
---
```

## Acceptance Criteria

- [ ] Public repo exists at `github.com/Bande-a-Bonnot/compounded-engineering`
- [ ] `.claude-plugin/plugin.json` with valid metadata and `prerequisites` field
- [ ] `commands/workflows/pipeline.md` at correct plugin path with enriched frontmatter
- [ ] All external command references fully-qualified in pipeline.md
- [ ] README.md follows hybrid Ankane style with ASCII flow, interaction contract table
- [ ] MIT LICENSE file
- [ ] `.gitignore` excludes local state files (`.claude/`, `*.local.md`, `.DS_Store`)
- [ ] No personal paths or secrets in committed files
- [ ] Design docs included in `docs/` for transparency

## Dependencies & Risks

**Dependencies:**
- `gh` CLI authenticated to Bande-a-Bonnot org with repo creation permission
- compound-engineering plugin (v2.30.0+) — required at runtime
- ralph-loop plugin — optional at runtime

**Risks:**
- Naming similarity with `compound-engineering` may confuse users → mitigated by clear README
- No plugin dependency declaration mechanism → mitigated by Prerequisites section
- Users without all prerequisites will fail at runtime → mitigated by clear error documentation

## References & Research

### Internal References
- Pipeline command: `.claude/commands/workflows/pipeline.md`
- Pipeline brainstorm: `docs/brainstorms/2026-02-11-workflows-pipeline-brainstorm.md`
- Pipeline plan: `docs/plans/2026-02-11-feat-workflows-pipeline-command-plan.md`

### External References
- compound-engineering plugin structure: `~/.claude/plugins/cache/every-marketplace/compound-engineering/2.30.0/`
- Plugin manifest reference: `.claude-plugin/plugin.json`
- Institutional learnings on plugin loading: infinidash `docs/solutions/integration-issues/`

### Deepening Research (2026-02-11)
- **Agent-native architecture skill**: Recommends outcome-oriented prompt over fixed sequence. Deferred for v0.1.0 — the pipeline follows the established /lfg pattern (sequential commands) which is proven and simple.
- **Agent-native reviewer**: 5/9 capabilities fully agent-accessible, 2/9 partial, 2/9 blocked. Recommends `--auto` mode for agent-to-agent composition. Deferred to v0.2.0.
- **Code-simplicity reviewer**: Cut CHANGELOG.md (no history at v0.1.0), permissions section (not actionable), namespace collision docs (internal concern).
- **Ankane README researcher**: Hybrid Ankane style — imperative voice, ≤15 words/sentence, ASCII flow diagram, 6-7 sections each under 10 lines. No badges, no Mermaid.
- **Plugin README researcher**: Tables over prose, Quick Start section, progressive disclosure, components inventory even for single-command plugins. Known Issues section if needed.
- **Architecture strategist**: Plugin structure matches compound-engineering conventions. CLAUDE.md versioning requirements apply.

### Deferred Improvements (v0.2.0+)
- `--auto` mode for agent-to-agent composition (bypasses interactive checkpoints)
- Pipeline state file for checkpoint/resume
- Failure contract with `<promise>PIPELINE_FAILED</promise>`
- Auto-merge strategy (`gh pr merge --auto --squash`)
