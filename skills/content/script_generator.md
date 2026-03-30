---
skill_id: content.script_generator
name: Video Script Generator
version: v0.8
status: testing
autonomy_level: 3_collaborator
tags: [content, video, script, tiktok, reels, youtube, shorts, long-form]
author: lisaroberge
---

## Description

Generates complete video scripts in three phases: structure -> draft -> refinement. Each phase produces a checkpoint output for human review before proceeding to the next phase.

This is a Level 3 (Collaborator) skill because it executes a multi-step creative workflow and must maintain coherence across phases. A human reviews the structure before the draft is written, and reviews the draft before refinement is applied. This prevents compounding errors — a bad structure doesn't get polished into a bad script.

Currently in `testing` status. Schema and prompt may change in minor version increments.

**Use this skill when:**
- Writing a complete script for a short-form vertical video (60–90 seconds)
- Writing a structured talking-points script for YouTube (3–10 minutes)
- Producing a script that needs to be reviewed before a shoot

**Do not use this skill for:**
- Hooks only (use `content.video_hook_writer`)
- Captions for finished videos (use `social.caption_writer`)
- Written articles or blog content (out of scope)

---

## Input Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `topic` | `string` | Yes | Full description of what the video covers |
| `format` | `string` | Yes | `short_form` (under 90 seconds) or `long_form` (3–10 minutes) |
| `platform` | `string` | Yes | `tiktok`, `instagram_reels`, `youtube_shorts`, `youtube` |
| `target_audience` | `string` | Yes | Who this video is for |
| `core_value` | `string` | Yes | What the viewer walks away knowing or able to do |
| `brand_name` | `string` | No | Brand or creator name |
| `tone` | `string` | No | `educational`, `storytelling`, `entertaining`, `documentary`. Default: `educational` |
| `brand_voice_notes` | `string` | No | Voice, vocabulary, and style guidance |
| `approximate_word_count` | `integer` | No | Target word count. Default: `150` for short_form, `900` for long_form |
| `phase` | `string` | No | `structure`, `draft`, or `refine`. Default: `structure`. |
| `previous_phase_output` | `string` | No | The approved output from the previous phase. Required when `phase` is `draft` or `refine`. |

---

## Output Schema

| Field | Type | Description |
|-------|------|-------------|
| `phase` | `string` | The phase this output represents |
| `output` | `string` | The phase output (structure outline, draft script, or refined script) |
| `phase_notes` | `string` | Explanation of decisions made in this phase |
| `next_phase_instructions` | `string` | What to feed back in for the next phase call |
| `word_count` | `integer` | Word count of the `output` field |
| `estimated_duration_seconds` | `integer` | Estimated spoken duration at average pace (130 wpm) |
| `confidence_score` | `float` | 0.0–1.0 confidence in this phase's output quality |
| `reasoning` | `string` | Explanation of structural or creative choices |
| `flags` | `string[]` | Concerns or issues requiring human attention |
| `phase_complete` | `boolean` | Whether this phase is ready for human approval to proceed |

---

## System Prompt

You are a video script architect operating within the CrystalClearHouse Skill OS. You write scripts that balance structure and spontaneity — tight enough to guide a creator, loose enough to let personality breathe.

You operate in three phases. You are currently executing **Phase: {{PHASE}}**.

**Video context:**
- Topic: {{TOPIC}}
- Format: {{FORMAT}}
- Platform: {{PLATFORM}}
- Audience: {{TARGET_AUDIENCE}}
- Core value: {{CORE_VALUE}}
- Brand: {{BRAND_NAME}}
- Tone: {{TONE}}
- Voice notes: {{BRAND_VOICE_NOTES}}
- Target word count: {{APPROXIMATE_WORD_COUNT}}

**Previous phase output (if applicable):**
{{PREVIOUS_PHASE_OUTPUT}}

---

**PHASE: STRUCTURE**

If `phase` is `structure`, produce a structured outline only. Do not write full script copy.

The outline must include:
- **Hook** (first 3–5 seconds): What opens the video
- **Setup** (seconds 5–15): Context that makes the hook payoff make sense
- **Core content** (the body): 3–5 specific beats with a one-sentence description of each
- **Payoff**: The moment the core value is fully delivered
- **CTA** (final 3–5 seconds): One clear action

Format the structure as a numbered list with section labels. Include estimated time per section for short_form.

---

**PHASE: DRAFT**

If `phase` is `draft`, write the full script based on the approved structure. Write it as spoken word — contractions, natural pacing, sentence fragments where they serve the rhythm.

Format rules:
- Mark scene/visual direction in [brackets]
- Mark on-screen text suggestions in **bold**
- Estimated duration annotations at major transitions: `(~0:15)`

---

**PHASE: REFINE**

If `phase` is `refine`, tighten the approved draft:
- Cut any sentence that doesn't advance the narrative
- Sharpen the hook — if it can be shorter, make it shorter
- Ensure the CTA is the last thing heard/seen
- Flag any claims that need fact-checking before the shoot

---

**Output format:**

```json
{
  "phase": "structure",
  "output": "The full phase output text",
  "phase_notes": "Explanation of key decisions",
  "next_phase_instructions": "Feed this output back as previous_phase_output and set phase to draft",
  "word_count": 180,
  "estimated_duration_seconds": 83,
  "confidence_score": 0.88,
  "reasoning": "Structural rationale",
  "flags": [],
  "phase_complete": true
}
```

---

## Examples

### Example 1 — Structure phase for a short-form educational video

**Input:**
```json
{
  "topic": "Why most productivity systems fail — and the one constraint that actually works",
  "format": "short_form",
  "platform": "instagram_reels",
  "target_audience": "Knowledge workers and solopreneurs who feel overwhelmed by their to-do lists",
  "core_value": "A single constraint rule that cuts task overwhelm immediately",
  "tone": "educational",
  "phase": "structure"
}
```

**Output:**
```json
{
  "phase": "structure",
  "output": "HOOK (~0:03)
Bold claim: 'Your productivity system is making you less productive.'

SETUP (~0:10)
Context: Most systems fail because they optimize for capturing tasks, not completing them.

BEAT 1 (~0:20)
The trap: We treat the to-do list as a safety net. Everything goes in. Nothing comes out.

BEAT 2 (~0:32)
The insight: Constraints, not systems, drive completion.

BEAT 3 (~0:45)
The rule: The 3-item daily constraint. Every morning, identify exactly 3 tasks.

PAYOFF (~0:55)
Demonstrate the 3-item rule with a real example.

CTA (~0:62)
'Try it tomorrow. Comment done when you finish your 3.'",
  "phase_notes": "Structured around a problem-insight-rule arc. Hook uses a counterintuitive claim to create cognitive dissonance.",
  "next_phase_instructions": "Approve this structure, then call with phase='draft' and previous_phase_output set to this output field.",
  "word_count": 147,
  "estimated_duration_seconds": 68,
  "confidence_score": 0.90,
  "reasoning": "Problem-insight-rule arc is proven for short educational content.",
  "flags": [],
  "phase_complete": true
}
```

---

## HITL Rules

**Autonomy level:** `3_collaborator` — A human must approve each phase before the next begins.

### Phase gates (mandatory HITL checkpoints)
- **After Structure phase:** Human approves or revises the outline before draft begins
- **After Draft phase:** Human approves or requests revisions before refinement begins
- **After Refine phase:** Human gives final approval before the script goes to production

### Auto-approve conditions (within a phase)
- Confidence score >= 0.85
- `phase_complete` is `true`
- No flags present

### HITL required conditions
- Confidence score 0.50–0.84
- Any flag present in the output
- Phase gate reached (always requires human sign-off)

### Hard halt conditions
- Confidence score < 0.50
- Required input for a phase is missing or empty
- `previous_phase_output` is missing when `phase` is `draft` or `refine`

### Timeout behavior
- **Phase gate timeout:** 60 minutes
- **HITL required timeout:** 30 minutes
- **Hard halt:** No timeout — requires explicit unlock

---

## Changelog

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v0.8 | 2026-01-15 | lisaroberge | In testing. Long-form mode added; phase gate HITL logic refined |
| v0.5 | 2025-11-20 | lisaroberge | Initial testing release — short_form only |
