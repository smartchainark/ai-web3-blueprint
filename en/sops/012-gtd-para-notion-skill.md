# GTD + PARA Dual System with Notion MCP Automation

> AI-powered task management combining GTD methodology with PARA organization through Claude Code Skills

**Document ID**: 012
**Date**: 2025-12-29
**Tags**: `GTD` `PARA` `Notion` `MCP` `Task Management` `Skill`

---

## Overview

This document presents a **dual-system architecture** for personal knowledge and task management:

| System | Creator | Purpose |
|--------|---------|---------|
| **GTD** | David Allen | Task capture, clarification, and execution |
| **PARA** | Tiago Forte | Information organization across projects and areas |

By combining these methodologies with **Claude Code Skills** and **Notion MCP**, we achieve:

- Natural language task capture ("add idea: build an automation script")
- Automatic categorization with semantic prefixes
- Zero-friction inbox processing
- Cross-device synchronization via Notion

```
User Intent → Skill Recognition → Notion MCP → Organized System
```

---

## Part 1: Dual System Architecture

### 1.1 GTD Core Workflow

GTD focuses on **capturing everything** and **clarifying next actions**:

```
Capture → Clarify → Organize → Reflect → Engage
```

Our implementation maps to:

| GTD Stage | Implementation |
|-----------|----------------|
| Capture | 📥 Inbox - everything goes here first |
| Clarify | Processing: is it actionable? |
| Organize | Route to Tasks/Areas/Resources/Archive |
| Reflect | Weekly review of all buckets |
| Engage | Execute from 🔥 In Progress |

### 1.2 PARA Organization

PARA provides **four categories** for all information:

| Category | Definition | Our Implementation |
|----------|------------|-------------------|
| **P**rojects | Outcomes with deadlines | 🎯 Tasks (active) |
| **A**reas | Ongoing responsibilities | 🏢 Areas |
| **R**esources | Reference material | 📖 Resources |
| **A**rchives | Completed/inactive | 🗄️ Archive |

### 1.3 Combined Structure

```
GTD + PARA System
├── 📥 Inbox              (GTD: Capture everything, empty daily)
├── 🎯 Tasks              (GTD: Organized actions)
│   ├── 🔥 In Progress    (Currently executing)
│   ├── 💡 Ideas          (Someday/Maybe, needs evaluation)
│   └── 💤 On Hold        (Waiting, future)
├── 🏢 Areas              (PARA: Ongoing responsibilities)
├── 📖 Resources          (PARA: Reference material)
├── 📋 Playbook           (SOPs and methodologies)
└── 🗄️ Archive            (PARA: Completed/inactive)
```

**Key Innovation**: Task states use emoji prefixes for instant visual recognition:
- 🔥 = Active, working on now
- 💡 = Idea, needs clarification
- 💤 = On hold, not now

---

## Part 2: Notion Setup

### 2.1 Prerequisites

1. **Notion Account** with a workspace
2. **Notion MCP** configured in Claude Code
3. **Integration access** to target pages

### 2.2 Create Page Structure

Create pages in Notion matching this hierarchy:

```
GTD System (root page)
├── 📥 Inbox
├── 🎯 Tasks
├── 🏢 Areas
├── 📖 Resources
├── 📋 Playbook
└── 🗄️ Archive
```

### 2.3 Collect Page IDs

Each Notion page has a unique ID. Extract from the page URL:

```
https://notion.so/Your-Page-Title-{PAGE_ID}
                                   ^^^^^^^^
                                   This part
```

**Format**: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` (UUID)

Store these IDs for skill configuration:

```bash
# Example (use your actual IDs)
GTD_ROOT={your-gtd-root-page-id}
INBOX={your-inbox-page-id}
TASKS={your-tasks-page-id}
AREAS={your-areas-page-id}
RESOURCES={your-resources-page-id}
PLAYBOOK={your-playbook-page-id}
ARCHIVE={your-archive-page-id}
```

### 2.4 Grant Integration Access

For each page:
1. Click `···` menu → Connections
2. Add your Claude Code integration
3. Confirm access

---

## Part 3: Skill Implementation

### 3.1 Skill Structure

```
gtd-notion/
└── SKILL.md
```

### 3.2 SKILL.md Template

```yaml
---
name: gtd-notion
description: GTD task management assistant. Uses Notion MCP to auto-categorize
  content into Inbox/Tasks/Areas/Resources/Archive. Supports adding ideas (💡),
  in-progress (🔥), on-hold (💤) tasks. Triggers on "organize GTD", "add task",
  "add idea", "on hold", "archive", "GTD status".
---

# GTD Notion Assistant

Based on GTD + PARA methodology.

## Page ID Configuration

```
GTD_ROOT={gtd-root-id}
INBOX={inbox-id}
TASKS={tasks-id}
AREAS={areas-id}
RESOURCES={resources-id}
PLAYBOOK={playbook-id}
ARCHIVE={archive-id}
```

## Operations Guide

### Add Task

| User Says | Prefix | Example |
|-----------|--------|---------|
| "add idea: xxx" | 💡 | 💡 Build automation script |
| "add task: xxx" | 🔥 | 🔥 Complete auth module |
| "on hold: xxx" | 💤 | 💤 Learn Rust |

**API Call**:
```
mcp__notion__API-post-page
- parent: {"page_id": "{TASKS}"}
- properties: {"title": [{"text": {"content": "💡 Title"}}]}
```

### Process Inbox

1. Get contents: `mcp__notion__API-get-block-children` (block_id: INBOX)
2. List items, ask for categorization
3. Move pages: `mcp__notion__API-move-page`
4. Add prefix: `mcp__notion__API-patch-page`

### View Status

1. Get child pages from each category
2. Count items per category
3. Display summary report

### Archive

1. Search matching pages: `mcp__notion__API-post-search`
2. Move to Archive: `mcp__notion__API-move-page`

## Classification Rules

| Characteristic | Category | Prefix |
|----------------|----------|--------|
| Clear action, has deadline | In Progress | 🔥 |
| Idea, needs evaluation | Ideas | 💡 |
| Future, temporarily paused | On Hold | 💤 |
| Completed, no longer needed | Archive | → Archive |
```

### 3.3 Install the Skill

**Personal skill** (available everywhere):
```bash
mkdir -p ~/.claude/skills/gtd-notion
# Copy SKILL.md to this directory
```

**Project skill** (specific project):
```bash
mkdir -p .claude/skills/gtd-notion
# Copy SKILL.md to this directory
```

---

## Part 4: Usage Examples

### 4.1 Quick Capture

```
User: add idea: AI-powered code review tool

Claude: Created in 🎯 Tasks:
- 💡 AI-powered code review tool
```

### 4.2 Process Inbox

```
User: organize GTD

Claude: Fetching Inbox contents...

📥 Inbox items to process:
1. "New marketing plan"
2. "React learning notes"
3. "Fix login bug"

Please categorize each:
- 1 → Tasks (idea/in-progress/on-hold) / Areas / Resources / Archive?
- 2 → ?
- 3 → ?

User: 1→idea, 2→resources, 3→in-progress

Claude: Done!
- 💡 New marketing plan → Tasks
- React learning notes → Resources
- 🔥 Fix login bug → Tasks
```

### 4.3 Status Overview

```
User: GTD status

Claude:
📊 GTD System Status

🎯 Tasks:
- 🔥 In Progress: 3 items
- 💡 Ideas: 5 items
- 💤 On Hold: 8 items

📥 Inbox: 2 items pending
🏢 Areas: 3 domains
📖 Resources: 12 references
🗄️ Archive: 25 items

⚠️ Recommendation: 2 items in Inbox - process soon.
```

### 4.4 Archive Completed

```
User: archive Fix login bug

Claude: Found matching page: "🔥 Fix login bug"
Moving to Archive...

✅ Archived: Fix login bug
```

---

## Part 5: Best Practices

### 5.1 Daily Workflow

| Time | Action |
|------|--------|
| Morning | Quick capture any overnight thoughts to Inbox |
| Throughout day | Capture everything to Inbox (don't categorize yet) |
| End of day | Process Inbox to zero |

### 5.2 Weekly Review

1. **View status**: Check all categories
2. **Process ideas**: Move viable ideas to In Progress
3. **Review on-hold**: Re-evaluate paused items
4. **Archive completed**: Clean up finished tasks
5. **Reflect on areas**: Are responsibilities balanced?

### 5.3 Prefix Discipline

**Always use prefixes** for tasks:
- Makes scanning instant
- Enables filtering
- Provides visual status at a glance

**Bad**: "Complete the report"
**Good**: "🔥 Complete Q4 report"

### 5.4 Inbox Zero

The Inbox is **not storage** - it's a **processing queue**.

- Items should not stay more than 24 hours
- If unsure, default to 💡 Ideas
- When in doubt, just capture - clarify later

---

## Part 6: Advanced Configuration

### 6.1 Multi-Area Setup

For complex responsibilities, create sub-areas:

```
🏢 Areas
├── Work
│   ├── Project A
│   └── Project B
└── Personal
    ├── Health
    └── Finance
```

### 6.2 Playbook Integration

Store SOPs and methodologies:

```
📋 Playbook
├── Meeting templates
├── Code review checklist
├── Deployment SOP
└── Incident response
```

### 6.3 Automation Extensions

Combine with other skills:

| Trigger | Skill Chain |
|---------|-------------|
| PRD complete | `/doc-coauthoring` → `gtd-notion` (auto-create tasks) |
| Research done | `/notion-knowledge-capture` → `gtd-notion` (file in Resources) |
| Weekly review | `gtd-notion` → Telegram notification (summary) |

---

## Summary

The GTD + PARA dual system provides:

| Benefit | Description |
|---------|-------------|
| **Cognitive offload** | Capture everything, trust the system |
| **Zero friction** | Natural language → organized structure |
| **Visual clarity** | Emoji prefixes for instant recognition |
| **Cross-platform** | Notion syncs everywhere |
| **AI-powered** | Claude handles categorization logic |

**Key Insight**: The power isn't in either system alone - it's in their combination:
- GTD provides the **workflow** (capture → clarify → organize → engage)
- PARA provides the **structure** (where things live)
- Claude provides the **automation** (natural language interface)
- Notion provides the **persistence** (cross-device sync)

---

## References

- [Getting Things Done](https://gettingthingsdone.com/) - David Allen
- [PARA Method](https://fortelabs.com/blog/para/) - Tiago Forte
- [Notion MCP Server](https://github.com/notionhq/notion-mcp-server)
- [Claude Code Skills Documentation](https://docs.anthropic.com/claude-code/skills)
