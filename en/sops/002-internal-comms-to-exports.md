# Internal Communications to Multi-Format Export: Complete Workflow SOP

> From generating project updates to PPT/PDF/Word export to Telegram notifications

**Document ID**: 002
**Date**: 2025-12-28
**Tags**: `Claude Code` `Internal Comms` `Document Export` `Workflow`

---

## Overview

This document demonstrates how to use Claude Code skills to automate internal communication generation and multi-format export:

| Step | Skill | Description |
|------|-------|-------------|
| 1. Generate project update | `/internal-comms` | Generate 3P update (Progress/Plans/Problems) |
| 2. Create presentation | `/pptx` | Convert to PowerPoint presentation |
| 3. Generate PDF | `/pdf` | Convert to PDF document |
| 4. Generate Word document | `/docx` | Convert to Word document |
| 5. Team notification | Telegram Bot API | Send update summary to team channel |

```
/internal-comms → /pptx → /pdf → /docx → Telegram notification
```

**Scenario Continuation**: This document continues the NotebookLM project scenario from document 001, demonstrating how to synchronize project progress and team communication after requirements are finalized.

---

## Part 1: Generating Internal Communications

### 1.1 What is a 3P Update

A 3P update is a concise project status report format, suitable for syncing progress with leadership, cross-functional teams, or stakeholders:

- **Progress**: What was accomplished this week
- **Plans**: What's planned for next week
- **Problems**: Current blockers or challenges

### 1.2 Using the internal-comms Skill

The internal-comms skill supports multiple internal communication formats:

| Type | Description | Template File |
|------|-------------|---------------|
| 3P Updates | Progress/Plans/Problems | `examples/3p-updates.md` |
| Company Newsletter | Company-wide announcements | `examples/company-newsletter.md` |
| FAQ Answers | Frequently asked questions | `examples/faq-answers.md` |
| General Comms | Other types | `examples/general-comms.md` |

### 1.3 3P Update Format Specification

```
[emoji] [Team Name] (Date Range)

Progress: [1-3 sentences describing completed work]
Plans: [1-3 sentences describing upcoming work]
Problems: [1-3 sentences describing current blockers]
```

**Key Points**:
- Readable in 30-60 seconds
- Data-driven, include specific metrics
- Clear and concise tone, avoid over-elaboration

### 1.4 Example: NotebookLM Project 3P Update

```markdown
📓 **NotebookLM Integration Assistant Team** (2025-12-23 ~ 2025-12-28)

**Progress**: Completed product PRD writing, used Claude Code skills to
automatically create Notion project page and 8 task items, covering
authentication management (3 tasks), notebook library management (3 tasks),
and query functionality (2 tasks), and sent task summary to team via
Telegram Bot.

**Plans**: Next week will start authentication module development, complete
Patchright browser automation framework setup, implement Google OAuth login
flow, expect to complete P0 authentication features.

**Problems**: No blockers. Current progress is on track, development
resources are in place.
```

---

## Part 2: Multi-Format Document Export

### 2.1 Export to PowerPoint (/pptx skill)

#### Usage

```
/pptx Convert 3P update to presentation, save to ./exports/project-update.pptx
```

#### Skill Features

- Create from scratch or based on template
- Automatic color scheme and layout design
- Support charts, tables, and other elements
- Uses html2pptx or PptxGenJS

#### Recommended Slide Structure

| Slide | Content |
|-------|---------|
| 1 | Title page: Team name + Date |
| 2 | Progress: This week's accomplishments |
| 3 | Plans: Next week's plans |
| 4 | Problems: Current blockers |
| 5 | Milestone table |
| 6 | Closing page |

### 2.2 Export to PDF (/pdf skill)

#### Usage

```
/pdf Convert 3P update to PDF document, save to ./exports/project-update.pdf
```

#### Skill Features

- Uses reportlab for professional PDF creation
- Supports Chinese fonts (PingFang)
- Includes tables, styles, pagination
- Suitable for formal document archiving

### 2.3 Export to Word Document (/docx skill)

#### Usage

```
/docx Convert 3P update to Word document, save to ./exports/project-update.docx
```

#### Skill Features

- Uses docx-js
- Preserves formatting and structure
- Supports tables, lists, styles
- Can be further edited

---

## Part 3: Telegram Notification

### 3.1 Send 3P Update Summary

```bash
BOT_TOKEN="your_bot_token"
CHAT_ID="your_chat_id"

MESSAGE="📓 *NotebookLM Integration Assistant - Weekly Update*

📅 2025-12-23 ~ 2025-12-28

✅ *Progress*:
• Completed PRD writing
• Created Notion project page with 8 tasks
• Configured team notification

📋 *Plans*:
• Start authentication module development
• Set up Patchright browser automation

⚠️ *Problems*:
• No blockers

📎 Attachments generated: PPT / PDF / Word"

curl -s -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendMessage" \
  -H "Content-Type: application/json" \
  -d "{
    \"chat_id\": \"$CHAT_ID\",
    \"text\": \"$MESSAGE\",
    \"parse_mode\": \"Markdown\"
  }"
```

### 3.2 Send File Attachments (Optional)

```bash
# Send PDF file
curl -s -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendDocument" \
  -F "chat_id=$CHAT_ID" \
  -F "document=@./exports/project-update.pdf" \
  -F "caption=📄 NotebookLM Project Weekly Report (PDF)"
```

---

## Part 4: Complete Workflow Example

### 4.1 Complete Flow in Claude Code

```
# Step 1: Generate 3P update
User: Generate this week's 3P update for NotebookLM project

Claude: [Using internal-comms skill format, generates and saves 3P update]
✅ Saved to ./drafts/notebooklm-3p-update.md

# Step 2: Convert to PPT
User: /pptx Convert 3P update to presentation

Claude: [Creates PowerPoint file]
✅ Saved to ./exports/002-project-update.pptx

# Step 3: Convert to PDF
User: /pdf Also generate a PDF version

Claude: [Creates PDF file]
✅ Saved to ./exports/002-project-update.pdf

# Step 4: Convert to Word
User: /docx Also generate a Word version

Claude: [Creates Word file]
✅ Saved to ./exports/002-project-update.docx

# Step 5: Send Telegram notification
User: Send the update summary to Telegram

Claude: [Sends notification]
✅ Message sent
```

### 4.2 Workflow Summary

| Step | Skill/Tool | Input | Output |
|------|------------|-------|--------|
| 1. Generate update | `/internal-comms` | Project info | Markdown 3P update |
| 2. Create PPT | `/pptx` | 3P update | .pptx file |
| 3. Create PDF | `/pdf` | 3P update | .pdf file |
| 4. Create Word | `/docx` | 3P update | .docx file |
| 5. Notify team | Telegram API | Summary | Message push |

---

## Part 5: Skill Dependencies

### 5.1 internal-comms Skill

This is a guideline skill providing format specifications for internal communications:

- Location: `~/.claude/skills/internal-comms/`
- Type: Format guide (not callable)
- Templates: 3P updates, company newsletters, FAQs, etc.

### 5.2 Document Export Skills

| Skill | Dependencies | Description |
|-------|--------------|-------------|
| `/pptx` | pptxgenjs, playwright, sharp | PowerPoint creation |
| `/pdf` | reportlab (Python) | PDF creation |
| `/docx` | docx (npm) | Word document creation |

### 5.3 Environment Setup

```bash
# Node.js dependencies
npm install pptxgenjs docx sharp

# Python dependencies (recommend using venv)
python3 -m venv venv
source venv/bin/activate
pip install reportlab
```

---

## FAQ

### Q1: How to invoke the internal-comms skill?

**Explanation**: internal-comms is a guideline skill, not a directly callable command. Claude automatically references the appropriate format template based on your request.

**Usage**: Simply describe your needs, such as "generate project 3P update" or "write a team weekly report".

### Q2: Chinese garbled text in PDF

**Cause**: Chinese font not configured correctly

**Solution**: Ensure using PingFang font (macOS) or other Chinese-supporting fonts:
```python
from reportlab.pdfbase.ttfonts import TTFont
pdfmetrics.registerFont(TTFont('PingFang', '/System/Library/Fonts/PingFang.ttc'))
```

### Q3: PPT export style not ideal

**Suggestions**:
- Provide clear design requirements when using `/pptx` skill
- Can specify color schemes, fonts, etc.
- Preview thumbnails first before adjusting

---

## Relationship with 001

This document (002) is a continuation of document 001:

| Document | Stage | Workflow |
|----------|-------|----------|
| 001 | Requirements | PRD → Notion tasks → Notification |
| 002 | Execution | 3P update → Document export → Notification |

The two workflows form a complete project management cycle:

```
001: Requirements gathering → Task breakdown → Team sync
         ↓
002: Progress tracking → Status reporting → Multi-format distribution
```

---

## Export Files

This document is available in the following formats:

- [PowerPoint Presentation (PPTX)](../../exports/002-internal-comms-export.pptx)
- [PDF Document](../../exports/002-internal-comms-export.pdf)
- [Word Document (DOCX)](../../exports/002-internal-comms-export.docx)

---

## References

- [reportlab Documentation](https://docs.reportlab.com/)
- [pptxgenjs Documentation](https://gitbrent.github.io/PptxGenJS/)
- [docx Documentation](https://docx.js.org/)
- [Telegram Bot API](https://core.telegram.org/bots/api)
