---
name: workflows:pipeline
description: End-to-end development pipeline from brainstorm to merged PR with compounded learnings
argument-hint: "[feature idea or problem to explore]"
completion-promise: "PIPELINE_DONE"
requires-plugins: ["compound-engineering", "ralph-loop"]
---

Run these steps in order. Do not do anything else.

1. `/ralph-loop:ralph-loop "finish all pipeline steps" --completion-promise "PIPELINE_DONE"`

## Phase 1: Brainstorm & Plan (Interactive)

2. `/compound-engineering:workflows:brainstorm $ARGUMENTS`
3. `/compound-engineering:workflows:plan` (auto-detects brainstorm from docs/brainstorms/)
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

   Run `/compound-engineering:workflows:work` to implement the plan on a feature branch and open a PR.

   Then request additional external reviews on the PR:
   - Comment `@codex review`
   - Comment `/gemini review`

   When reviews come in, triage them. Address comments that improve correctness,
   security, or clarity. Explain your rationale when you disagree.

   Run `/compound-engineering:workflows:review` for internal code review. Address any P1/P2 findings.

   Monitor CI with `gh pr checks --watch`. If CI fails, reproduce locally first.

   When done, run `/compound-engineering:workflows:compound` and commit learnings to the branch.
   Notify me the PR is ready for merge with the PR URL."

8. For milestone plans: ask me "Milestone N complete. PR ready at $URL.
   Merge when ready and let me know to continue." Wait for my confirmation
   before spawning the next milestone's agent.

## Phase 3: Done

9. Report completion with PR URL(s).
10. Output `<promise>PIPELINE_DONE</promise>`

Start with step 1 now.
