# Agent Instructions

Guidance for AI coding agents working on this project.

## This Project Uses Squads

AI agents are organized into squads — domain-aligned teams defined in `.agents/squads/`.

```
.agents/
├── config/
│   └── SYSTEM.md            # Rules every agent follows
├── squads/
│   └── <squad>/
│       ├── SQUAD.md          # Squad identity, goals, output format
│       └── <agent>.md        # Agent definition
└── memory/
    └── <squad>/<agent>/      # Persistent state
```

## Before Starting Work

```bash
squads status                     # See all squads, milestones, open PRs
squads status <squad>             # Squad detail
squads memory read <squad>        # What the squad already knows
```

## During Work

- Check for existing PRs, specs, and both open and closed issues before creating new ones
- Prefer editing existing files over creating new ones
- Keep changes focused — one task per commit/PR
- Use `--json` on any squads command for machine-readable output

## Research and Proposal Guardrails

- Review `/specs/**` before proposing a new tool so you do not duplicate active or planned work.
- Treat closed issues as prior art unless there is a clearly different angle and you can explain why it is still worth proposing.
- For discovery work, research first, then decide whether any issues should be created.
- Keep discovery bounded: prefer a small shortlist of strong ideas over broad market scans.
- In a single research pass, create at most 3 proposal issues and only when the ideas are distinct and high-confidence.

## Usability Expectations

When proposing or designing a tool, consider whether users need:
- edit, undo, reset, or clear actions
- color selection when color affects the output
- quality or compression controls when fidelity can vary
- preview before export when transformations are involved
- safe defaults, clear validation, and recovery from mistakes

## After Work

- Persist learnings: `squads memory write <squad> "insight"`
- Update state in `.agents/memory/<squad>/<agent>/state.md`
- Create GitHub issues for follow-up work

## Commands

```bash
squads run <squad/agent>          # Run an agent
squads status                     # Overview
squads memory read <squad>        # Recall squad knowledge
squads memory write <squad> "x"   # Persist a learning
squads env show <squad> --json    # Execution context
squads goal list                  # View squad goals
```
