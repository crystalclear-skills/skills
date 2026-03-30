# Integration Guide

> How to use CCH skills in Claude, n8n, OpenAI-compatible APIs, and Supabase

---

## Overview

CCH skills are format-agnostic. A skill file contains a system prompt, an input/output schema, and HITL rules. You can inject these into any LLM infrastructure that accepts a system message. This guide shows you how to do it in the four most common environments.

---

## 1. Claude (Direct System Prompt Injection)

The simplest integration: copy the system prompt from any skill file and use it as Claude's system prompt.

### How to inject a skill into Claude

1. Open the skill file (e.g., `skills/social/post_generator.md`)
2. Copy the contents of the `## System Prompt` section
3. Replace all `{{VARIABLE}}` placeholders with your actual values
4. Use the filled-in prompt as your system message

### Example: Claude API (Python)

```python
import anthropic

# Load and fill the skill system prompt
def load_skill(skill_path: str, variables: dict) -> str:
    with open(skill_path, "r") as f:
        content = f.read()
    
    # Extract the System Prompt section
    start = content.find("## System Prompt\n") + len("## System Prompt\n")
    end = content.find("\n## ", start)
    system_prompt = content[start:end].strip()
    
    # Remove markdown code fences if present
    if system_prompt.startswith("```"):
        system_prompt = "\n".join(system_prompt.split("\n")[1:-1])
    
    # Inject variables
    for key, value in variables.items():
        system_prompt = system_prompt.replace(f"{{{{{key}}}}}", str(value))
    
    return system_prompt

system_prompt = load_skill(
    "skills/social/post_generator.md",
    {
        "BRAND_NAME": "Meridian Analytics",
        "PLATFORM": "linkedin",
        "TOPIC": "New AI report feature cuts generation time from 4h to 12min",
        "TONE": "professional",
        "BRAND_VOICE_NOTES": "Data-driven. Let results speak.",
        "CALL_TO_ACTION": "Link in comments",
        "HASHTAG_STRATEGY": "include",
        "VARIATIONS": "1"
    }
)

client = anthropic.Anthropic(api_key="{{ANTHROPIC_API_KEY}}")

response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    system=system_prompt,
    messages=[
        {"role": "user", "content": "Generate the post now."}
    ]
)

print(response.content[0].text)
```

### Claude Projects (no-code)

If you're using Claude Projects:
1. Create a new Project
2. Paste the filled-in system prompt into the **Project Instructions** field
3. Start a conversation — the skill is active for the entire project

---

## 2. n8n (AI Agent Node with Skill as System Prompt)

n8n is the primary workflow engine in the CCH stack. The pattern: AI Agent node with skill system prompt → Approval Gate → IF routing → action.

### Basic workflow structure

```
[Trigger Node]
    → [Set Node: prepare skill inputs]
    → [AI Agent Node: skill system prompt]
    → [AI Agent Node: hitl.approval_gate]
    → [IF Node: route on decision]
        → auto_approve: [Action Node]
        → hitl_required: [Wait Node → Discord alert]
        → halt: [Error handler]
```

### Configuring the AI Agent Node

In the **AI Agent** node:

1. **Model:** Select your preferred model (Claude 3.5 Sonnet, GPT-4o, etc.)
2. **System Message:** Paste the skill's system prompt with n8n variable expressions replacing `{{VARIABLE}}` placeholders

**Example System Message field in n8n:**

```
You are a professional social media strategist operating within the CrystalClearHouse Skill OS.

**Your task:** Write {{ $json.variations }} social media post(s) for {{ $json.brand_name }} on {{ $json.platform }} about: {{ $json.topic }}

**Brand and context:**
- Brand name: {{ $json.brand_name }}
- Platform: {{ $json.platform }}
- Tone: {{ $json.tone }}
[... rest of system prompt ...]
```

### n8n variable mapping

| Skill `{{VARIABLE}}` | n8n expression |
|---|---|
| `{{BRAND_NAME}}` | `{{ $json.brand_name }}` |
| `{{PLATFORM}}` | `{{ $json.platform }}` |
| `{{TOPIC}}` | `{{ $json.topic }}` |
| `{{API_KEY}}` | `{{ $credentials.apiKey }}` |

### Parsing JSON output in n8n

Skills output JSON. In n8n, parse it after the AI Agent node:

```javascript
// In a Code node after the AI Agent
const rawOutput = $input.first().json.output;

// Remove markdown code fences if model wrapped the JSON
const cleaned = rawOutput.replace(/```json\n?/g, '').replace(/```\n?/g, '').trim();

const parsed = JSON.parse(cleaned);

return [{ json: parsed }];
```

### Complete n8n workflow: Social post with HITL

```
Webhook trigger (POST /generate-post)
    → Set node: extract brand_name, platform, topic, tone from body
    → AI Agent (social.post_generator system prompt)
    → Code node: parse JSON output
    → AI Agent (hitl.approval_gate system prompt)
        inputs: skill_id="social.post_generator", confidence_score, action_type="publish_social", flags
    → IF node: {{ $json.decision === "auto_approve" }}
        TRUE → HTTP Request: publish to social platform
        FALSE → IF node: {{ $json.decision === "halt" }}
            TRUE → Send error notification
            FALSE → [HITL flow below]

HITL flow:
    → Set node: build Discord embed payload
    → HTTP Request: POST to Discord webhook
    → Wait node: 30 minutes (resumeWebhookUrl in Discord message)
    → IF node: check approval decision from resume payload
        TRUE → HTTP Request: publish to social platform
        FALSE → HTTP Request: log rejection
    → Supabase insert: write to hitl_audit_log
```

---

## 3. Any OpenAI-Compatible API

Any API that accepts `{"role": "system", "content": "..."}` works with CCH skills.

### Python (OpenAI SDK)

```python
import openai
import json
import re

def extract_skill_section(skill_path: str, section: str) -> str:
    """Extract a named section from a skill Markdown file."""
    with open(skill_path, "r") as f:
        content = f.read()
    
    pattern = rf"## {section}\n(.*?)(?=\n## |\Z)"
    match = re.search(pattern, content, re.DOTALL)
    if not match:
        raise ValueError(f"Section '{section}' not found in {skill_path}")
    return match.group(1).strip()

def fill_variables(text: str, variables: dict) -> str:
    for key, value in variables.items():
        text = text.replace(f"{{{{{key}}}}}", str(value))
    return text

# Load the skill
system_prompt = extract_skill_section(
    "skills/social/post_generator.md", 
    "System Prompt"
)

# Fill variables
system_prompt = fill_variables(system_prompt, {
    "BRAND_NAME": "Meridian Analytics",
    "PLATFORM": "linkedin",
    "TOPIC": "New AI-powered report feature",
    "TONE": "professional",
    "BRAND_VOICE_NOTES": "Data-driven audience. Let results speak.",
    "CALL_TO_ACTION": "Link in comments",
    "HASHTAG_STRATEGY": "include",
    "VARIATIONS": "1"
})

client = openai.OpenAI(api_key="{{OPENAI_API_KEY}}")

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": "Generate the post."}
    ],
    response_format={"type": "json_object"}
)

result = json.loads(response.choices[0].message.content)
print(f"Post: {result['posts'][0]['text']}")
print(f"Confidence: {result['confidence_score']}")
```

### JavaScript / Node.js

```javascript
import OpenAI from "openai";
import fs from "fs";

function extractSection(skillPath, sectionName) {
  const content = fs.readFileSync(skillPath, "utf-8");
  const pattern = new RegExp(`## ${sectionName}\\n([\\s\\S]*?)(?=\\n## |$)`);
  const match = content.match(pattern);
  if (!match) throw new Error(`Section '${sectionName}' not found`);
  return match[1].trim();
}

function fillVariables(text, variables) {
  return Object.entries(variables).reduce(
    (t, [key, value]) => t.replaceAll(`{{${key}}}`, value),
    text
  );
}

const systemPrompt = fillVariables(
  extractSection("skills/social/post_generator.md", "System Prompt"),
  {
    BRAND_NAME: "Meridian Analytics",
    PLATFORM: "linkedin",
    TOPIC: "New AI report feature",
    TONE: "professional",
    BRAND_VOICE_NOTES: "Data-driven. Let results speak.",
    CALL_TO_ACTION: "Link in comments",
    HASHTAG_STRATEGY: "include",
    VARIATIONS: "1",
  }
);

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [
    { role: "system", content: systemPrompt },
    { role: "user", content: "Generate the post." },
  ],
  response_format: { type: "json_object" },
});

const result = JSON.parse(response.choices[0].message.content);
console.log(result.posts[0].text);
```

### Using with Ollama (local models)

```python
import requests
import json

system_prompt = "..."  # filled skill system prompt

response = requests.post(
    "http://localhost:11434/api/chat",
    json={
        "model": "llama3.1",
        "messages": [
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": "Generate the post."}
        ],
        "stream": False
    }
)

result = json.loads(response.json()["message"]["content"])
```

---

## 4. Supabase (Skills Table Schema and Querying)

Store and query skills from Supabase for dynamic skill loading in production workflows.

### Skills table schema

```sql
CREATE TABLE skills (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  skill_id TEXT UNIQUE NOT NULL,           -- e.g. "social.post_generator"
  name TEXT NOT NULL,
  version TEXT NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('draft', 'testing', 'active', 'deprecated')),
  autonomy_level TEXT NOT NULL,
  tags TEXT[] DEFAULT '{}',
  author TEXT NOT NULL,
  system_prompt TEXT NOT NULL,             -- the raw system prompt (no filler text)
  input_schema JSONB,                      -- structured input schema
  output_schema JSONB,                     -- structured output schema
  hitl_rules JSONB,                        -- HITL thresholds and rules
  description TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX skills_skill_id_idx ON skills(skill_id);
CREATE INDEX skills_status_idx ON skills(status);
CREATE INDEX skills_autonomy_level_idx ON skills(autonomy_level);
CREATE INDEX skills_tags_idx ON skills USING gin(tags);
```

### Querying a skill by ID

```javascript
import { createClient } from "@supabase/supabase-js";

const supabase = createClient(
  "{{SUPABASE_URL}}", 
  "{{SUPABASE_SERVICE_ROLE_KEY}}"
);

// Fetch a skill and fill its system prompt
async function loadSkill(skillId, variables) {
  const { data, error } = await supabase
    .from("skills")
    .select("skill_id, name, system_prompt, hitl_rules, autonomy_level")
    .eq("skill_id", skillId)
    .eq("status", "active")
    .single();

  if (error) throw new Error(`Skill not found: ${skillId}`);

  let systemPrompt = data.system_prompt;
  for (const [key, value] of Object.entries(variables)) {
    systemPrompt = systemPrompt.replaceAll(`{{${key}}}`, value);
  }

  return { ...data, filledPrompt: systemPrompt };
}

const skill = await loadSkill("social.post_generator", {
  BRAND_NAME: "Meridian Analytics",
  PLATFORM: "linkedin",
  TOPIC: "New AI report feature",
  TONE: "professional",
  BRAND_VOICE_NOTES: "",
  CALL_TO_ACTION: "Link in comments",
  HASHTAG_STRATEGY: "include",
  VARIATIONS: "1",
});

console.log(skill.filledPrompt);
```

### Querying skills for the orchestrator router

```javascript
// Fetch all active skills for the orchestrator.router input
async function getSkillRegistry() {
  const { data, error } = await supabase
    .from("skills")
    .select("skill_id, name, description, autonomy_level, status")
    .in("status", ["active", "testing"])
    .order("skill_id");

  if (error) throw error;
  return data;
}

const registry = await getSkillRegistry();
// Pass to orchestrator.router as skill_registry input
```

### Inserting a skill from a Markdown file

```python
import re
import yaml
from supabase import create_client

supabase = create_client("{{SUPABASE_URL}}", "{{SUPABASE_SERVICE_ROLE_KEY}}")

def parse_skill_file(path: str) -> dict:
    with open(path) as f:
        content = f.read()
    
    # Extract frontmatter
    fm_match = re.match(r'^---\n(.*?)\n---', content, re.DOTALL)
    frontmatter = yaml.safe_load(fm_match.group(1))
    
    # Extract sections
    def get_section(name):
        pattern = rf"## {name}\n([\s\S]*?)(?=\n## |\Z)"
        match = re.search(pattern, content)
        return match.group(1).strip() if match else None
    
    return {
        "skill_id": frontmatter["skill_id"],
        "name": frontmatter["name"],
        "version": frontmatter["version"],
        "status": frontmatter["status"],
        "autonomy_level": frontmatter["autonomy_level"],
        "tags": frontmatter.get("tags", []),
        "author": frontmatter["author"],
        "description": get_section("Description"),
        "system_prompt": get_section("System Prompt"),
    }

skill_data = parse_skill_file("skills/social/post_generator.md")
result = supabase.table("skills").upsert(skill_data, on_conflict="skill_id").execute()
```

---

## Environment Variables Reference

All integration examples use these placeholder variables. Replace with your actual values in your secrets manager (never in code or config files checked into source control).

| Variable | Description |
|---|---|
| `{{ANTHROPIC_API_KEY}}` | Anthropic API key for Claude |
| `{{OPENAI_API_KEY}}` | OpenAI API key |
| `{{SUPABASE_URL}}` | Your Supabase project URL |
| `{{SUPABASE_SERVICE_ROLE_KEY}}` | Supabase service role key (server-side only) |
| `{{SUPABASE_ANON_KEY}}` | Supabase anon key (client-side) |
| `{{DISCORD_WEBHOOK_URL}}` | Discord webhook URL for HITL alerts |
| `{{N8N_WEBHOOK_URL}}` | n8n webhook URL for resume triggers |

---

For HITL routing logic, see [hitl-protocol.md](hitl-protocol.md).
For autonomy level guidance, see [autonomy-levels.md](autonomy-levels.md).
