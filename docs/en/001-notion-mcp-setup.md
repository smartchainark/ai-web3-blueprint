# PRD to Tasks to Notification: Complete Workflow SOP

> From PRD writing to Notion task management to Telegram notification - a complete closed loop

**Document**: 001
**Date**: 2025-12-28
**Tags**: `Claude Code` `Notion` `Telegram` `Workflow`

---

## Overview

This document demonstrates a complete project management workflow:

```
PRD Writing → Notion Task Creation → Telegram Notification
```

Through Claude Code's skill combinations, we automate the entire process from requirement analysis to team notifications.

---

## Part 1: Environment Setup

### 1.1 Configure Notion MCP Server

#### Obtain Notion API Token

1. Visit [Notion Integrations](https://www.notion.so/profile/integrations)
2. Click "New integration"
3. Enter integration name (e.g., "Claude Code")
4. Select the associated workspace
5. Click "Submit" and copy the generated Token (starts with `ntn_` or `secret_`)

#### Add MCP Server

```bash
# Project-level configuration (current project only)
claude mcp add notion -e NOTION_TOKEN=your_token -- npx -y @notionhq/notion-mcp-server

# Or: User-level configuration (all projects)
claude mcp add notion -s user -e NOTION_TOKEN=your_token -- npx -y @notionhq/notion-mcp-server
```

#### Verify Connection

```bash
claude mcp list
```

Expected output:
```
notion: npx -y @notionhq/notion-mcp-server - ✓ Connected
```

#### Authorize Page Access

**Important**: Notion API can only access authorized pages.

1. Open the Notion page you want Claude to access
2. Click the `···` menu in the top right corner
3. Select "Connections"
4. Find and select your integration

### 1.2 Configure Telegram Bot

#### Create Bot

1. Search for `@BotFather` in Telegram
2. Send `/newbot` and follow the prompts
3. Save the Bot Token you receive

#### Get Chat ID

```bash
BOT_TOKEN="your_bot_token"

# If there's an active webhook, delete it first
curl -s "https://api.telegram.org/bot$BOT_TOKEN/deleteWebhook"

# Send a message to your Bot, then get the chat_id
curl -s "https://api.telegram.org/bot$BOT_TOKEN/getUpdates" | jq '.result[0].message.chat.id'
```

---

## Part 2: Write PRD Document

### 2.1 Using doc-coauthoring Skill

In Claude Code, type:

```
/doc-coauthoring Write a PRD for [Project Name]
```

### 2.2 Recommended PRD Structure

A concise MVP PRD should include:

```markdown
# Project Name PRD

## 1. Background
- Pain points
- Why this product is needed

## 2. Product Goals
- Core value proposition
- What problem it solves

## 3. Target Users
- User personas

## 4. Functional Requirements
| Feature | Description | Priority |
|---------|-------------|----------|
| Feature A | Description | P0 |
| Feature B | Description | P1 |

## 5. Non-functional Requirements
- Performance requirements
- Security requirements

## 6. MVP Scope
- What's included
- What's not included

## 7. Success Metrics
- How to measure success
```

### 2.3 Example: NotebookLM Integration Assistant PRD

```markdown
# NotebookLM Integration Assistant PRD

> Query Google NotebookLM notebooks via Claude Code

**Version**: MVP v1.0

## 1. Background

NotebookLM is Google's AI notebook tool that provides accurate answers based on
user-uploaded documents. However, it can only be accessed via web browser and
cannot integrate with development tools.

## 2. Product Goals

Enable Claude Code users to query NotebookLM notebooks directly, reducing context switching.

## 3. Functional Requirements

### Authentication
| Feature | Description | Priority |
|---------|-------------|----------|
| Initial Auth | Browser popup Google login | P0 |
| Auth Status Check | View current auth state | P0 |
| Re-authentication | Re-login after token expires | P1 |

### Notebook Library
| Feature | Description | Priority |
|---------|-------------|----------|
| Add Notebook | Add via URL | P0 |
| List Notebooks | View all added notebooks | P0 |
| Set Default | Activate frequently used notebook | P1 |

### Query
| Feature | Description | Priority |
|---------|-------------|----------|
| Ask Question | Query notebook, get AI answer | P0 |
| Specify Notebook | Query specific notebook | P0 |

## 4. Technical Approach

- Browser Automation: Patchright
- Runtime: Python venv
- Data Storage: Local JSON
```

---

## Part 3: PRD to Notion Tasks

### 3.1 API Setup

```bash
# Set environment variables
TOKEN="your_notion_token"
PARENT_PAGE="parent_page_id"
```

### 3.2 Create Project Page

```bash
PROJECT_RESPONSE=$(curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d "{
    \"parent\": {\"page_id\": \"$PARENT_PAGE\"},
    \"properties\": {
      \"title\": {\"title\": [{\"text\": {\"content\": \"Project Name\"}}]}
    },
    \"children\": [
      {
        \"object\": \"block\",
        \"type\": \"heading_2\",
        \"heading_2\": {\"rich_text\": [{\"type\": \"text\", \"text\": {\"content\": \"Project Overview\"}}]}
      },
      {
        \"object\": \"block\",
        \"type\": \"paragraph\",
        \"paragraph\": {\"rich_text\": [{\"type\": \"text\", \"text\": {\"content\": \"Project description...\"}}]}
      }
    ]
  }")

PROJECT_ID=$(echo $PROJECT_RESPONSE | jq -r '.id')
echo "Project page: $PROJECT_ID"
```

### 3.3 Create Task Database

```bash
DB_RESPONSE=$(curl -s -X POST "https://api.notion.com/v1/databases" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d "{
    \"parent\": {\"page_id\": \"$PROJECT_ID\"},
    \"title\": [{\"type\": \"text\", \"text\": {\"content\": \"Task List\"}}],
    \"properties\": {
      \"Task Name\": {\"title\": {}},
      \"Status\": {
        \"select\": {
          \"options\": [
            {\"name\": \"Not Started\", \"color\": \"gray\"},
            {\"name\": \"In Progress\", \"color\": \"blue\"},
            {\"name\": \"Completed\", \"color\": \"green\"}
          ]
        }
      },
      \"Priority\": {
        \"select\": {
          \"options\": [
            {\"name\": \"P0\", \"color\": \"red\"},
            {\"name\": \"P1\", \"color\": \"yellow\"},
            {\"name\": \"P2\", \"color\": \"gray\"}
          ]
        }
      },
      \"Module\": {
        \"select\": {
          \"options\": [
            {\"name\": \"Auth\", \"color\": \"purple\"},
            {\"name\": \"Library\", \"color\": \"blue\"},
            {\"name\": \"Query\", \"color\": \"green\"}
          ]
        }
      }
    }
  }")

DATABASE_ID=$(echo $DB_RESPONSE | jq -r '.id')
echo "Database: $DATABASE_ID"
```

### 3.4 Batch Create Tasks

```bash
create_task() {
  local name="$1"
  local priority="$2"
  local module="$3"

  curl -s -X POST "https://api.notion.com/v1/pages" \
    -H "Authorization: Bearer $TOKEN" \
    -H "Notion-Version: 2022-06-28" \
    -H "Content-Type: application/json" \
    -d "{
      \"parent\": {\"database_id\": \"$DATABASE_ID\"},
      \"properties\": {
        \"Task Name\": {\"title\": [{\"text\": {\"content\": \"$name\"}}]},
        \"Status\": {\"select\": {\"name\": \"Not Started\"}},
        \"Priority\": {\"select\": {\"name\": \"$priority\"}},
        \"Module\": {\"select\": {\"name\": \"$module\"}}
      }
    }" > /dev/null
}

# Auth Module
create_task "Implement Google Login" "P0" "Auth"
create_task "Auth Status Check" "P0" "Auth"
create_task "Token Expiry Handling" "P1" "Auth"

# Library Module
create_task "Add Notebook via URL" "P0" "Library"
create_task "List All Notebooks" "P0" "Library"
create_task "Set Default Notebook" "P1" "Library"

# Query Module
create_task "Basic Question Query" "P0" "Query"
create_task "Specify Notebook Query" "P0" "Query"

echo "Tasks created!"
```

---

## Part 4: Telegram Notification

### 4.1 Send Task Summary

```bash
BOT_TOKEN="your_bot_token"
CHAT_ID="your_chat_id"

# Build message
MESSAGE="📋 *Project Tasks Created*

📁 Project: NotebookLM Integration Assistant
🔗 [View Notion Page](https://notion.so/$PROJECT_ID)

✅ Tasks Created:
• Auth Module: 3 tasks
• Library Module: 3 tasks
• Query Module: 2 tasks

📊 Total: 8 tasks"

# Send message
curl -s -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendMessage" \
  -H "Content-Type: application/json" \
  -d "{
    \"chat_id\": \"$CHAT_ID\",
    \"text\": \"$MESSAGE\",
    \"parse_mode\": \"Markdown\"
  }"
```

### 4.2 Message Formatting

Telegram supports Markdown formatting:

| Format | Syntax | Result |
|--------|--------|--------|
| Bold | `*text*` | **text** |
| Italic | `_text_` | *text* |
| Code | `` `code` `` | `code` |
| Link | `[text](URL)` | hyperlink |

---

## Part 5: Complete Automation Script

Combine all steps into one complete script:

```bash
#!/bin/bash

# === Configuration ===
NOTION_TOKEN="ntn_xxx"
PARENT_PAGE="parent_page_id"
BOT_TOKEN="bot_token"
CHAT_ID="chat_id"

# === Step 1: Create Project Page ===
echo "Creating project page..."
PROJECT_RESPONSE=$(curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d "{
    \"parent\": {\"page_id\": \"$PARENT_PAGE\"},
    \"properties\": {
      \"title\": {\"title\": [{\"text\": {\"content\": \"NotebookLM Integration Assistant\"}}]}
    }
  }")
PROJECT_ID=$(echo $PROJECT_RESPONSE | jq -r '.id')

# === Step 2: Create Task Database ===
echo "Creating task database..."
DB_RESPONSE=$(curl -s -X POST "https://api.notion.com/v1/databases" \
  -H "Authorization: Bearer $NOTION_TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d "{
    \"parent\": {\"page_id\": \"$PROJECT_ID\"},
    \"title\": [{\"type\": \"text\", \"text\": {\"content\": \"Task List\"}}],
    \"properties\": {
      \"Task\": {\"title\": {}},
      \"Status\": {\"select\": {\"options\": [{\"name\": \"Not Started\", \"color\": \"gray\"}]}},
      \"Priority\": {\"select\": {\"options\": [{\"name\": \"P0\", \"color\": \"red\"}, {\"name\": \"P1\", \"color\": \"yellow\"}]}}
    }
  }")
DATABASE_ID=$(echo $DB_RESPONSE | jq -r '.id')

# === Step 3: Create Tasks ===
echo "Creating tasks..."
TASK_COUNT=0
for task in "Google Login Auth" "Auth Status Check" "Add Notebook" "List Notebooks" "Query Notebook"; do
  curl -s -X POST "https://api.notion.com/v1/pages" \
    -H "Authorization: Bearer $NOTION_TOKEN" \
    -H "Notion-Version: 2022-06-28" \
    -H "Content-Type: application/json" \
    -d "{
      \"parent\": {\"database_id\": \"$DATABASE_ID\"},
      \"properties\": {
        \"Task\": {\"title\": [{\"text\": {\"content\": \"$task\"}}]},
        \"Status\": {\"select\": {\"name\": \"Not Started\"}}
      }
    }" > /dev/null
  ((TASK_COUNT++))
done

# === Step 4: Send Telegram Notification ===
echo "Sending notification..."
PROJECT_URL="https://notion.so/${PROJECT_ID//-/}"
MESSAGE="📋 *Project Tasks Created*

📁 Project: NotebookLM Integration Assistant
🔗 [View Notion](${PROJECT_URL})
📊 Tasks: ${TASK_COUNT}

✅ Workflow Complete!"

curl -s -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendMessage" \
  -H "Content-Type: application/json" \
  -d "{
    \"chat_id\": \"$CHAT_ID\",
    \"text\": \"$MESSAGE\",
    \"parse_mode\": \"Markdown\"
  }" > /dev/null

echo "Done! Project page: $PROJECT_URL"
```

---

## Troubleshooting

### Q1: Notion shows "Could not find page with ID"

**Cause**: Page not authorized for the integration

**Solution**: Open the page in Notion → `···` → Connections → Select your integration

### Q2: Telegram getUpdates returns empty

**Cause**: Active webhook or no new messages

**Solution**:
1. Call `deleteWebhook` first
2. Send a message to your Bot
3. Then call `getUpdates`

### Q3: MCP tools not available

**Cause**: MCP tools require a new session after being added

**Solution**: Exit the current Claude Code session and restart

---

## References

- [Notion API Documentation](https://developers.notion.com/docs)
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Notion MCP Server](https://github.com/makenotion/notion-mcp-server)
- [Claude Code Documentation](https://docs.anthropic.com/claude-code)
