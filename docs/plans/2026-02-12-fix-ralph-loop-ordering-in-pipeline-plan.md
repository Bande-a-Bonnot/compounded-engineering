---
title: "fix: Move ralph-loop start to after brainstorm in pipeline"
type: fix
date: 2026-02-12
---

# fix: Move ralph-loop start to after brainstorm in pipeline

The pipeline command starts a ralph loop in step 1, before the brainstorm in step 2. Ralph loop auto-continues the agent, which inhibits interactive questioning during brainstorm. The brainstorm phase needs the agent to ask the user questions — ralph loop suppresses this.

## Acceptance Criteria

- [x] Ralph loop starts after the brainstorm step, not before it
- [x] Brainstorm remains step 1 (interactive, no ralph loop)
- [x] Ralph loop starts as step 2, before the plan step
- [x] Step numbering is updated throughout the file
- [x] No other behavioral changes

## Context

**File:** `plugins/compounded-engineering/commands/workflows/pipeline.md`

**Current order (broken):**
1. `/ralph-loop:ralph-loop` ← starts autonomous loop
2. `/workflows:brainstorm` ← needs interactivity, but ralph loop suppresses it

**Fixed order:**
1. `/workflows:brainstorm $ARGUMENTS` ← interactive, asks questions
2. `/ralph-loop:ralph-loop` ← now safe to start autonomous loop
3. `/workflows:plan` ← autonomous from here on

## Enhancement Summary

**Deepened on:** 2026-02-12
**Sections enhanced:** 1
**Research agents used:** learnings-researcher (no relevant learnings found)

### Key Consideration

The `completion-promise` in the frontmatter (`PIPELINE_DONE`) and the ralph-loop's `--completion-promise` must stay in sync. The ralph loop's promise text should match the `<promise>` tag output in the final step. No change needed here — just noting the invariant.

## MVP

Reorder steps in `pipeline.md`:
- Move brainstorm to step 1 (before ralph loop)
- Move ralph loop to step 2 (after brainstorm, before plan)
- Renumber all subsequent steps
- Split Phase 1 into pre-loop (brainstorm) and post-loop (plan, deepen, review) sections
