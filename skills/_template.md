---
skill_id: category.skill_name
name: Human Readable Skill Name
version: v1.0
status: draft
autonomy_level: 2_advisor
tags: [tag1, tag2]
author: your-github-username
---

## Description

What this skill does, when to use it, and when NOT to use it.

## Input Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `field_name` | `string` | Yes | Description |

## Output Schema

| Field | Type | Description |
|-------|------|-------------|
| `output_field` | `string` | Description |
| `confidence_score` | `float` | 0.0–1.0 self-assessed confidence |
| `flags` | `string[]` | Any concerns or caveats |

## HITL Rules

- `confidence_score >= 0.85` → [auto-proceed / route to human / describe action]
- `confidence_score < 0.85` → Flag for human review
- [Any other routing rules]

## System Prompt

[Your system prompt here. Use {{VARIABLE}} placeholders for dynamic values.]

## Examples

### Example 1
**Input:**
```json
{
  "field_name": "example value"
}
```
**Output:**
```json
{
  "output_field": "example output",
  "confidence_score": 0.92,
  "flags": []
}
```
