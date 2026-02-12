# compounded-engineering

End-to-end development pipeline for Claude Code. One command from idea to merged PR.

## How It Works

```
brainstorm → plan → deepen → review → implement → PR → compound
|________ interactive ________|  |_______ autonomous _______|
```

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

```
claude plugin install compound-engineering
```

## Installation

```
git clone https://github.com/Bande-a-Bonnot/compounded-engineering.git
claude --plugin-dir ./compounded-engineering
```

## Usage

Run the full pipeline:

```
/workflows:pipeline
```

Seed with a feature idea:

```
/workflows:pipeline "add rate limiting to the API"
```

### What Happens

Phase 1 (interactive):
1. Brainstorm -- explore the idea with you
2. Plan -- generate an implementation plan
3. Deepen -- enrich with parallel research
4. Review -- multi-agent plan review; approve or reject

Phase 2 (autonomous):

5. Implement -- fresh Task agent per milestone
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
- Remove ralph-loop steps (steps 1 and 10)
- Add your own review commands

## License

MIT
