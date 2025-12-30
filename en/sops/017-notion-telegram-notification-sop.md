# Notion + Telegram Internal Document Publishing SOP

> Standardized internal document publishing workflow: Write document → Publish to Notion → Telegram group notification

**Document ID**: 017
**Date**: 2025-12-31
**Tags**: `Notion` `Telegram` `SOP` `Automation` `Notification`

---

## Overview

This document describes the standard operating procedure for publishing internal documents to Notion and sending group notifications via Telegram bot. Applicable for project launch announcements, feature releases, technical documentation, etc.

### Core Workflow

```
Local Document → Notion API Publish → Telegram Bot Group Notification
```

### Use Cases

- Project/Feature launch announcements
- Technical architecture documentation
- Deployment records
- Internal team communication documents

---

## Part 1: Prerequisites

### 1.1 Notion Configuration

| Configuration | Description |
|---------------|-------------|
| API Token | Notion Integration API Token |
| Target Page | Parent page for documents (e.g., "Output Plans") |
| Page ID | Parent page UUID |
| Naming Convention | `{MMDD}-{DocumentName}`, e.g., `1231-feature-launch` |

**Getting API Token**:
1. Visit https://www.notion.so/my-integrations
2. Create new Integration
3. Copy Internal Integration Token

**Getting Page ID**:
1. Open target Notion page
2. The 32-character string after `notion.so/` in URL is the ID
3. Or use Search API to find it

### 1.2 Telegram Configuration

| Configuration | Description |
|---------------|-------------|
| Bot Token | Bot Token from Telegram BotFather |
| Chat ID | Target group Chat ID (negative number) |

**Getting Group Chat ID**:
1. Add Bot to group and set as admin
2. Call `getUpdates` API to get group messages
3. Extract `chat.id` from response

```bash
curl -s "https://api.telegram.org/bot<BOT_TOKEN>/getUpdates"
```

---

## Part 2: Operation Flow

### Step 1: Search Target Notion Page

```bash
curl -s -X POST 'https://api.notion.com/v1/search' \
  -H 'Authorization: Bearer <NOTION_TOKEN>' \
  -H 'Content-Type: application/json' \
  -H 'Notion-Version: 2022-06-28' \
  -d '{"query": "target page name", "page_size": 5}'
```

**Response**: Contains page information with `id` field.

### Step 2: Create Notion Child Page

```bash
curl -s -X POST 'https://api.notion.com/v1/pages' \
  -H 'Authorization: Bearer <NOTION_TOKEN>' \
  -H 'Content-Type: application/json' \
  -H 'Notion-Version: 2022-06-28' \
  -d '{
  "parent": { "page_id": "<PARENT_PAGE_ID>" },
  "properties": {
    "title": {
      "title": [{ "text": { "content": "MMDD-Document Title" } }]
    }
  },
  "children": [
    {
      "object": "block",
      "type": "heading_1",
      "heading_1": {
        "rich_text": [{ "type": "text", "text": { "content": "Document Title" } }]
      }
    },
    {
      "object": "block",
      "type": "paragraph",
      "paragraph": {
        "rich_text": [{ "type": "text", "text": { "content": "Document content..." } }]
      }
    }
  ]
}'
```

### Step 3: Send Telegram Group Notification

```bash
curl -s -X POST "https://api.telegram.org/bot<BOT_TOKEN>/sendMessage" \
  -H "Content-Type: application/json" \
  -d '{
    "chat_id": "<CHAT_ID>",
    "parse_mode": "Markdown",
    "text": "📢 *Notification Title*\n\nContent summary...\n\n📋 [View Full Document](NOTION_PAGE_URL)"
  }'
```

---

## Part 3: Notion Block Types Reference

| Type | Usage | Example |
|------|-------|---------|
| `heading_1` | Level 1 heading | Document title |
| `heading_2` | Level 2 heading | Section title |
| `heading_3` | Level 3 heading | Subsection title |
| `paragraph` | Paragraph | Body content |
| `bulleted_list_item` | Unordered list | Feature list |
| `numbered_list_item` | Ordered list | Step instructions |
| `divider` | Divider line | Section separator |
| `code` | Code block | Technical content |
| `callout` | Callout box | Warnings, notes |

---

## Part 4: Telegram Markdown Format

| Format | Syntax | Effect |
|--------|--------|--------|
| Bold | `*text*` | **text** |
| Italic | `_text_` | *text* |
| Code | `` `code` `` | `code` |
| Link | `[text](url)` | Clickable link |

**Note**: Telegram Markdown syntax differs slightly from standard Markdown.

---

## Part 5: Document Template

### Launch Announcement Structure

```markdown
# {Feature Name} Launch Announcement

Release Date: {YYYY-MM-DD} | Status: Live

---

## 📋 Overview
{Brief description, 1-2 sentences}

## 🔗 Live URLs
- Frontend: {URL}
- API: {URL} (if applicable)
- Admin Panel: {URL} (if applicable)

## 📜 Contract Addresses (if applicable)
- Contract1: {address}
- Contract2: {address}

## ✨ Features
1. Feature 1
2. Feature 2
3. Feature 3

## 👤 Access Control (if applicable)
- Admin: {address/role}

## ⚠️ Important Notes
1. Note 1
2. Note 2

---

Contact the development team for questions.
```

---

## Part 6: Troubleshooting

### Notion API Errors

| Error Code | Cause | Solution |
|------------|-------|----------|
| 401 | Invalid Token | Verify API Token is correct |
| 403 | No Permission | Ensure Integration is added to target page |
| 404 | Page Not Found | Verify page_id is correct |
| 400 | Bad Request | Check JSON format and block types |

### Telegram API Errors

| Error Code | Cause | Solution |
|------------|-------|----------|
| 401 | Invalid Bot Token | Verify Token is correct |
| 400 | Invalid Chat ID | Confirm Chat ID format (negative for groups) |
| 403 | No Permission | Ensure Bot is group admin |

---

## Part 7: Claude Code Integration

### Automated Workflow

In Claude Code, this SOP can be encapsulated as a Skill to achieve:

1. **Document Generation**: Auto-generate standardized documents based on project info
2. **Notion Publishing**: Call API to automatically create pages
3. **Group Notification**: Send Telegram notification after publishing

### Configuration Storage

Store configuration in project SOP document:

```yaml
notion:
  token: "ntn_xxx..."
  parent_page_id: "xxx-xxx-xxx"
  naming: "{MMDD}-{DocumentName}"

telegram:
  bot_token: "123456:ABC..."
  chat_id: "-123456789"
  group_name: "Document Notification Group"
```

---

## References

- [Notion API Documentation](https://developers.notion.com/)
- [Telegram Bot API Documentation](https://core.telegram.org/bots/api)
- [Notion Block Types Reference](https://developers.notion.com/reference/block)

---

*Last Updated: 2025-12-31*
