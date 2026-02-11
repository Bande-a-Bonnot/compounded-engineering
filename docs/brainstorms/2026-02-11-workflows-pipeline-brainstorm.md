# Brainstorm: /workflows:pipeline Command

**Date:** 2026-02-11
**Status:** Ready for planning

## What We're Building

A single Claude Code command (`/workflows:pipeline`) that automates the full development lifecycle from brainstorm to merged PR with compounded learnings. It chains existing compound-engineering plugin commands into a coherent pipeline with natural interaction points during planning and full autonomy during implementation.

The command takes no arguments. It starts with an interactive brainstorm, flows through planning with user checkpoints, then spawns a fresh agent for autonomous implementation.

## Why This Approach

- **Single command file** following the exact pattern used by `/lfg` and `/slfg` (a `commands/workflows/pipeline.md` markdown file with YAML frontmatter)
- **Fresh agent for implementation** mimics the user's current session-clear workflow, keeping context clean between planning and execution
- **Auto-detection of complexity** determines whether to use single-agent (`/lfg` pattern) or swarm mode (`/slfg` pattern) based on plan structure (roadmap vs simple plan)
- **Hardcoded review requests** (Copilot, Codex, Gemini) -- can be edited later if needs change

## Key Decisions

1. **Command format**: Single `pipeline.md` file in `commands/workflows/`, no skill needed
2. **No arguments**: Brainstorm starts interactively, feature description emerges from dialogue
3. **Interactive phases**: Brainstorm is interactive; deepening plan is user's choice; plan review suggestions require user approval
4. **Autonomous phases**: Implementation, PR creation, review handling, CI monitoring, and compounding run without user intervention
5. **Session separation**: Spawn fresh Task agent for implementation to get clean context (mimics current session-clear workflow)
6. **Agent mode auto-detection**: Roadmap plans or plans with 3+ milestones use swarm mode (/slfg pattern); simpler plans use single-agent (/lfg pattern)
7. **Milestone handling**: For roadmap plans, process all milestones sequentially in one session. Each milestone: implement → PR → address reviews → update plan to mark completion → wait for user merge → proceed to next
8. **PR merge**: Wait for user to merge each PR. Notify when reviews are addressed and CI passes. (Future improvement: auto-merge when CI passes)
9. **Review requests**: Hardcoded -- assign PR to user, request Copilot review, Codex review (`@codex review` comment), Gemini review (`/gemini review` comment)
10. **Compound learnings**: After each milestone/feature implementation, run `/workflows:compound` and commit learnings to the feature branch

## Pipeline Phases

### Phase 1: Brainstorm (Interactive)
- Run `/workflows:brainstorm` inline
- User collaborates on what to build
- Produces brainstorm document at `docs/brainstorms/`

### Phase 2: Plan (Interactive with checkpoints)
- Run `/workflows:plan` using the brainstorm
- Ask user: "Deepen the plan?" → If yes, run `/compound-engineering:deepen-plan`
- Run `/compound-engineering:plan_review`
- Present review suggestions, user approves/rejects

### Phase 3: Assess Complexity (Automatic)
- Parse the plan to determine: roadmap (multiple milestones) vs simple implementation plan
- If roadmap with 3+ milestones → swarm mode
- If simple plan → single-agent mode

### Phase 4: Implement (Autonomous, fresh agent)
- Spawn a fresh Task agent with clean context
- Pass: plan path, brainstorm path, agent mode decision
- Agent creates feature branch off main
- For simple plans: implement the full plan
- For roadmaps: loop through milestones:
  - Create dedicated deepened plan for milestone
  - Implement milestone
  - Open PR, assign to user, request reviews (Copilot, Codex, Gemini)
  - Wait for reviews by polling PR
  - Triage and address review comments
  - Monitor CI, reproduce failures locally
  - Run `/workflows:compound` for learnings, commit to feature branch
  - Notify user PR is ready for merge
  - Wait for user merge
  - Proceed to next milestone

### Phase 5: Finalize
- Confirm all milestones complete (for roadmaps)
- Update plan to reflect completion status
- Final compounding of learnings

## Open Questions

- Should there be a timeout/retry mechanism for waiting on user merges?
- Should the command support resuming from a specific phase if interrupted?
- How should the command handle merge conflicts when multiple milestones modify overlapping files?

## Technical Notes

- Command file location: `.claude/commands/workflows/pipeline.md`
- Follows pattern of `/lfg` (19 lines) and `/slfg` (31 lines) but will be longer (~80-100 lines) due to phase logic
- The spawned Task agent receives a detailed prompt with all context (plan path, brainstorm path, mode, review instructions)
- Plan completion tracking: update plan markdown with checkbox marks or status annotations per milestone
