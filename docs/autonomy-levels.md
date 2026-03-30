# Autonomy Levels

> A complete guide to the CCH five-level autonomy spectrum

---

## Why Autonomy Levels Exist

Not all AI actions carry equal risk. Classifying a customer's intent is different from sending them an SMS. Writing a draft post is different from publishing it.

Autonomy levels exist to match governance intensity to action consequence. Over-governing costs speed and trust in the system. Under-governing costs accountability and safety.

A skill's autonomy level is not a statement about how intelligent or capable the skill is — it is a statement about how much consequence its outputs carry, and therefore how much human oversight is appropriate.

---

## The Five Levels

### Level 0 — `0_tool`

**What it is:** A deterministic function. Input goes in, structured output comes out. No inference, no judgment.

**HITL behavior:** Never required. Level 0 tools may log flags, but flags are informational only.

**Example skills:** `hitl.approval_gate`, `hitl.confidence_scorer`

---

### Level 1 — `1_assistant`

**What it is:** A classification, scoring, or labeling skill. It produces data — not actions, not content.

**HITL behavior:** Required when confidence falls below 0.70.

**Example skills:** `social.hashtag_researcher`, `voice.intent_classifier`

**HITL rule:** Auto-approve when confidence >= 0.70, no flags. HITL required when confidence < 0.70. Hard halt < 0.50.

---

### Level 2 — `2_advisor`

**What it is:** A draft generator. The skill produces content that a human reviews before it reaches the outside world.

**HITL behavior:** Always required before publishing. A Level 2 skill never publishes directly.

**Example skills:** `social.post_generator`, `social.caption_writer`, `content.video_hook_writer`, `voice.call_handler`

**HITL rule:** HITL required before every external action. Even a 0.99 confidence score goes through human review before publishing.

---

### Level 3 — `3_collaborator`

**What it is:** A multi-step execution skill with built-in phase gates. The agent executes phases autonomously, with human checkpoints between phases.

**HITL behavior:** Required at every defined phase gate.

**Example skills:** `content.script_generator`

**HITL rule:** Phase gate HITL is mandatory and non-negotiable. At a phase gate, the human always reviews — even at 0.99 confidence.

**Infrastructure requirement:** Requires a workflow engine that supports pause-and-resume.

---

### Level 4 — `4_actor`

**What it is:** A full orchestration agent. The agent makes decisions about what other agents to run, in what order, with what inputs.

**HITL behavior:** Required on exceptions and all irreversible actions.

**Example skills:** `orchestrator.router`

**HITL rule:** Two mandatory checkpoints: (1) before the routing plan is executed, and (2) before any irreversible action. Hard halt < 0.70 confidence.

---

## How to Choose the Right Level

1. **Does the output require human judgment to evaluate quality?**
   - No -> Level 0 or 1
   - Yes -> Level 2, 3, or 4

2. **Is the output a classification/score, or publishable content?**
   - Classification/score -> Level 1
   - Publishable content -> Level 2+

3. **Does the task require multiple steps with interdependencies?**
   - Single step -> Level 2
   - Multiple steps with checkpoints -> Level 3

4. **Does the task orchestrate other agents or span multiple systems?**
   - Yes -> Level 4
   - No -> Level 0–3

5. **Can an error be easily reversed?**
   - Yes -> can accept higher autonomy
   - No -> stay at lower autonomy or add HITL gates

---

## Level Promotion

Skills can be promoted to higher autonomy levels as they demonstrate reliability.

| From -> To | Required Evidence |
|---|---|
| `2_advisor` -> `3_collaborator` | 100+ successful runs; < 5% HITL rejection rate; defined phase structure |
| `3_collaborator` -> `4_actor` | Full production stability; audit trail in place; error recovery tested |

**Do not promote based on intuition.** Promote based on data from production runs.

---

## HITL Quick Reference by Level

| Level | Auto-Approve | HITL Required | Hard Halt |
|---|---|---|---|
| `0_tool` | Always | Never | On system error |
| `1_assistant` | confidence >= 0.70, no flags | confidence 0.50–0.69, or flags | confidence < 0.50 |
| `2_advisor` | confidence >= 0.85, no flags (AI quality); HITL always before publishing | confidence < 0.85, flags, or before publish | confidence < 0.50 |
| `3_collaborator` | Within phases: confidence >= 0.85 | At every phase gate; confidence < 0.85 in phase | confidence < 0.50 |
| `4_actor` | Within approved plan, confidence >= 0.90 | Plan review; before irreversible actions | confidence < 0.70 |

For detailed threshold tables by action type, see [hitl-protocol.md](hitl-protocol.md).
