# Agent Ergonomics Skills

> Skill repository for the agent-ergonomics family of Claude Code / OpenCode skills.
> These skills encode domain expertise for designing, reviewing, evaluating, and building agent-facing systems — tools, MCP servers, APIs, harnesses, and multi-agent workflows — from the perspective of the AI agents that operate them.

 To install clone and place into .agents/.claude or:
> `npx skills add rokasklive/agent-ergonomics`

## Included Skills

- **Agent Ergonomics Inspector** (`agent-ergonomics-inspector/`) — Reviews agent-facing systems for usability friction, ambiguity, hidden state, coordination failures, and token waste.
- **Design Templates** (`design-templates/`) — Builds structured templates that force key agent-ergonomics obligations into architecture artifacts.
- **Evaluation Harness Designer** (`evaluation-harness-designer/`) — Designs and audits agent evaluation methods so metrics measure real purpose, reliability, trajectories, and cost.
- **Implementation Playbooks** (`implementation-playbooks/`) — Turns agent-ergonomic designs into dependency-ordered build plans with safety gates and verification steps.
- **Review Checklists** (`review-checklists/`) — Produces evidence-backed go/no-go checklists for agent-facing changes before deployment or sign-off.

## Evals

Each skill ships its own eval suite under `<skill>/evals/evals.json` — benchmark scenarios.

## Structure

Each skill directory contains:
- `SKILL.md` — the skill definition and workflow instructions
- `references/` — reference material (the five laws, five domains, anti-patterns, decision rules, templates, and procedures)
- `evals/` — benchmark scenarios and graded assertions (`evals.json`)

## Export

This repository is generated from a research corpus (`agent-ergonomics`).
The export script (`export-skills.py`) packages the distilled skills and aggregates eval scores.

*Last export: 2026-06-18 17:11 UTC*
