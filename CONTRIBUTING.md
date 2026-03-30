# Contributing to Skill OS
*Welcome to the revolution. Here's how to join it.*

We're building the operating system for human-aligned intelligence — and we need builders, thinkers, writers, and critics to do it well. Every contribution matters.

---

## Before You Begin

1. Read the [README.md](README.md) — understand the mission.
2. Read the [MANIFESTO.md](MANIFESTO.md) — understand the principles.
3. Read the [Code of Conduct](CODE_OF_CONDUCT.md) — understand the culture.

---

## Ways to Contribute

**Design a new skill** — The most impactful contribution. Add a new skill at any autonomy level.

**Improve an existing skill** — Sharpen a spec, fix an ambiguity, add examples, improve the contract.

**Write documentation** — Clarity is a skill. Help others understand the system.

**Report a governance issue** — Found a skill that violates safety principles? That's a critical contribution.

**Propose architecture changes** — Open a discussion. We build in public.

**Review pull requests** — Skill review is craft.

---

## How to Add a New Skill

### Step 1 — Check existing skills
Browse the `skills/` directory to confirm your skill doesn't already exist.

### Step 2 — Choose the right autonomy level

| Level | Use when… |
|-------|----------|
| **0_tool** | Single deterministic operation. No side effects. |
| **1_assistant** | Executes with minor adjustments. Low autonomy. |
| **2_advisor** | Suggests before acting. HITL on consequential actions. |
| **3_collaborator** | Co-creates with the operator. Reviews before finalizing. |
| **4_actor** | Acts independently within documented constraints. |

### Step 3 — Use the skill template
Copy `skills/_template.md`. Every skill requires:
- `skill_id` — unique, domain.verb_noun format
- `name` — human readable
- `version` — semver (start at v1.0)
- `status` — draft | testing | active | deprecated
- `autonomy_level` — from the table above
- `tags` — array of relevant tags
- Input Schema, Output Schema, System Prompt, Examples, HITL Rules

### Step 4 — Open a pull request
Branch name: `skill/your-skill-id` or `docs/your-change`
Fill out the PR template completely.

---

## n8n Endpoint Config Rules

All n8n HTTP endpoints are defined in `config/n8n_endpoints.json`. Each endpoint must include a `category`:
- `content_automation` — posting, captioning, scheduling, content workflows
- `infra_sensitive` — operations touching infra, secrets, or data pipelines
- `reporting` — analytics, dashboards, metrics-only flows

Run validation before pushing:
```bash
./scripts/check_n8n_endpoints.sh
```

---

## Skill Quality Standards

A skill is ready to merge when:

- [ ] Complete, unambiguous spec
- [ ] Inputs and outputs fully typed
- [ ] Contract explicitly states what it will NOT do
- [ ] HITL rules documented
- [ ] At least one worked example
- [ ] Does not duplicate an existing skill

---

## What We Don't Accept

- Skills with no governance contract
- Skills designed for deception or harm
- Duplicates without clear differentiation
- Breaking changes without discussion
- PRs that skip the template

---

## Where to Discuss

- **GitHub Issues** — proposals, bugs, governance questions
- **GitHub Discussions** — architecture, philosophy, ideas
- **Pull Requests** — specific skill reviews

---

## Recognition

Every merged contributor is listed in commit history.
Every skill carries the name of its author.
Governance contributions are highlighted — they protect the whole system.

---

*This is a community of builders who believe intelligence should be transparent.*
*Your contribution makes that belief real.*
