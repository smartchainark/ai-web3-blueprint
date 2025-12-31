# Replicate Skills from X Platform SOP

> See a tech influencer sharing a cool workflow? Replicate it in 10 minutes.

**Document ID**: 018
**Date**: 2025-12-31
**Tags**: `Grok` `Notion` `Claude Code` `Skill Replication` `Interview Collab`

---

## Overview

This document explains how to discover valuable tech content on X (Twitter), extract key information quickly, and automatically implement reusable skills or tools using Claude Code.

**Core Flow**:
```
Discover Content → Extract Information → Structured Storage → AI Implementation
```

**Key Toolchain**:
- **Grok**: X platform's native AI, excellent at processing tweets/videos
- **Notion**: Structured storage, directly readable by Claude
- **Claude Code**: Automatic implementation from documentation

---

## Part 1: Replication Workflow

### Scenario 1: Video Content

For: Demo videos, tutorials, product demos posted by influencers

#### Step 1: Get Video Link

1. Find the target video tweet on X
2. Click Share → Copy Link
3. Link format: `https://x.com/username/status/xxxxx`

#### Step 2: Grok Transcription

Open Grok (built into X or grok.x.ai), send:

```
Please help me create a complete transcript of this video, including:
1. Word-by-word narration content
2. Key text/code appearing on screen
3. Description of operation steps

Video link: [paste link]
```

**Grok Advantages**:
- Native support for X platform videos
- No download needed, direct processing
- Can recognize screen content

#### Step 3: Store in Notion

1. Create a new page in Notion
2. Title: `[Influencer Name] - [Topic]`
3. Paste Grok's transcript
4. Add your notes (features to implement, improvement ideas, etc.)

#### Step 4: Claude Code Implementation

In Claude Code terminal:

```
Please check this Notion page: [Notion link]

Based on the content, help me implement it as a Claude Skill.
```

Claude will:
1. Read Notion content
2. Analyze workflow structure
3. Create Skill directory and files
4. Implement core functionality

### Scenario 2: Text Tweets

For: Long tweets, Threads, tutorials with screenshots

**Grok Extraction**:
```
Help me organize the complete content of this tweet/Thread: [link]
Extract all text and code/config from screenshots
```

Structure in Notion:

```markdown
## Original Content
[Tweet text]

## Core Features
- Feature 1
- Feature 2

## Implementation Points
- Tech stack
- Key configurations
- Notes

## My Improvement Ideas
- ...
```

### Scenario 3: GitHub Projects

For: Open source projects, code repositories shared by influencers

```
Help me analyze this GitHub project: [link]

Summarize:
1. Project features
2. Core implementation logic
3. Key file structure
4. Usage instructions
```

Then:
```
Based on this project's approach, help me implement a similar Skill/tool.
No need to copy exactly, understand core logic and re-implement.
```

---

## Part 2: Case Study - Interview Collab Skill

Here's a complete case study using this SOP: **Interview Collaborative Writing Skill (interview-collab)**

### Background

Saw an influencer sharing interview writing techniques on X, replicated into a working Claude Skill in 10 minutes.

### Replication Process

```
1. Discovery: Influencer shares interview writing tips video on X
2. Grok: Complete transcript (triggers, questions, platform rules)
3. Notion: Store transcript + add personal requirements
4. Claude Code: Read Notion → Auto-create interview-collab Skill
5. Result: Replicated in 10 minutes, immediately usable
```

### Skill Overview

**Core Philosophy**: Good stories are talked out, not written directly

**Triggers**: `interview`, `访谈`, `采访我`, `帮我写文章`

**Workflow**:
```
User Input → Interview (3-5 rounds) → Material Organization → Platform Selection → Style Learning → Web Research → Article Generation
```

**Two Phases**:
1. **Interview Phase** (Interactive): Like a pro journalist, dig out conflict, emotion, insights - "talk out" the story
2. **Writing Phase** (Automatic): Read style references → Web research for facts → Output complete article

### Four Interview Elements

| Element | Definition | Example Question |
|---------|------------|------------------|
| **Conflict** | Twist, surprise, contradiction | "Was there anything that surprised you?" |
| **Emotion** | Feelings, attitude | "How did you feel at that moment?" |
| **Insight** | Unique perspective, deep understanding | "What do you think is the hidden meaning?" |
| **Detail** | Scene, numbers, dialogue | "Can you describe the specific scene?" |

**End Condition**: Core viewpoint + 1-2 stories/details + emotion + unique perspective + potential quotes

### Platform Adaptation

| Parameter | Xiaohongshu | WeChat Official | Weibo |
|-----------|-------------|-----------------|-------|
| Word Count | 500-800 | 1500-2500 | <140 or 500 |
| Title | emoji + question/exclamation | viewpoint/suspense | direct |
| Opening | Quote/conflict (3-sec hook) | story/background | viewpoint |
| Ending | interaction/resonance | elevation/call-to-action | quote |

### Skill Directory Structure

```
interview-collab/
├── SKILL.md                      # Main file
├── references/
│   ├── SOP.md                    # Complete workflow
│   ├── interview-questions.md    # Question bank
│   ├── platform-styles.md        # Platform style guide
│   ├── story-extraction.md       # Story element extraction
│   └── style-samples/            # User's style reference articles
└── scripts/
    └── interview_tracker.py      # State tracking script
```

### Usage Example

```
User: interview

Claude: Hello! I'm your interview collaboration assistant.
       What topic would you like to discuss today?

User: Want to talk about Manus being acquired by Meta

Claude: What was your first reaction when you heard this news?

User: Pretty surprised, we were just discussing if they'd IPO...

[3-5 rounds later]

Claude: Great, I've collected enough material!
       Which platform would you like to publish on?

User: Xiaohongshu

Claude: [Outputs complete article]
```

### Real Results

Article created using this skill "Manus Acquired by Meta, Entrepreneurs Devastated":
- 10 minutes to complete
- Published on Xiaohongshu
- Nearly 10,000 views

---

## Part 3: Best Practices

### Choose Suitable Content

✅ **Good for Replication**:
- Workflow/SOP types (interview writing, code review processes)
- Automation scripts (batch processing, scheduled tasks)
- Prompt templates (writing assistants, analysis frameworks)

❌ **Not Suitable**:
- Products requiring specific APIs/services
- Highly customized enterprise internal tools
- Features involving paid services

### Improve Success Rate

1. **Information Completeness**: More detailed transcript = more accurate implementation
2. **Add Context**: Tell Claude Code your use case and preferences
3. **Iterate**: First version won't be perfect, keep improving

### Notion Organization Suggestion

```
Replication Projects/
├── To Process/
│   └── [Influencer] - [Topic].md
├── Implemented/
│   └── [Skill Name] - Source: [Influencer].md
└── Reference Materials/
    └── Original transcript archive
```

---

## Quick Reference

| Content Type | Extraction Tool | Key Prompt |
|-------------|-----------------|------------|
| Video | Grok | "Complete transcript" |
| Thread | Grok | "Organize Thread content" |
| Screenshot | Grok/OCR | "Extract text/code from screenshot" |
| GitHub | Claude | "Analyze project core logic" |

| Goal | Claude Code Prompt |
|------|-------------------|
| Create Skill | "Create Skill from Notion content" |
| Create Script | "Implement this automation flow" |
| Create Template | "Organize into reusable Prompt template" |

---

## Final Thoughts

This SOP was created using this very method:

1. Saw influencer sharing interview writing tips
2. Grok transcript
3. Notion storage
4. Claude Code implements Skill
5. Summarize methodology → Write this SOP

**Replication isn't plagiarism - it's the fastest way to learn.**

Understand others' approaches, combine with your needs, create your own tools.

---

## Download Skill Package

📦 **Interview Collab Skill Package**: [interview-collab.zip](../../assets/skills/interview-collab.zip)

**Installation**:

```bash
# 1. Download and extract
unzip interview-collab.zip -d ~/.claude/skills/

# 2. Or add to skill library (recommended)
unzip interview-collab.zip -d ~/Projects/my_skils/library/
cd ~/Projects/my_skils/enabled && ln -s ../library/interview-collab .
```

**Contents**:

- `SKILL.md` - Main skill file
- `references/` - Question bank, platform style guides, story extraction guide
- `scripts/` - Python state tracking script

---

## Resources

- [Grok](https://grok.x.ai) - X Platform AI Assistant
- [Notion MCP](https://github.com/modelcontextprotocol/servers) - Claude reads Notion
- [006 - Skill Creation Standards](./006-claude-skill-creation-sop.md) - Claude Skill Development Guide
