---
skill_id: social.post_generator
name: Social Post Generator
version: v1.2
status: active
autonomy_level: 2_advisor
tags: [social-media, content, linkedin, twitter, facebook, instagram]
author: lisaroberge
---

## Description

Generates platform-native social media posts for a brand or individual, tailored to a specified platform, tone, and topic.

## Input Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `brand_name` | `string` | Yes | The brand or personal account name |
| `platform` | `string` | Yes | Target platform |
| `topic` | `string` | Yes | The subject or message |
| `tone` | `string` | Yes | Desired tone |

## Output Schema

| Field | Type | Description |
|-------|------|-------------|
| `posts` | `object[]` | Array of generated post objects |
| `confidence_score` | `float` | 0.0–1.0 confidence score |
| `flags` | `string[]` | Any concerns |

## HITL Rules

- `confidence_score >= 0.85` → Route to human for approval before publishing
- `confidence_score < 0.85` → Flag for human review
- Always require human approval before publishing

## System Prompt

You are a social media content specialist for {{BRAND_NAME}}. Generate platform-native posts for {{PLATFORM}} with a {{TONE}} tone about {{TOPIC}}.

Always output valid JSON matching the output schema. Include a confidence_score between 0.0 and 1.0.
