# HITL Protocol

> Human-in-the-loop governance specification for CrystalClearHouse Skill OS

Version 2.1 | Last updated: 2026-03

---

## What HITL Means in the CCH Context

Human-in-the-loop (HITL) is not error handling. It is an architectural decision about where human judgment belongs in a workflow.

In the CCH Skill OS, HITL is a first-class component: every skill defines its own HITL rules, every workflow includes a routing layer that enforces them, and every human decision is logged. The goal is not to require human approval everywhere — that negates the value of automation. The goal is to require human approval *exactly where it matters* and nowhere else.

A well-governed agentic workflow has two properties:
1. **Consequential actions don't happen without human sign-off.** A published post, a sent email, a booked appointment, a triggered webhook — these touch the real world. A human approved them.
2. **Non-consequential work happens fast and automatically.** Classifications, drafts, research, scoring — these don't need a human in the loop and shouldn't have one.

HITL is the mechanism that enforces this distinction.

---

## The Confidence Threshold Table

Every skill output carries a `confidence_score` (0.0–1.0). The score is combined with the action type to determine routing.

### Default thresholds by action type

| Action Type | Auto-Approve (>=) | HITL Required (<) | Hard Halt (<) |
|---|---|---|---|
| `advisory_only` | 0.50 | 0.30 | 0.10 |
| `internal_log` | 0.60 | 0.40 | 0.20 |
| `update_record` | 0.75 | 0.60 | 0.40 |
| `webhook_trigger` | 0.80 | 0.65 | 0.45 |
| `publish_social` | 0.85 | 0.70 | 0.50 |
| `send_email` | 0.88 | 0.72 | 0.50 |
| `send_sms` | 0.90 | 0.75 | 0.50 |

### Flags override thresholds

If a skill output contains any flags, HITL is **always** required regardless of confidence score, except for Level 0 skills where flags are informational only.

---

## The Approval Flow

```
1. AGENT DRAFTS OUTPUT
   Skill runs and produces output with confidence_score and flags

2. CONFIDENCE CHECK (hitl.approval_gate)
   Input: skill_id, confidence_score, action_type, flags
   Output: decision (auto_approve / hitl_required / halt)

3A. AUTO-APPROVE PATH
   Log the decision with full audit record
   Proceed to next workflow step immediately

3B. HITL PATH
   Package review bundle
   Route to human review queue
   Start timeout timer
   Wait for human decision
   -> APPROVE: Log and proceed
   -> APPROVE WITH EDITS: Log original + edited + human ID
   -> REJECT: Log rejection reason and trigger re-draft or abandon

3C. HALT PATH
   Log halt with full context
   Notify workflow owner immediately
   Dead letter the output
   Require explicit human unlock to restart

4. AUDIT LOG
   Every decision is written to the audit log
   Record: timestamp, skill_id, decision, confidence_score, action_type, flags, human_id
```

---

## Timeout Rules

| Action Type | Timeout Window | Timeout Action |
|---|---|---|
| `send_sms` | 15 minutes | Escalate to supervisor queue |
| `publish_social` | 30 minutes | Escalate to content manager |
| `send_email` | 30 minutes | Escalate to communications lead |
| `webhook_trigger` | 60 minutes | Escalate to technical lead |
| `update_record` | 60 minutes | Escalate to data owner |
| `internal_log` | 60 minutes | Auto-approve with log note |
| `advisory_only` | 60 minutes | Auto-approve with log note |

**Hard halt has no timeout.** A halted workflow stays halted until a human explicitly unlocks it.

---

## Function Signatures

### `approve_hitl`

```typescript
approve_hitl({
  review_id: string,
  human_id: string,
  edited_output?: string,
  approval_notes?: string,
}): {
  success: boolean,
  review_id: string,
  decision: "approved" | "approved_with_edits",
  human_id: string,
  timestamp_utc: string,
  next_step: string
}
```

### `reject_hitl`

```typescript
reject_hitl({
  review_id: string,
  human_id: string,
  rejection_reason: string,
  fallback_action?: "redraft" | "escalate" | "abandon",
  notes?: string,
}): {
  success: boolean,
  review_id: string,
  decision: "rejected",
  human_id: string,
  rejection_reason: string,
  fallback_action: string,
  timestamp_utc: string
}
```

---

## Integration with n8n

### Basic IF Node Routing Pattern

```
[Skill Node]
    -> [AI Agent Node: hitl.approval_gate]
        -> [IF Node: decision == "auto_approve"]
            -> TRUE branch: [Next Action Node]
            -> FALSE branch: [IF Node: decision == "halt"]
                -> TRUE branch: [Error Handler]
                -> FALSE branch: [HITL branch]

[HITL Branch]
    -> [Set Node: package review bundle]
    -> [Discord Webhook: send alert]
    -> [Wait Node: 30 min timeout]
    -> [HTTP Request: check for human decision]
    -> [IF Node: approved?]
```

### n8n Expressions

```javascript
// In IF Node condition
{{ $json.decision === "auto_approve" }}

// Check for flags
{{ $json.flags_present === true }}
```

---

## Audit Log Schema

```sql
CREATE TABLE hitl_audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  review_id TEXT NOT NULL,
  skill_id TEXT NOT NULL,
  workflow_id TEXT,
  decision TEXT NOT NULL CHECK (decision IN ('auto_approve', 'approved', 'approved_with_edits', 'rejected', 'halted', 'timed_out')),
  confidence_score DECIMAL(3,2) NOT NULL,
  action_type TEXT NOT NULL,
  flags JSONB DEFAULT '[]',
  original_output TEXT,
  final_output TEXT,
  human_id TEXT,
  rejection_reason TEXT,
  notes TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  resolved_at TIMESTAMPTZ
);
```

---

## Quick Reference

| Confidence | No Flags | Flags Present | Action |
|---|---|---|---|
| >= threshold | Yes | — | Auto-approve |
| < threshold, >= halt | — | — | HITL required |
| < halt | — | — | Hard halt |
| Any | — | Yes | HITL required |

For full HITL implementation in your stack, see the [integration guide](integration-guide.md).
