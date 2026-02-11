---
title: "feat: Add /workflows:pipeline end-to-end automation command"
type: feat
date: 2026-02-11
brainstorm: docs/brainstorms/2026-02-11-workflows-pipeline-brainstorm.md
---

## Enhancement Summary

**Deepened on:** 2026-02-11
**Sections enhanced:** 6
**Research agents used:** orchestrating-swarms skill, agent-native-architecture skill, create-agent-skills skill, architecture-strategist, code-simplicity-reviewer, agent-native-reviewer, best-practices-researcher, pattern-recognition-specialist

### Key Improvements
1. **Thin orchestrator pattern**: Collapsed from ~100 lines to ~25, following the /lfg precedent -- commands are prompts, not programs
2. **One Task per milestone**: Pipeline command owns the milestone loop, spawning a fresh agent per milestone to prevent context exhaustion
3. **Single unified implementation prompt**: Eliminated duplicated simple/roadmap prompts (52% duplication ratio) with one outcome-oriented prompt
4. **Agent-drivable design**: Added `$ARGUMENTS` support and sensible defaults so agents can invoke the pipeline without interactive gates

### New Considerations Discovered
- The "deepen plan?" checkpoint is redundant -- `/workflows:plan` already offers this
- Swarm mode within milestones should be deferred to a future iteration
- A pipeline state file enables checkpoint/resume for long-running roadmaps
- Outcome-oriented prompts with judgment criteria outperform step-by-step choreography

### Conflicting Advice (for user review)
- **create-agent-skills vs agent-native**: create-agent-skills recommends removing auto-detection entirely and letting the agent infer from plan content; agent-native recommends keeping complexity assessment but having the pipeline command (not the agent) manage the loop. Resolution below: keep lightweight detection, pipeline manages loop.

---

# feat: Add /workflows:pipeline end-to-end automation command

## Overview

Create a single Claude Code command that automates the full development lifecycle: brainstorm, plan, deepen, review, implement, PR, handle reviews, compound learnings. The command orchestrates existing plugin commands with natural interaction points during planning and full autonomy during implementation.

## Problem Statement / Motivation

The current workflow requires manually invoking 6-8 commands in sequence across multiple sessions, remembering file paths between steps, and crafting detailed prompts for the implementation phase. This is repetitive and error-prone. A single `/workflows:pipeline` command should orchestrate the entire sequence, preserving the interactive brainstorm/planning experience while automating the implementation and review cycle.

## Proposed Solution

A single command file at `.claude/commands/workflows/pipeline.md` that:

1. Chains existing commands sequentially (brainstorm, plan, deepen, review)
2. Spawns a fresh Task agent per unit of work for autonomous implementation (clean context)
3. For roadmaps, the pipeline command itself loops over milestones, spawning one fresh agent per milestone

### Research Insights

**Thin Orchestrator Pattern (from architecture-strategist, code-simplicity-reviewer, pattern-recognition-specialist):**
- `/lfg` is 19 lines. `/slfg` is 31 lines. `/plan_review` is 3 lines of content. The pipeline should be ~25 lines -- a flat command list, not a logic-heavy program
- Commands are prompts, not programs. Conditional logic belongs in subcommands, not the orchestrator
- The existing commands already handle their own interactivity: `/workflows:brainstorm` uses `AskUserQuestion`, `/workflows:plan` offers deepening as a post-generation option

**Composition over Re-implementation (from architecture-strategist, pattern-recognition-specialist):**
- The pipeline's Phase 4 in the original plan re-specifies branch creation, PR creation, and CI monitoring that `/workflows:work` already handles
- If `/workflows:work` evolves, inline instructions will drift. Delegate instead of duplicating

## Technical Approach

### Command File Structure

```
.claude/commands/workflows/pipeline.md
```

### `.claude/commands/workflows/pipeline.md`

```markdown
---
name: workflows:pipeline
description: End-to-end development pipeline from brainstorm to merged PR with compounded learnings
argument-hint: "[feature idea or problem to explore]"
---

Run these steps in order. Do not do anything else.

1. `/ralph-wiggum:ralph-loop "finish all pipeline steps" --completion-promise "PIPELINE_DONE"`

## Phase 1: Brainstorm & Plan (Interactive)

2. `/workflows:brainstorm $ARGUMENTS`
3. `/workflows:plan` (auto-detects brainstorm from docs/brainstorms/)
4. `/compound-engineering:deepen-plan`
5. `/compound-engineering:plan_review` on the plan file
6. Present review findings. Ask: "Apply review suggestions? (Yes/No)"
   - Yes → apply changes to the plan
   - No → proceed as-is

## Phase 2: Implement (Autonomous -- fresh Task agent per unit of work)

7. Read the plan. If it has milestones, process each sequentially. For each
   milestone (or the whole plan if no milestones), spawn a fresh Task agent:

   "Implement the feature described in $PLAN_PATH (brainstorm: $BRAINSTORM_PATH).
   Your goal is a PR that is reviewed, CI-passing, and ready for merge.

   Run `/workflows:work` to implement the plan on a feature branch and open a PR.

   Then request additional external reviews on the PR:
   - Comment `@codex review`
   - Comment `/gemini review`

   When reviews come in, triage them. Address comments that improve correctness,
   security, or clarity. Explain your rationale when you disagree.

   Run `/workflows:review` for internal code review. Address any P1/P2 findings.

   Monitor CI with `gh pr checks --watch`. If CI fails, reproduce locally first.

   When done, run `/workflows:compound` and commit learnings to the branch.
   Notify me the PR is ready for merge with the PR URL."

8. For milestone plans: ask me "Milestone N complete. PR ready at $URL.
   Merge when ready and let me know to continue." Wait for my confirmation
   before spawning the next milestone's agent.

## Phase 3: Done

9. Report completion with PR URL(s).
10. Output `<promise>PIPELINE_DONE</promise>`

Start with step 1 now.
```

### Research Insights

**Outcome-Oriented Prompts (from agent-native-architecture):**
- The original plan used step-by-step numbered instructions (choreography). The improved prompt describes the desired outcome with judgment criteria, letting the agent decide how to achieve it
- "Address comments that improve correctness, security, or clarity" gives judgment criteria vs "Triage them, address the relevant ones" which is vague
- `gh pr checks --watch` is the right primitive for CI monitoring (single blocking command vs manual polling loop)

**One Task Per Milestone (from agent-native-architecture, architecture-strategist, agent-native-reviewer):**
- The original plan sent one Task agent to process ALL milestones. For a roadmap with 5+ milestones, this single agent will exhaust its context window
- Each milestone involves: reading a plan, implementing code, creating a PR, polling reviews, triaging comments, debugging CI, and compounding -- enormous context per milestone
- The pipeline command itself should manage the milestone loop, spawning one fresh Task agent per milestone. Between milestones, the pipeline waits for the user merge, then spawns the next agent

**$ARGUMENTS Support (from agent-native-reviewer, create-agent-skills):**
- `/lfg` and `/slfg` accept `$ARGUMENTS` and pass down. The pipeline does the same, passing to `/workflows:brainstorm`

**Dynamic Context Injection (from agent-native-architecture):**
- Before spawning the Task agent, gather and pass: current git branch, repo structure summary, available slash commands
- Include the plan content (or summary) in the prompt -- not just the path -- to avoid burning context on file reads

**Deferred Swarm Mode (from agent-native-architecture, code-simplicity-reviewer):**
- The original plan auto-detected roadmap vs simple and switched between single-agent and swarm mode. Multiple agents recommend deferring swarm mode
- Start with single-agent mode for all plans. Each milestone gets one dedicated Task agent. Add swarm when the need emerges from real usage
- The user already has `/slfg` for swarm when they want it explicitly

**Filesystem as State (from best-practices-researcher, create-agent-skills):**
- Commands do not pass state explicitly. Each command writes artifacts to well-known locations (`docs/brainstorms/`, `docs/plans/`), and subsequent commands discover them via filesystem conventions and conversation context
- `/workflows:plan` already auto-detects brainstorms from `docs/brainstorms/`. No explicit path-passing needed

## Acceptance Criteria

- [ ] Command file exists at `.claude/commands/workflows/pipeline.md`
- [ ] Running `/workflows:pipeline` starts an interactive brainstorm
- [ ] Running `/workflows:pipeline "feature idea"` seeds the brainstorm with the description
- [ ] After brainstorm, automatically transitions to planning
- [ ] Plan is automatically deepened (no separate checkpoint -- follows /lfg pattern)
- [ ] Plan review runs and user can approve/reject suggestions (Yes/No)
- [ ] Implementation spawns as a fresh Task agent (clean context)
- [ ] Single implementation prompt handles both simple and milestone plans
- [ ] For milestone plans, one fresh Task agent per milestone (pipeline manages loop)
- [ ] PR review requests include Copilot, Codex (`@codex review`), and Gemini (`/gemini review`)
- [ ] Agent waits for user merge between roadmap milestones
- [ ] Learnings are compounded and committed to feature branch after each implementation
- [ ] Command is under 40 lines (thin orchestrator pattern)

### Research Insights

**Removed Acceptance Criteria (from code-simplicity-reviewer):**
- "Complexity auto-detection correctly identifies simple vs roadmap plans" -- removed because auto-detection is deferred; the agent reads the plan and acts accordingly
- "Roadmap plans with 3+ milestones use swarm mode for implementation" -- removed because swarm mode is deferred to future iteration
- "User is prompted whether to deepen the plan" -- removed because deepening runs automatically (matching `/lfg` pattern)

**Added Acceptance Criteria (from agent-native-reviewer, pattern-recognition-specialist):**
- "`$ARGUMENTS` support" -- makes the command agent-drivable
- "Under 40 lines" -- enforces the thin orchestrator pattern
- "Single implementation prompt" -- prevents prompt duplication

## Dependencies & Risks

**Dependencies:**
- All referenced commands must exist: `/workflows:brainstorm`, `/workflows:plan`, `/compound-engineering:deepen-plan`, `/compound-engineering:plan_review`, `/workflows:compound`
- `gh` CLI must be installed and authenticated for PR operations
- GitHub Copilot, Codex, and Gemini review integrations must be configured on the repository

**Risks:**
- Long-running pipeline: for roadmaps with many milestones, the pipeline session itself runs for hours waiting for user merges between milestones
- Merge conflicts between milestones if they touch overlapping files
- Review polling: external reviewers (Copilot, Codex, Gemini) may be slow or unavailable

### Research Insights

**Mitigations (from agent-native-architecture, architecture-strategist, best-practices-researcher):**

**Context exhaustion mitigated:**
- One Task per milestone means each implementation agent starts fresh (bounded context)
- The pipeline command itself only tracks high-level state (milestone names, PR URLs) -- low context growth
- If the pipeline's own context fills, user can resume by re-running with `$ARGUMENTS` pointing to the plan file (skips brainstorm/plan phases)

**Merge conflicts mitigated:**
- Between milestones (after user merge), each new Task agent starts from updated main. Add `git pull origin main` before starting next milestone
- Each milestone agent naturally starts from fresh state on main

**Review timeout:**
- The agent should use judgment, not rigid polling: check every 2 min initially, then every 5 min, proceed after 30 min with whatever reviews are available
- `gh pr checks --watch` handles CI monitoring without manual polling

**Pipeline state file (future improvement from agent-native-reviewer):**
- Write state to `docs/pipelines/YYYY-MM-DD-<topic>-pipeline.md` after each phase transition
- Enables resume-from-checkpoint for interrupted pipelines
- Deferred to future iteration to keep initial implementation simple

**Completion contract (future improvement from architecture-strategist, pattern-recognition-specialist):**
- `/lfg` and `/slfg` use `/ralph-wiggum:ralph-loop` with `<promise>DONE</promise>` for verifiable completion
- The pipeline could adopt this pattern for composability into larger workflows
- Deferred to future iteration

**Auto-merge (future improvement from agent-native-reviewer, brainstorm):**
- Add merge strategies: `wait` (default), `auto-merge-on-green` (`gh pr merge --auto --squash`)
- Already noted in brainstorm as future improvement

## References & Research

### Internal References
- `/lfg` command pattern: `~/.claude/plugins/cache/every-marketplace/compound-engineering/2.30.0/commands/lfg.md`
- `/slfg` command pattern: `~/.claude/plugins/cache/every-marketplace/compound-engineering/2.30.0/commands/slfg.md`
- Workflow commands: `~/.claude/plugins/cache/every-marketplace/compound-engineering/2.30.0/commands/workflows/`
- Brainstorm: `docs/brainstorms/2026-02-11-workflows-pipeline-brainstorm.md`

### External Research (from best-practices-researcher)
- [Temporal.io](https://temporal.io/) -- durable workflow orchestration patterns: checkpoint/resume, human-in-the-loop approval gates via signals
- [LangGraph](https://docs.langchain.com/oss/python/langgraph/) -- three durability modes (exit, async, sync) for agent state persistence
- [Claude Code Hooks](https://code.claude.com/docs/en/hooks-guide) -- `SessionStart` hook with `compact` matcher for context recovery; `PreToolUse` for approval gates
- [Command Line Interface Guidelines](https://clig.dev/) -- composability principles, anti-patterns for CLI workflow design
- [Taskfile.dev](https://taskfile.dev/) -- checksum-based dependency tracking for task runners (Taskfile, Just, Make comparison)
