---
skill_id: voice.intent_classifier
name: Voice Intent Classifier
version: v0.7
status: testing
autonomy_level: 1_assistant
tags: [voice, intent, classification, nlp, telephony, triage]
author: lisaroberge
---

## Description

Classifies the intent of a caller utterance against a predefined set of intent categories. Returns a structured classification with confidence scores for each candidate intent, plus a sentiment assessment and urgency signal.

This is a data/classification skill (Level 1). It produces a classification result, not an agent response. It is designed to feed the `voice.call_handler` skill or any routing system that needs to know what a caller wants.

**Use this skill when:**
- Building an IVR-replacement that classifies caller intent before routing
- Analyzing call recordings (transcribed) to categorize call types at scale
- Creating a pre-triage layer before a live voice agent engages

**Do not use this skill for:**
- Generating voice agent responses (use `voice.call_handler`)
- Entity extraction (caller name, phone number, account ID)
- Emotion detection beyond a basic sentiment signal

---

## Input Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `caller_utterance` | `string` | Yes | The transcribed text of the caller's statement or question |
| `available_intents` | `object[]` | Yes | Array of intent objects: `[{"id": "string", "label": "string", "description": "string"}]` |
| `conversation_context` | `string` | No | Brief context from earlier in the call |
| `business_type` | `string` | No | Type of business — helps interpret domain-specific language |
| `allow_multi_intent` | `boolean` | No | Whether to return multiple intents. Default: `false` |

---

## Output Schema

| Field | Type | Description |
|-------|------|-------------|
| `primary_intent` | `string` | The intent ID with the highest confidence score |
| `primary_confidence` | `float` | Confidence in the primary intent (0.0–1.0) |
| `all_scores` | `object[]` | Full scoring: `[{"intent_id": "string", "score": 0.0}]` |
| `secondary_intent` | `string` | Second-highest intent ID (if `allow_multi_intent` or gap < 0.10) |
| `secondary_confidence` | `float` | Confidence in the secondary intent |
| `is_ambiguous` | `boolean` | True if top two intents are within 0.15 of each other |
| `sentiment` | `string` | `positive`, `neutral`, `frustrated`, `distressed` |
| `urgency` | `string` | `low`, `normal`, `high`, `immediate` |
| `utterance_notes` | `string` | Notable observations about the utterance |
| `confidence_score` | `float` | Overall classifier confidence |
| `flags` | `string[]` | Issues: empty utterance, no confident match, distressed caller |

---

## System Prompt

You are a voice intent classifier operating within the CrystalClearHouse Skill OS. You analyze a single caller utterance and classify it against a set of predefined intents. You are a classification system, not a conversational agent — you produce structured data, not a response.

**Caller utterance to classify:**
"{{CALLER_UTTERANCE}}"

**Available intents:**
{{AVAILABLE_INTENTS}}

**Additional context:**
- Conversation context: {{CONVERSATION_CONTEXT}}
- Business type: {{BUSINESS_TYPE}}
- Allow multi-intent: {{ALLOW_MULTI_INTENT}}

**Classification instructions:**

1. Score every available intent from 0.0 to 1.0 based on how well the utterance matches.

2. **Primary intent** is the highest-scoring intent.

3. **Is ambiguous** is `true` when the top two intents are within 0.15 of each other.

4. **Sentiment assessment:**
   - `positive`: Caller sounds pleasant, cooperative
   - `neutral`: No strong emotional signal
   - `frustrated`: Caller expresses irritation or complaint language
   - `distressed`: Caller sounds upset, scared, or in an emergency situation

5. **Urgency assessment:**
   - `immediate`: Safety, medical, or legal emergency language
   - `high`: Time-sensitive language
   - `normal`: Standard request
   - `low`: Casual inquiry

6. If `sentiment` is `distressed` or `urgency` is `immediate`, always add flag: `"ESCALATE_IMMEDIATELY"`.

7. If no intent scores above 0.40, flag: `"NO_CONFIDENT_MATCH"`.

**Output format:**

```json
{
  "primary_intent": "intent_id",
  "primary_confidence": 0.88,
  "all_scores": [{"intent_id": "schedule_appointment", "score": 0.88}],
  "secondary_intent": null,
  "secondary_confidence": null,
  "is_ambiguous": false,
  "sentiment": "neutral",
  "urgency": "normal",
  "utterance_notes": "",
  "confidence_score": 0.88,
  "flags": []
}
```

---

## Examples

### Example 1 — Clear single intent

**Input:**
```json
{
  "caller_utterance": "I need to reschedule my appointment for Thursday",
  "available_intents": [
    {"id": "reschedule_appointment", "label": "Reschedule Appointment", "description": "Caller wants to change an existing appointment"},
    {"id": "cancel_appointment", "label": "Cancel Appointment", "description": "Caller wants to cancel"},
    {"id": "billing_inquiry", "label": "Billing Inquiry", "description": "Caller has a billing question"}
  ],
  "business_type": "medical office"
}
```

**Output:**
```json
{
  "primary_intent": "reschedule_appointment",
  "primary_confidence": 0.97,
  "all_scores": [
    {"intent_id": "reschedule_appointment", "score": 0.97},
    {"intent_id": "cancel_appointment", "score": 0.18},
    {"intent_id": "billing_inquiry", "score": 0.00}
  ],
  "secondary_intent": null,
  "secondary_confidence": null,
  "is_ambiguous": false,
  "sentiment": "neutral",
  "urgency": "normal",
  "utterance_notes": "Caller referenced a specific day (Thursday).",
  "confidence_score": 0.97,
  "flags": []
}
```

---

## HITL Rules

**Autonomy level:** `1_assistant` — This skill classifies intent and produces data. It does not take action.

### Auto-approve conditions
- `primary_confidence` >= 0.70
- `is_ambiguous` is `false`
- No critical flags present

### HITL required conditions
- `primary_confidence` < 0.70
- `is_ambiguous` is `true`
- Flag `"NO_CONFIDENT_MATCH"` present

### Hard halt conditions
- Flag `"ESCALATE_IMMEDIATELY"` present — route to human immediately
- `caller_utterance` is null or empty

### Timeout behavior
- `"ESCALATE_IMMEDIATELY"`: Route within 30 seconds
- Standard HITL: 15 minutes

---

## Changelog

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v0.7 | 2026-02-15 | lisaroberge | Added urgency assessment; improved ambiguity detection |
| v0.5 | 2025-12-20 | lisaroberge | Initial testing release |
