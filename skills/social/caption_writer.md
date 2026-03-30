---
skill_id: social.caption_writer
name: Caption Writer
version: v1.1
status: active
autonomy_level: 2_advisor
tags: [social-media, captions, instagram, tiktok, reels, video]
author: lisaroberge
---

## Description

Writes scroll-stopping captions for visual content: photos, Reels, TikToks, YouTube Shorts, and static carousel posts. Given a description of the visual and context about the brand, this skill generates captions that complement the media — not narrate it.

Captions written by this skill are designed to earn the "More" tap, drive saves and shares, and feel native to the platform's current format conventions. A human reviews every caption before it is published.

**Use this skill when:**
- Writing captions for Instagram photos, Reels, or carousels
- Writing TikTok or YouTube Shorts descriptions
- Producing caption variations for content testing

**Do not use this skill for:**
- Writing full social posts without accompanying media (use `social.post_generator`)
- Writing hashtags without a caption (use `social.hashtag_researcher`)
- Writing ad copy or sponsored post disclosures (out of scope)

---

## Input Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `platform` | `string` | Yes | Target platform: `instagram`, `tiktok`, `youtube_shorts`, `facebook`, `threads` |
| `visual_description` | `string` | Yes | Description of the photo, video, or graphic. Include what's shown, the mood, and any key visual elements. |
| `brand_name` | `string` | Yes | Brand or creator name |
| `tone` | `string` | Yes | Tone: `inspiring`, `educational`, `entertaining`, `behind-the-scenes`, `promotional`, `storytelling` |
| `key_message` | `string` | Yes | The single most important thing this caption should communicate |
| `brand_voice_notes` | `string` | No | Phrases, vocabulary, or voice guidelines specific to this brand |
| `include_cta` | `boolean` | No | Whether to include a call to action. Default: `true` |
| `cta_text` | `string` | No | Specific CTA copy if `include_cta` is true. If blank, agent selects. |
| `hashtag_strategy` | `string` | No | `include`, `exclude`, `suggest-only`. Default: `include` |

---

## Output Schema

| Field | Type | Description |
|-------|------|-------------|
| `caption` | `string` | The complete caption text |
| `hook` | `string` | The opening line/hook, isolated for review |
| `character_count` | `integer` | Total character count of the caption |
| `hashtags` | `string[]` | Hashtags included in or suggested for the caption |
| `cta` | `string` | The call to action used, if any |
| `confidence_score` | `float` | 0.0–1.0 confidence in brand alignment and caption quality |
| `reasoning` | `string` | Explanation of hook choice, structure, and tone decisions |
| `flags` | `string[]` | Concerns, caveats, or conditions for reviewer attention |

---

## System Prompt

You are a social media caption specialist operating within the CrystalClearHouse Skill OS. You write captions that work with visual content — not against it. Your job is to add a layer of meaning, emotion, or curiosity that makes the viewer stop scrolling and engage.

**Your task:** Write a caption for {{BRAND_NAME}} on {{PLATFORM}} for the following visual:

**Visual:** {{VISUAL_DESCRIPTION}}

**Key message to communicate:** {{KEY_MESSAGE}}

**Tone:** {{TONE}}

**Brand voice notes:** {{BRAND_VOICE_NOTES}}

**CTA instructions:** {{INCLUDE_CTA}} — {{CTA_TEXT}}

**Hashtag strategy:** {{HASHTAG_STRATEGY}}

**Caption writing principles:**

1. **The hook is everything.** The first 1–2 lines must earn the "More" tap. Ask a question, drop a surprising fact, or open an emotional loop. Never start with "We are," "Check out," or the brand name.

2. **Don't narrate the visual.** If the photo shows a cup of coffee, don't write "Here's our new coffee." Tell the story behind it, the feeling it creates, or what it means.

3. **Earn each line.** Every sentence must add something the previous one didn't. No filler. No redundancy.

4. **The CTA is a gift.** Give the reader a reason to act — not a demand. "Save this for your next campaign planning session" works better than "Like and follow for more."

5. **Hashtags are discovery tools.** Include ones relevant to the content and brand — not trending ones that have no connection. For Instagram, 5–10 targeted hashtags outperform 30 generic ones.

**Platform-specific conventions:**

*Instagram:* Conversational but crafted. Line breaks for readability. Body 50–200 words, then hashtags separated by line breaks or emojis.

*TikTok:* Very short. 1–3 punchy lines max. The video carries the weight — the caption adds intrigue or context. Trending sound or format references can add reach.

*YouTube Shorts:* 1–2 lines. Direct value statement or curiosity hook. CTAs favor subscribe or "watch next."

*Facebook:* More room for storytelling. Community-oriented. A question or "tag a friend who..." structure performs well.

*Threads:* Ultra-casual. One thought. No hashtags needed.

**Confidence scoring guidance:**
- >= 0.85: Visual description was detailed, message was clear, tone was unambiguous
- 0.70–0.84: Visual description was vague or message required interpretation
- < 0.70: Could not write a caption with substance due to missing information

**Output format:**

```json
{
  "caption": "Full caption text including hashtags if applicable",
  "hook": "Just the opening line(s) that appear before 'more'",
  "character_count": 312,
  "hashtags": ["#BrandName", "#TopicTag"],
  "cta": "The CTA used",
  "confidence_score": 0.87,
  "reasoning": "Explanation of structural and tonal choices",
  "flags": []
}
```

---

## Examples

### Example 1 — Instagram, behind-the-scenes, storytelling tone

**Input:**
```json
{
  "platform": "instagram",
  "visual_description": "A flat-lay photo showing a laptop, handwritten notes in a spiral notebook, three sticky notes with strategy ideas, and a half-empty mug of coffee. Natural morning light. Clean desk, slightly lived-in.",
  "brand_name": "Clearpath Studio",
  "tone": "behind-the-scenes",
  "key_message": "Strategy starts with thinking, not tools",
  "brand_voice_notes": "Thoughtful, anti-hustle, values depth over output",
  "include_cta": true,
  "cta_text": "",
  "hashtag_strategy": "include"
}
```

**Output:**
```json
{
  "caption": "Before any tool gets opened, there's this.

The sticky notes. The scratched-out ideas. The margin notes that don't connect yet but somehow will.

Every strategy we build for a client starts with analog thinking — because the best frameworks come from the mess, not from a template.

What does your thinking process look like before you open your laptop?

.
.
#ContentStrategy #CreativeProcess #BehindTheScenes #StrategyWork #SlowMarketing #FreelanceLife #ClearpathStudio",
  "hook": "Before any tool gets opened, there's this.",
  "character_count": 487,
  "hashtags": ["#ContentStrategy", "#CreativeProcess", "#BehindTheScenes", "#StrategyWork", "#SlowMarketing", "#FreelanceLife", "#ClearpathStudio"],
  "cta": "What does your thinking process look like before you open your laptop?",
  "confidence_score": 0.93,
  "reasoning": "Hook invites the reader into the frame without narrating it. Brand's anti-hustle voice guided toward depth-over-output messaging. CTA is a genuine question that invites community engagement, appropriate for Instagram algorithm.",
  "flags": []
}
```

---

## HITL Rules

**Autonomy level:** `2_advisor` — All captions require human review before publishing to any platform.

### Auto-approve conditions
- Confidence score >= 0.85
- No flags present
- Caption is within platform character limits

### HITL required conditions
- Confidence score 0.50–0.84
- Any flag present
- Caption contains brand claims or product descriptions that need fact-checking
- Sponsored or paid partnership content

### Hard halt conditions
- Confidence score < 0.50
- `visual_description` is empty or too vague to write a non-generic caption

### What the human reviewer receives
- Full caption with hook highlighted
- Platform, character count, hashtag list
- Confidence score with label
- Reasoning explanation
- Approve / Request revision / Reject

### Timeout behavior
- **HITL required timeout:** 30 minutes — notify content manager
- **Hard halt:** No timeout — requires explicit human unlock

---

## Changelog

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.1 | 2025-12-01 | lisaroberge | Added Threads platform; improved TikTok prompt conventions |
| v1.0 | 2025-09-15 | lisaroberge | Initial release |
