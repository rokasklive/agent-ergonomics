# Agent Ergonomics Skills

> Skill repository for the agent-ergonomics family of Claude Code / OpenCode skills.
> These skills encode domain expertise for designing, reviewing, evaluating, and building agent-facing systems — tools, MCP servers, APIs, harnesses, and multi-agent workflows — from the perspective of the AI agents that operate them.

## Included Skills

- **Agent Ergonomics Inspector** (`agent-ergonomics-inspector/`)
- **Design Templates** (`design-templates/`)
- **Evaluation Harness Designer** (`evaluation-harness-designer/`)
- **Implementation Playbooks** (`implementation-playbooks/`)
- **Review Checklists** (`review-checklists/`)

## Eval Results

Each skill ships its own eval suite under `<skill>/evals/evals.json` — benchmark scenarios with graded assertions comparing with-skill vs. without-skill agent performance.

## Structure

Each skill directory contains:
- `SKILL.md` — the skill definition and workflow instructions
- `references/` — reference material (the five laws, five domains, anti-patterns, decision rules, templates, and procedures)
- `evals/` — benchmark scenarios and graded assertions (`evals.json`)

## Export

This repository is generated from a research corpus (`agent-ergonomics`).
The export script (`export-skills.py`) packages the distilled skills and aggregates eval scores.

*Last export: 2026-06-18 17:11 UTC*
