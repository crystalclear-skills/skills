---
skill_id: social.hashtag_researcher
name: Hashtag Researcher
version: v1.0
status: active
autonomy_level: 1_assistant
tags: [social-media, hashtags, instagram, tiktok, research, seo]
author: lisaroberge
---

## Description

Produces a structured set of hashtag recommendations for a given brand, niche, and content type. The skill reasons about hashtag strategy — balancing high-volume discovery tags with mid-tier engagement tags and niche community tags — and outputs a tiered, ready-to-use list.

This is a data/classification skill (Level 1). It outputs research and recommendations, not finished content. The output is intended to feed into `social.post_generator` or `social.caption_writer`, or to be used directly by a human building a hashtag bank.

**Use this skill when:**
- Building a hashtag bank for a new brand or campaign
- Auditing existing hashtag strategy
- Generating platform-specific hashtag sets for different content pillars

**Do not use this skill for:**
- Real-time trending hashtag data (the skill reasons from training knowledge, not live data)
- Generating captions or posts (use `social.post_generator` or `social.caption_writer`)
- Competitor hashtag audits that require live platform data

---

## Input Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `brand_name` | `string` | Yes | Brand or creator name |
| `niche` | `string` | Yes | Primary industry or content niche (e.g., "sustainable fashion," "B2B SaaS," "fitness coaching") |
| `platform` | `string` | Yes | Target platform: `instagram`, `tiktok`, `linkedin`, `twitter` |
| `content_pillars` | `string[]` | Yes | 2–5 content themes or topics the account covers |
| `target_audience` | `string` | No | Description of the intended audience (helps with niche tag selection) |
| `competitor_accounts` | `string[]` | No | Known competitor or peer accounts to draw inspiration from |
| `exclude_tags` | `string[]` | No | Tags to exclude (banned, off-brand, or already used extensively) |
| `output_size` | `integer` | No | Total hashtags to return per pillar. Default: `10`. Range: 5–30. |

---

## Output Schema

| Field | Type | Description |
|-------|------|-------------|
| `strategy_summary` | `string` | 2–3 sentence explanation of the recommended hashtag strategy |
| `tiers` | `object` | Three hashtag tiers: `broad`, `mid`, `niche` |
| `tiers.broad` | `string[]` | High-volume (1M+ posts) discovery tags — use sparingly (2–3 per post) |
| `tiers.mid` | `string[]` | Mid-volume (50K–1M posts) engagement tags — core of the strategy (5–7 per post) |
| `tiers.niche` | `string[]` | Low-volume (<50K posts) community tags — highest engagement rate (3–5 per post) |
| `by_pillar` | `object` | Tags organized by content pillar for easy lookup |
| `by_pillar.{pillar_name}` | `string[]` | Recommended tags for that specific content pillar |
| `usage_guidance` | `string` | Platform-specific instructions on how to deploy these tags |
| `confidence_score` | `float` | 0.0–1.0. Confidence in tag relevance given the niche and platform |
| `flags` | `string[]` | Concerns: niche too broad, platform mismatch, tags that may be flagged/banned |

---

## System Prompt

You are a social media strategist specializing in hashtag research and discovery optimization, operating within the CrystalClearHouse Skill OS. You understand that hashtags serve two purposes: discovery (reaching new audiences) and community (signaling belonging to a niche). Your recommendations balance both.

**Your task:** Research and recommend a hashtag strategy for {{BRAND_NAME}} on {{PLATFORM}}.

**Brand context:**
- Niche: {{NICHE}}
- Content pillars: {{CONTENT_PILLARS}}
- Target audience: {{TARGET_AUDIENCE}}
- Exclude these tags: {{EXCLUDE_TAGS}}
- Competitor/peer accounts: {{COMPETITOR_ACCOUNTS}}
- Output size per category: {{OUTPUT_SIZE}} tags

**Hashtag strategy framework:**

**Tier 1 — Broad (Discovery):** Tags with 1M+ posts. High competition, low engagement rate. Use 2–3 per post for discovery exposure. Examples for fitness: #fitness, #workout, #gym.

**Tier 2 — Mid (Engagement):** Tags with 50K–1M posts. Better signal-to-noise ratio. Use 5–7 per post. These are your workhorses. Examples for fitness: #fitnessmotivation, #strengthtraining, #personaltrainer.

**Tier 3 — Niche (Community):** Tags with under 50K posts. Highest engagement rate. Use 3–5 per post. You're joining a conversation, not shouting. Examples for fitness: #functionalstrengthcoach, #mobilitywork, #movewithintention.

**Platform-specific rules:**

*Instagram:* 5–10 focused tags outperform 30 generic ones. Post tags in the caption or first comment. The algorithm prefers relevance over volume.

*TikTok:* Hashtags drive For You Page placement. Mix 1 trending broad tag with 3–4 content-specific tags. Avoid tag stuffing. #fyp and #viral have declining marginal value — use sparingly.

*LinkedIn:* 3 hashtags max. Choose professional, searchable terms. Hashtags on LinkedIn are followed topics, not keyword stuffing tools.

*Twitter:* 1–2 hashtags max. Only use when joining a genuine trending conversation or recurring chat (#MondayMotivation, etc.).

**Quality standards:**
- All recommended tags must be genuinely relevant to the niche — not trending orphans
- Flag any tag that is known to be banned, shadowbanned, or associated with inappropriate content
- Organize `by_pillar` output so a human can grab the right tags quickly for each content type
- If the niche is very competitive (e.g., "business coaching"), note this and lean into niche tags

**Output format:**

```json
{
  "strategy_summary": "2–3 sentence overview of the recommended approach",
  "tiers": {
    "broad": ["#tag1", "#tag2"],
    "mid": ["#tag3", "#tag4"],
    "niche": ["#tag5", "#tag6"]
  },
  "by_pillar": {
    "pillar_name": ["#tag1", "#tag2", "#tag3"]
  },
  "usage_guidance": "Platform-specific deployment instructions",
  "confidence_score": 0.82,
  "flags": []
}
```

---

## Examples

### Example 1 — Instagram hashtag strategy for B2B SaaS company

**Input:**
```json
{
  "brand_name": "Clearpath Studio",
  "niche": "B2B SaaS / product-led growth",
  "platform": "instagram",
  "content_pillars": ["product updates", "behind the scenes", "founder story", "customer success", "industry insights"],
  "target_audience": "Startup founders and product managers at early-stage B2B companies",
  "output_size": 10
}
```

**Output:**
```json
{
  "strategy_summary": "For a B2B SaaS brand on Instagram targeting founders and PMs, lean heavily into the Tier 2 and Tier 3 tiers. The audience is niche and won't be found via #startup.",
  "tiers": {
    "broad": ["#startup", "#saas", "#technology", "#entrepreneurship"],
    "mid": ["#productledgrowth", "#b2bmarketing", "#startupfounder", "#productmanagement", "#growthhacking", "#techstartup"],
    "niche": ["#plgstrategy", "#b2bsaas", "#startuplife", "#founderjourney", "#saasmarketing", "#productgrowth"]
  },
  "by_pillar": {
    "product_updates": ["#saasproduct", "#productledgrowth", "#newfeature", "#productupdate", "#b2bsaas"],
    "behind_the_scenes": ["#founderjourney", "#startuplife", "#buildinginsaas", "#techteam", "#behindthescenes"],
    "founder_story": ["#startupfounder", "#founderlife", "#entrepreneurship", "#buildinpublic", "#founderjourney"],
    "customer_success": ["#customerstory", "#b2bmarketing", "#customerexperience", "#saassuccess", "#casestudy"],
    "industry_insights": ["#productmanagement", "#growthhacking", "#saasinsights", "#b2bgrowth", "#techtrends"]
  },
  "usage_guidance": "On Instagram, use 7–10 tags per post: 2 from Broad for discovery, 4–5 from Mid for engagement, 3 from the relevant pillar in Niche.",
  "confidence_score": 0.87,
  "flags": ["NOTE: B2B SaaS is a competitive niche on Instagram — organic reach via hashtags is limited."]
}
```

---

## HITL Rules

**Autonomy level:** `1_assistant` — This skill produces research outputs, not published content. Low-stakes output that can be reviewed asynchronously.

### Auto-approve conditions
- Confidence score >= 0.70 (lower threshold than publishing skills — this is research, not publication)
- No flags indicating banned or problematic tags

### HITL required conditions
- Confidence score < 0.70
- Any flag present
- Niche was ambiguous and tags may be off-target

### Hard halt conditions
- Confidence score < 0.50
- `niche` or `content_pillars` inputs are empty

### What the human reviewer receives
- Full hashtag strategy output
- Confidence score with label
- Flags with descriptions
- Approve for use / Request revision

### Timeout behavior
- **HITL required timeout:** 60 minutes — low urgency; auto-log and notify on next review cycle
- **Hard halt:** No timeout — requires explicit unlock

---

## Changelog

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.0 | 2025-10-08 | lisaroberge | Initial release |
