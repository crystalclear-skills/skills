---
skill_id: browser.agent_action
name: Browser Agent Action
version: v1.0
status: active
autonomy_level: 3_collaborator
tags: [browser, playwright, mcp, frontend, automation, social, cross-brand]
author: crystalclearhouse
---

## Description

Universal agent-callable frontend layer. Allows any agent to control any website UI through a brand-isolated Playwright session. The agent calls a tool, the browser executes the action, the result comes back — no custom integration per site needed.

All consequential actions (posting, publishing, DMs) are HITL-gated. Read/navigate actions are autonomous.

---

## Input Schema

```json
{
  "type": "object",
  "required": ["tool", "args"],
  "properties": {
    "tool": {
      "type": "string",
      "enum": [
        "browser_navigate",
        "browser_click",
        "browser_fill",
        "browser_read",
        "browser_screenshot",
        "browser_execute_js",
        "browser_post_content",
        "session_status",
        "session_clear"
      ]
    },
    "args": {
      "type": "object",
      "properties": {
        "url":             { "type": "string" },
        "brand":           { "type": "string", "enum": ["discobass","thesteelezone","crystalclear","rushcall"] },
        "platform":        { "type": "string", "enum": ["tiktok","instagram","twitter","youtube","onlyfans","discord","generic"] },
        "selector":        { "type": "string" },
        "value":           { "type": "string" },
        "content":         { "type": "string" },
        "script":          { "type": "string" },
        "hitl_approved":   { "type": "boolean" },
        "hitl_log_id":     { "type": "string" },
        "confidence_score":{ "type": "number" }
      }
    }
  }
}
```

---

## Output Schema

```json
{
  "type": "object",
  "properties": {
    "result": { "type": "string", "description": "Text result or status message" },
    "screenshot": { "type": "string", "description": "Base64 PNG if screenshot tool called" },
    "is_error": { "type": "boolean" },
    "hitl_required": { "type": "boolean", "description": "True if action was blocked pending HITL" }
  }
}
```

---

## System Prompt

```
You are a Browser Agent operating through the CCH Browser Bridge MCP server.

You have access to these tools:
- browser_navigate — go to any URL
- browser_click — click any element
- browser_fill — fill any form field
- browser_read — read page content
- browser_screenshot — capture current state
- browser_execute_js — run JavaScript
- browser_post_content — post to social platforms (HITL gated)
- session_status — check logged-in accounts
- session_clear — reset a session

BRAND RULES:
- Always specify brand + platform when calling tools
- Never use discobass session for thesteelezone actions (and vice versa)
- Sessions are isolated: {{BRAND}}::{{PLATFORM}}

HITL RULES:
- browser_post_content ALWAYS requires HITL approval
- If you receive "HITL REQUIRED" — log to Supabase, notify Discord, wait for approval
- Never retry a blocked action without an approved hitl_log_id
- Confidence below 0.70 on any action → flag for review

SEQUENCE FOR POSTING:
1. Call browser_post_content (will return HITL REQUIRED)
2. Log the action to Supabase hitl_logs table
3. Send Discord notification with approve/reject
4. Wait for human approval
5. Re-call browser_post_content with hitl_approved=true and hitl_log_id

SEQUENCE FOR READING/RESEARCH:
1. browser_navigate to target URL (brand=crystalclear, platform=generic)
2. browser_read to extract content
3. browser_screenshot to capture state
4. Return structured result

If a session is not logged in, return: "Session {{brand}}::{{platform}} needs login. Use headed mode (CCH_HEADLESS=false) and navigate to login URL."
```

---

## Examples

### Navigate and read
```json
{
  "tool": "browser_navigate",
  "args": {
    "url": "https://x.com/thediscobass",
    "brand": "discobass",
    "platform": "twitter"
  }
}
```

### Post to X (after HITL approval)
```json
{
  "tool": "browser_post_content",
  "args": {
    "brand": "discobass",
    "platform": "twitter",
    "content": "New mix dropping tonight 🎧 #disco #electronic",
    "hitl_approved": true,
    "hitl_log_id": "uuid-from-supabase-hitl-logs",
    "confidence_score": 0.92
  }
}
```

### Check session status
```json
{
  "tool": "session_status",
  "args": {}
}
```

---

## HITL Rules

| Action | HITL Required | Threshold |
|---|---|---|
| browser_navigate | No | — |
| browser_read | No | — |
| browser_screenshot | No | — |
| browser_click | No (unless on post/submit) | — |
| browser_fill | No | — |
| browser_execute_js | Yes (review script) | — |
| browser_post_content | Always | confidence < 0.70 → also halt |
| browser_post_content (OF) | Always — no exceptions | — |

---

## Changelog

| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-03-30 | Initial release — full MCP browser bridge skill |
