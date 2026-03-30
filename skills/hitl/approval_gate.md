---
skill_id: hitl.approval_gate
name: HITL Approval Gate
version: v1.0
status: active
autonomy_level: 0_tool
tags: [hitl, governance, approval]
author: lisaroberge
---

## Description

A deterministic gate that pauses agent execution and routes output to a human for review before proceeding. This is not an AI skill — it is a control flow tool. It accepts agent output plus routing metadata and returns an approval decision.

## Input Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `content` | `string` | Yes | The content requiring human approval |
| `skill_id` | `string` | Yes | The originating skill |
| `confidence_score` | `float` | Yes | Confidence from originating skill |
| `approval_type` | `string` | Yes | `publish`, `send`, `execute`, `delete`, `financial` |
| `timeout_hours` | `integer` | No | Hours before auto-escalation. Default: 24 |

## Output Schema

| Field | Type | Description |
|-------|------|-------------|
| `decision` | `string` | `approved`, `rejected`, `revised`, `escalated` |
| `reviewer` | `string` | Identity of the human reviewer |
| `timestamp` | `string` | ISO 8601 decision timestamp |
| `notes` | `string` | Reviewer notes |

## HITL Rules

This IS the HITL gate. Every execution requires human review. No auto-approval path exists.
