---
skill_id: content.video_hook_writer
name: Video Hook Writer
version: v1.1
status: active
autonomy_level: 2_advisor
tags: [content, video, hooks, reels, tiktok, youtube, shorts]
author: lisaroberge
---

## Description

Generates high-converting video hooks — the first 3–5 seconds of a video that determine whether a viewer stays or scrolls. The skill produces multiple hook variations using different proven hook formulas, ranked by estimated effectiveness for the target platform and audience.

This skill is narrowly scoped to hooks only. It does not write scripts, captions, or full video outlines (use `content.script_generator` for those).

**Use this skill when:**
- Writing the opening line(s) of a Reel, TikTok, or YouTube Short
- A/B testing multiple hook approaches for the same video topic
- Improving performance of existing video content by rewriting the opening

**Do not use this skill for:**
- Full video scripts (use `content.script_generator`)
- Written captions for completed videos (use `social.caption_writer`)
- Long-form YouTube video intros

---

## Input Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `topic` | `string` | Yes | What the video is about — be as specific as possible |
| `platform` | `string` | Yes | Platform: `tiktok`, `instagram_reels`, `youtube_shorts`, `facebook_reels` |
| `target_audience` | `string` | Yes | Who this video is for — describe the viewer |
| `core_value` | `string` | Yes | The single most valuable thing the viewer gets by watching |
| `brand_name` | `string` | No | Brand or creator name, for voice alignment |
| `tone` | `string` | No | `educational`, `entertaining`, `inspirational`, `controversial`, `storytelling`. Default: `educational` |
| `hook_count` | `integer` | No | Number of hook variations to generate. Default: `5`. Max: `8`. |
| `avoid_phrases` | `string[]` | No | Phrases, hooks, or formats to avoid |

---

## Output Schema

| Field | Type | Description |
|-------|------|-------------|
| `hooks` | `object[]` | Array of hook variations |
| `hooks[].text` | `string` | The hook text |
| `hooks[].formula` | `string` | The hook formula used |
| `hooks[].estimated_effectiveness` | `string` | `high`, `medium`, or `low` |
| `hooks[].notes` | `string` | Brief note on why this hook works |
| `top_recommendation` | `integer` | Index (0-based) of the hook most recommended |
| `confidence_score` | `float` | 0.0–1.0 confidence in hook quality |
| `reasoning` | `string` | Overall explanation of hook selection strategy |
| `flags` | `string[]` | Concerns or caveats |

---

## System Prompt

You are a short-form video strategist and hook specialist operating within the CrystalClearHouse Skill OS. You understand that the first 3 seconds of a video are the entire game. Your job is not to make a clever opening — it is to make a viewer physically unable to scroll past.

**Your task:** Write {{HOOK_COUNT}} video hook variations for a {{PLATFORM}} video about:

**Topic:** {{TOPIC}}
**Target audience:** {{TARGET_AUDIENCE}}
**Core value the video delivers:** {{CORE_VALUE}}
**Tone:** {{TONE}}
**Brand:** {{BRAND_NAME}}
**Avoid:** {{AVOID_PHRASES}}

**The nine proven hook formulas. Use a mix:**

1. **Pattern Interrupt:** Subvert the expected. Forces a double-take.
   Example: "Stop using hashtags. Seriously."

2. **Bold Claim:** Make a strong, specific, potentially controversial statement.
   Example: "Most LinkedIn advice will tank your reach in 2025."

3. **Curiosity Gap:** Set up a question that can only be answered by watching more.
   Example: "There's a reason your reels get 200 views every single time — and it's not the algorithm."

4. **Direct Address:** Speak directly to the specific viewer. Use "you" and a specific identity marker.
   Example: "If you're a first-year founder and you're doing your own bookkeeping, this is for you."

5. **Number / List Promise:** Promise a specific, finite number of valuable things.
   Example: "Three things I stopped doing on Instagram that tripled my reach."

6. **Story Opener:** Start in the middle of a story. Create immediate context and tension.
   Example: "I lost $8,000 on a product launch last year. Here's exactly what went wrong."

7. **Proof Hook:** Lead with a result or credential that earns attention.
   Example: "My client went from 400 followers to 40,000 in 90 days."

8. **Relatable Problem:** Name a pain point the viewer recognizes immediately.
   Example: "You spend 3 hours making a reel and it gets 47 views."

9. **Counterintuitive Statement:** Say the opposite of conventional wisdom.
   Example: "The more you post, the less you should be posting."

**Quality standards:**
- Hooks must be <= 15 words for text overlays; <= 25 words for spoken hooks
- No hook should start with "In this video I'm going to..."
- No hook should start with "Today we're talking about..."
- Every hook should create a reason to keep watching — not just introduce a topic
- Rank by estimated effectiveness for the specific platform and audience

**Output format:**

```json
{
  "hooks": [
    {
      "text": "Hook text here",
      "formula": "Pattern Interrupt",
      "estimated_effectiveness": "high",
      "notes": "Delivery note."
    }
  ],
  "top_recommendation": 0,
  "confidence_score": 0.90,
  "reasoning": "Explanation of selection strategy",
  "flags": []
}
```

---

## Examples

### Example 1 — TikTok hooks for an AI productivity topic

**Input:**
```json
{
  "topic": "How I use AI to batch-create a full month of social content in 2 hours",
  "platform": "tiktok",
  "target_audience": "Solopreneurs and small business owners who struggle with content consistency",
  "core_value": "A repeatable system for creating 30 days of content quickly",
  "tone": "educational",
  "hook_count": 5
}
```

**Output:**
```json
{
  "hooks": [
    {
      "text": "I create a full month of content in 2 hours. Here's the exact system.",
      "formula": "Proof Hook",
      "estimated_effectiveness": "high",
      "notes": "Lead with the result. Let the number do the work."
    },
    {
      "text": "You don't have a content problem. You have a systems problem.",
      "formula": "Counterintuitive Statement",
      "estimated_effectiveness": "high",
      "notes": "Reframes the audience's assumed problem."
    },
    {
      "text": "30 days of content. 2 hours. AI. I'll show you.",
      "formula": "Number / List Promise",
      "estimated_effectiveness": "high",
      "notes": "Staccato structure works well for TikTok pacing."
    },
    {
      "text": "Stop writing social posts from scratch.",
      "formula": "Pattern Interrupt",
      "estimated_effectiveness": "medium",
      "notes": "Short command format. Best paired with a demonstration cut."
    },
    {
      "text": "Solopreneurs who post consistently have automated something you haven't found yet.",
      "formula": "Relatable Problem",
      "estimated_effectiveness": "medium",
      "notes": "Speaks to competitive frustration. Best as spoken hook."
    }
  ],
  "top_recommendation": 1,
  "confidence_score": 0.91,
  "reasoning": "The Counterintuitive statement creates a knowledge gap that pulls viewers into the explanation.",
  "flags": []
}
```

---

## HITL Rules

**Autonomy level:** `2_advisor` — Hook variations are drafts for human selection.

### Auto-approve conditions
- Confidence score >= 0.85
- No flags present
- At least 3 hooks generated

### HITL required conditions
- Confidence score 0.50–0.84
- Any flag present
- Topic was ambiguous and hooks may miss the mark

### Hard halt conditions
- Confidence score < 0.50
- `topic` or `core_value` is empty

### What the human reviewer receives
- All hook variations with formulas, effectiveness ratings, and delivery notes
- Top recommendation highlighted
- Confidence score + reasoning
- Approve selection / Request revision / Reject all

### Timeout behavior
- **HITL required timeout:** 30 minutes — flag for content team
- **Hard halt:** No timeout — requires explicit unlock

---

## Changelog

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.1 | 2025-12-10 | lisaroberge | Added `avoid_phrases` input; expanded hook formula list to 9 |
| v1.0 | 2025-10-20 | lisaroberge | Initial release |---
skill_id: content.video_hook_writer
name: Video Hook Writer
version: v1.1
status: active
autonomy_level: 2_advisor
tags: [content, video, hooks, reels, tiktok, youtube, shorts]
author: lisaroberge
---

## Description

Generates high-converting video hooks — the first 3–5 seconds of a video that determine whether a viewer stays or scrolls. The skill produces multiple hook variations using different proven hook formulas, ranked by estimated effectiveness for the target platform and audience.

This skill is narrowly scoped to hooks only. A hook is the spoken or on-screen text that opens a video. It does not write scripts, captions, or full video outlines (use `content.script_generator` for those).

**Use this skill when:**
- Writing the opening line(s) of a Reel, TikTok, or YouTube Short
- A/B testing multiple hook approaches for the same video topic
- Improving the performance of existing video content by rewriting the opening

**Do not use this skill for:**
- Full video scripts (use `content.script_generator`)
- Written captions for completed videos (use `social.caption_writer`)
- Long-form YouTube video intros (hooks here are designed for short-form vertical video)

---

## Input Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `topic` | `string` | Yes | What the video is about — be as specific as possible |
| `platform` | `string` | Yes | Platform: `tiktok`, `instagram_reels`, `youtube_shorts`, `facebook_reels` |
| `target_audience` | `string` | Yes | Who this video is for — describe the viewer |
| `core_value` | `string` | Yes | The single most valuable thing the viewer gets by watching |
| `brand_name` | `string` | No | Brand or creator name, for voice alignment |
| `tone` | `string` | No | `educational`, `entertaining`, `inspirational`, `controversial`, `storytelling`. Default: `educational` |
| `hook_count` | `integer` | No | Number of hook variations to generate. Default: `5`. Max: `8`. |
| `avoid_phrases` | `string[]` | No | Phrases, hooks, or formats to avoid |

---

## Output Schema

| Field | Type | Description |
|-------|------|-------------|
| `hooks` | `object[]` | Array of hook variations |
| `hooks[].text` | `string` | The hook text — exactly what is spoken or shown on screen |
| `hooks[].formula` | `string` | The hook formula used |
| `hooks[].estimated_effectiveness` | `string` | `high`, `medium`, or `low` |
| `hooks[].notes` | `string` | Brief note on why this hook works and how to deliver it |
| `top_recommendation` | `integer` | Index (0-based) of the hook most recommended |
| `confidence_score` | `float` | 0.0–1.0 confidence in hook quality given the inputs |
| `reasoning` | `string` | Overall explanation of hook selection strategy |
| `flags` | `string[]` | Concerns or caveats |

---

## System Prompt

You are a short-form video strategist and hook specialist operating within the CrystalClearHouse Skill OS. You understand that the first 3 seconds of a video are the entire game.

**Your task:** Write {{HOOK_COUNT}} video hook variations for a {{PLATFORM}} video about:

**Topic:** {{TOPIC}}
**Target audience:** {{TARGET_AUDIENCE}}
**Core value the video delivers:** {{CORE_VALUE}}
**Tone:** {{TONE}}
**Brand:** {{BRAND_NAME}}
**Avoid:** {{AVOID_PHRASES}}

**The nine proven hook formulas. Use a mix:**

1. **Pattern Interrupt:** Subvert the expected. Forces a double-take.
2. **Bold Claim:** Make a strong, specific, potentially controversial statement.
3. **Curiosity Gap:** Set up a question that can only be answered by watching more.
4. **Direct Address:** Speak directly to the specific viewer. Use "you" and a specific identity marker.
5. **Number / List Promise:** Promise a specific, finite number of valuable things.
6. **Story Opener:** Start in the middle of a story. Create immediate context and tension.
7. **Proof Hook:** Lead with a result or credential that earns attention.
8. **Relatable Problem:** Name a pain point the viewer recognizes immediately.
9. **Counterintuitive Statement:** Say the opposite of conventional wisdom.

**Quality standards:**
- Hooks must be <= 15 words for text overlays; <= 25 words for spoken hooks
- No hook should start with "In this video I'm going to..."
- No hook should start with "Today we're talking about..."
- Every hook should create a reason to keep watching — not just introduce a topic

**Output format:**

```json
{
  "hooks": [
    {
      "text": "Hook text here",
      "formula": "Pattern Interrupt",
      "estimated_effectiveness": "high",
      "notes": "Delivery note."
    }
  ],
  "top_recommendation": 0,
  "confidence_score": 0.90,
  "reasoning": "Explanation of selection strategy",
  "flags": []
}
```

---

## Examples

### Example 1 — TikTok hooks for an AI productivity topic

**Input:**
```json
{
  "topic": "How I use AI to batch-create a full month of social content in 2 hours",
  "platform": "tiktok",
  "target_audience": "Solopreneurs and small business owners who struggle with content consistency",
  "core_value": "A repeatable system for creating 30 days of content quickly",
  "tone": "educational",
  "hook_count": 5
}
```

**Output:**
```json
{
  "hooks": [
    {
      "text": "I create a full month of content in 2 hours. Here's the exact system.",
      "formula": "Proof Hook",
      "estimated_effectiveness": "high",
      "notes": "Lead with the result. Let the number do the work."
    },
    {
      "text": "You don't have a content problem. You have a systems problem.",
      "formula": "Counterintuitive Statement",
      "estimated_effectiveness": "high",
      "notes": "Reframes the audience's assumed problem."
    },
    {
      "text": "30 days of content. 2 hours. AI. I'll show you.",
      "formula": "Number / List Promise",
      "estimated_effectiveness": "high",
      "notes": "Staccato structure works well for TikTok pacing."
    },
    {
      "text": "Stop writing social posts from scratch.",
      "formula": "Pattern Interrupt",
      "estimated_effectiveness": "medium",
      "notes": "Short command format. Best paired with a demonstration cut."
    },
    {
      "text": "Solopreneurs who post consistently have automated something you haven't found yet.",
      "formula": "Relatable Problem",
      "estimated_effectiveness": "medium",
      "notes": "Speaks to competitive frustration. Best as spoken hook."
    }
  ],
  "top_recommendation": 1,
  "confidence_score": 0.91,
  "reasoning": "The Counterintuitive statement creates a knowledge gap that pulls viewers into the explanation.",
  "flags": []
}
```

---

## HITL Rules

**Autonomy level:** `2_advisor` — Hook variations are drafts for human selection.

### Auto-approve conditions
- Confidence score >= 0.85
- No flags present
- At least 3 hooks generated

### HITL required conditions
- Confidence score 0.50–0.84
- Any flag present

### Hard halt conditions
- Confidence score < 0.50
- `topic` or `core_value` is empty

### Timeout behavior
- **HITL required timeout:** 30 minutes
- **Hard halt:** No timeout — requires explicit unlock

---

## Changelog

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.1 | 2025-12-10 | lisaroberge | Added `avoid_phrases` input; expanded hook formula list to 9 |
| v1.0 | 2025-10-20 | lisaroberge | Initial release |
