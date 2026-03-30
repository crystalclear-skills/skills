# Skills

This directory contains all published CCH skills, organized by domain.

---

## What Is a Skill?

A skill is a self-contained, versioned AI agent capability. It defines:

- **What the agent can do** — scoped to a specific, well-defined task
- **What inputs it expects** — typed and documented
- **What outputs it produces** — structured and predictable
- **How it governs itself** — confidence thresholds, HITL rules, escalation paths
- **How to use it** — examples that actually work

Each skill is a Markdown file with a YAML frontmatter block and a set of required sections. The format is designed to be readable by humans and parseable by machines.

---

## Skill File Format

Every skill file has this structure:

```
---
[YAML frontmatter]
---

## Description
## Input Schema
## Output Schema
## System Prompt
## Examples
## HITL Rules
## Changelog
```

See [`_template.md`](_template.md) for the canonical template with all fields documented.

---

## Skill Lifecycle

Skills move through four lifecycle states:

```
draft -> testing -> active -> deprecated
```

| Status | Meaning | Can Be Used In Production? |
|--------|---------|--------------------------|
| `draft` | Being written. Schema and prompt not finalized. | No |
| `testing` | Running in staging/test environments. Gathering feedback. | With caution |
| `active` | Validated in production. Schema is stable. | Yes |
| `deprecated` | Superseded by a newer skill or retired. | No — migrate to replacement |

**Do not build production workflows on `draft` skills.** `testing` skills are usable but their schemas may change without a major version bump.

---

## Autonomy Levels and HITL Routing

The `autonomy_level` field in a skill's frontmatter determines how a hosting framework should route its outputs.

| Level | Behavior | HITL Requirement |
|-------|----------|-----------------|
| `0_tool` | Deterministic — no judgment involved | Never requires HITL |
| `1_assistant` | Classification/scoring — produces data, not actions | HITL when confidence < 0.70 |
| `2_advisor` | Drafts and recommendations — human approves before acting | HITL before every external action |
| `3_collaborator` | Multi-step workflows with intermediate checkpoints | HITL at defined phase gates |
| `4_actor` | Full orchestration across systems | HITL on exceptions and irreversible actions |

When building n8n workflows, the autonomy level should map to your IF-node routing logic. See [`docs/hitl-protocol.md`](../docs/hitl-protocol.md) for the full specification.

---

## Domain Structure

Skills are organized into domain directories:

| Domain | Description | Skills |
|--------|-------------|--------|
| `social/` | Social media posting, captioning, hashtag research | 3 |
| `content/` | Video hooks, scripts, long-form content | 2 |
| `hitl/` | Governance primitives — approval gates, confidence scoring | 2 |
| `voice/` | Phone call handling, intent classification | 2 |
| `orchestrator/` | Agent routing and workflow orchestration | 1 |

---

## How to Read a Skill File

1. **Check `status` and `autonomy_level` first.** These tell you whether you can use the skill in production and how much governance you need to wrap around it.

2. **Read `## Description` carefully.** Skills are scoped. A skill that says "writes LinkedIn posts" does not write Twitter posts. Using a skill outside its described scope produces degraded results.

3. **Map your data to `## Input Schema`.** Every required field must be provided. Optional fields are optional — but including them improves output quality.

4. **Understand `## HITL Rules` before deploying.** These are the conditions under which the skill expects a human to be in the loop. Skipping them defeats the governance architecture.

5. **Run the `## Examples`.** Before integrating a skill into a workflow, run its example end-to-end. If the example doesn't work, the skill shouldn't be used.

---

## Template

Start every new skill from [`_template.md`](_template.md). Do not start from scratch — the template enforces the required structure and includes inline documentation for every field.

---

## Questions

See [`../CONTRIBUTING.md`](../CONTRIBUTING.md) or open an issue.
