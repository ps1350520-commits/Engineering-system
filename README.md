# Engineering System

A [Claude Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills) that gives Claude a unified engineering-quality and debugging discipline — a matched **forge + debug** pair in a single skill file.

## What it does

- **Part I — Forge**: quality rules for writing, refactoring, reviewing, and designing code. Ceremony scales across three tiers (**Tier 0–2**), from "always-on hygiene" for small edits up to design notes, threat models, and rollout plans for high-stakes work.
- **Part II — Debug**: a strict, evidence-driven **reproduce → hypothesize → probe → verify → fix** loop for bugs, crashes, flaky tests, and regressions. Depth scales across **D0–D2**, with bisection for large search spaces and a circuit breaker that stops guess-spirals after three disproven hypotheses.

Because ceremony is tiered, the skill stays active on small tasks without over-building them — Claude applies it proactively to any non-trivial coding work, not only when you ask for "production-grade."

## Installation

### Claude Code

Copy the skill into your skills directory:

```bash
mkdir -p ~/.claude/skills/engineering-system
cp SKILL.md ~/.claude/skills/engineering-system/
```

Or per-project, under `.claude/skills/engineering-system/` in the repository.

### Claude.ai

Zip the skill folder and upload it under **Settings → Capabilities → Skills**.

## Usage

No invocation needed — the skill triggers automatically on:

- Non-trivial coding work: writing, refactoring, reviewing, or designing code and systems (backend, frontend, security, architecture, performance, testing, git, databases, APIs, CI/CD).
- Any report that something is broken: bugs, crashes, errors, flaky tests, wrong output, regressions, or a pasted stack trace.

It deliberately does **not** trigger on purely conceptual questions that produce no code, or trivial one-line edits.

## Structure

```
.
├── SKILL.md    # The skill: Part I (Forge) + Part II (Debug)
├── README.md
└── LICENSE
```

## License

[MIT](LICENSE)
