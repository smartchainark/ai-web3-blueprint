# Claude Code Skills Force Activation Hook

> Boost skill activation rate from 20% to 84% using UserPromptSubmit Hook

**Document ID**: 011
**Date**: 2025-12-29
**Tags**: `Claude Code` `Skills` `Hook` `Activation Rate` `Commitment Mechanism`

---

## Overview

Claude Code Skills are designed for "automatic activation," but the actual success rate is only about **20%**. Developer Scott Spence ran 200+ systematic tests and found a solution that boosts the rate to **84%**.

This document explains how to configure this force activation hook.

---

## Problem Analysis

### Why is the native activation rate so low?

Official docs say Claude will "autonomously decide when to use skills based on your request," but in practice:

- Claude often **ignores** configured Skills
- Starts writing code based on its own understanding
- Skill activation feels like "luck"

### Why isn't a simple instruction enough?

Many try adding this to a Hook:

```bash
echo 'INSTRUCTION: If the prompt matches any available skill keywords,
use Skill(skill-name) to activate it.'
```

Success rate is only **50%** — this is just a **passive suggestion** that Claude easily forgets.

---

## Solution: Force Evaluation Hook

### Core Principle: Commitment Mechanism

Force evaluation creates a **three-step commitment mechanism**:

```
1. EVALUATE — State YES/NO with reason for each skill
2. ACTIVATE — Immediately call Skill(skill-name)
3. IMPLEMENT — Only start after activation is complete
```

Once Claude writes "YES - need code generation" in its response, it **commits** to activating that skill.

### Test Results Comparison

| Approach | Success Rate |
|----------|--------------|
| No Hook (native) | ~20% |
| Simple instruction | ~50% |
| **Force evaluation Hook** | **84%** ✅ |

Based on 5 typical tasks, 10 runs each:

| Task Type | Simple Instruction | Force Evaluation |
|-----------|-------------------|------------------|
| Form route creation | 0% | 80% |
| Data loading | 0% | 100% |
| Server operations | 60% | 40% |
| Remote functions | 100% | 100% |
| Reactive state | 100% | 100% |

---

## Installation

### Step 1: Create Hook Script

```bash
# Create directory
mkdir -p ~/.claude/skills/skill-force-activation/scripts

# Create script
cat > ~/.claude/skills/skill-force-activation/scripts/force-skill-eval.sh << 'EOF'
#!/bin/bash
# UserPromptSubmit Hook - Force skill evaluation

cat <<'INSTRUCTION'
INSTRUCTION: Mandatory Skill Activation Protocol

Step 1 - EVALUATE (Must complete in your response):
For each skill in <available_skills>, state: [skill-name] - YES/NO - [reason]

Step 2 - ACTIVATE (Immediately after Step 1):
If any skill is "YES" → Immediately use Skill(skill-name) tool for each relevant skill
If all skills are "NO" → State "No skills needed" and proceed

Step 3 - IMPLEMENT:
Only begin implementation AFTER Step 2 is complete.

CRITICAL: You MUST call the Skill() tool in Step 2. Do NOT skip to implementation.
Evaluation (Step 1) is WORTHLESS without activation (Step 2).

Correct flow example:
- research: NO - not a research task
- gemini-cli: YES - need multimodal processing
- codex-cli: YES - need code generation

[Then IMMEDIATELY use the Skill() tool:]
> Skill(gemini-cli)
> Skill(codex-cli)

[Only AFTER these are complete, begin implementation]
INSTRUCTION
EOF

# Set execute permission
chmod +x ~/.claude/skills/skill-force-activation/scripts/force-skill-eval.sh
```

### Step 2: Configure settings.json

Edit `~/.claude/settings.json` and add the `UserPromptSubmit` Hook:

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/skills/skill-force-activation/scripts/force-skill-eval.sh"
          }
        ]
      }
    ]
  }
}
```

If you have existing hooks, just add the `UserPromptSubmit` section:

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/skills/skill-force-activation/scripts/force-skill-eval.sh"
          }
        ]
      }
    ],
    "Stop": [
      ...existing hooks...
    ]
  }
}
```

### Step 3: Restart Claude Code

After configuration, restart Claude Code for the Hook to take effect.

---

## Workflow

```
User sends message
       ↓
UserPromptSubmit Hook triggers
       ↓
Inject force evaluation instruction
       ↓
Claude must evaluate each skill (YES/NO + reason)
       ↓
Claude activates matching skills
       ↓
Begin task implementation
```

### Example in Action

User input:
```
Help me create a form component
```

Claude response:
```
Skill Evaluation:
- frontend-design: YES - need to create UI component
- typescript-write: YES - need to write TypeScript code
- research: NO - not a research task

> Skill(frontend-design)
> Skill(typescript-write)

Now implementing the form component...
```

---

## Why This Works

### Power of Keywords

Aggressive wording makes Claude harder to ignore:

- **"CRITICAL"** — Emphasizes importance
- **"MUST"** — Mandatory requirement
- **"WORTHLESS"** — Negates value of non-execution

### Power of Commitment

Once Claude writes in its response:
```
- codex-cli: YES - need code generation
```

It has **publicly committed** to activating that skill. Skipping would be self-contradictory.

### Structured Flow

The explicit three-step flow prevents "skipping":

1. Must **evaluate** first (visible in response)
2. Must **activate** next (call Skill tool)
3. Only then can **implement**

---

## Considerations

### Benefits

- Success rate from 20% to 84%
- No skill file modifications needed
- One-time config, global effect
- Doesn't affect other Hooks

### Trade-offs

- Each response consumes slightly more tokens (evaluation portion)
- Response starts with skill evaluation content
- Still 16% failure rate occasionally

### Best Practices

1. **Keep skill descriptions clear** — Easier for Claude to match
2. **Use meaningful skill names** — Like `frontend-design` not `fd`
3. **Check activation regularly** — Confirm Hook is working

---

## Alternative: LLM Pre-evaluation

Another approach uses Claude API to pre-evaluate skill matching:

```bash
#!/bin/bash
RESPONSE=$(curl -s https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -d '{
    "model": "claude-haiku-4-5-20251001",
    "messages": [{"role": "user", "content": "Return matching skills as JSON array..."}]
  }')
```

**Pros**: 10% cheaper per prompt, 17% faster
**Cons**: Unstable, may completely miss some tasks

Recommend using **Force Evaluation Hook** for stability.

---

## FAQ

### Q1: Hook not triggering?

Check script path and permissions:
```bash
ls -la ~/.claude/skills/skill-force-activation/scripts/force-skill-eval.sh
```

### Q2: Evaluation done but no activation?

Confirm settings.json format is correct and Hook is in the right place.

### Q3: Want to disable Hook?

Comment out or remove the `UserPromptSubmit` section in settings.json.

---

## References

- [Original: Claude Code Skills 100% Activation](https://scottspence.com/posts/claude-code-skills-activation)
- [Scott Spence's Testing Framework](https://github.com/spences10/claude-skills-testing)
- [004 - Claude Skills Cross-Platform Setup](./004-claude-skills-setup.md)
- [006 - Claude Skill Creation SOP](./006-claude-skill-creation-sop.md)
