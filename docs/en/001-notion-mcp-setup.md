# Claude Code Notion MCP Setup & PRD to Tasks SOP

## Overview

This document describes how to configure the Notion MCP server in Claude Code and automatically convert local PRD documents into Notion project pages and task databases.

---

## Part 1: Configure Notion MCP Server

### 1.1 Obtain Notion API Token

1. Visit [Notion Integrations](https://www.notion.so/profile/integrations)
2. Click "New integration"
3. Enter integration name (e.g., "Claude Code")
4. Select the associated workspace
5. Click "Submit" and copy the generated Token (starts with `ntn_` or `secret_`)

### 1.2 Add MCP Server in Claude Code

**Option 1: Project-level configuration (current project only)**

```bash
claude mcp add notion -e NOTION_TOKEN=your_token -- npx -y @notionhq/notion-mcp-server
```

**Option 2: User-level configuration (all projects)**

```bash
claude mcp add notion -s user -e NOTION_TOKEN=your_token -- npx -y @notionhq/notion-mcp-server
```

### 1.3 Verify Connection

```bash
claude mcp list
```

Expected output:
```
notion: npx -y @notionhq/notion-mcp-server - ✓ Connected
```

### 1.4 Authorize Page Access

**Important**: Notion API can only access authorized pages.

1. Open the Notion page you want Claude to access
2. Click the `···` menu in the top right corner
3. Select "Connections"
4. Find and select your integration

---

## Part 2: PRD to Notion Tasks

### 2.1 Prerequisites

Ensure:
- Notion MCP is configured and connected
- Target pages are authorized for the integration
- PRD document is ready (Markdown format)

### 2.2 API Call Pattern

Since MCP tools require a new session to take effect, you can call the Notion API directly using curl:

```bash
# Set environment variable
TOKEN="your_notion_token"
```

### 2.3 Search Authorized Pages

```bash
curl -s -X POST "https://api.notion.com/v1/search" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"page_size": 10}'
```

### 2.4 Create Project Page

```bash
curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"page_id": "parent_page_id"},
    "properties": {
      "title": {"title": [{"text": {"content": "Project Name"}}]}
    },
    "children": [
      {
        "object": "block",
        "type": "heading_1",
        "heading_1": {"rich_text": [{"type": "text", "text": {"content": "Project Overview"}}]}
      },
      {
        "object": "block",
        "type": "paragraph",
        "paragraph": {"rich_text": [{"type": "text", "text": {"content": "Project description..."}}]}
      }
    ]
  }'
```

### 2.5 Create Task Database

```bash
curl -s -X POST "https://api.notion.com/v1/databases" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"page_id": "project_page_id"},
    "title": [{"type": "text", "text": {"content": "Task List"}}],
    "properties": {
      "Task Name": {"title": {}},
      "Status": {
        "select": {
          "options": [
            {"name": "Not Started", "color": "gray"},
            {"name": "In Progress", "color": "blue"},
            {"name": "Completed", "color": "green"},
            {"name": "Blocked", "color": "red"}
          ]
        }
      },
      "Priority": {
        "select": {
          "options": [
            {"name": "P0-Critical", "color": "red"},
            {"name": "P1-Important", "color": "yellow"},
            {"name": "P2-Normal", "color": "gray"}
          ]
        }
      },
      "Phase": {
        "select": {
          "options": [
            {"name": "Phase 1: Design", "color": "purple"},
            {"name": "Phase 2: Development", "color": "blue"},
            {"name": "Phase 3: Release", "color": "green"}
          ]
        }
      },
      "Module": {
        "select": {
          "options": [
            {"name": "Module A", "color": "orange"},
            {"name": "Module B", "color": "yellow"}
          ]
        }
      }
    }
  }'
```

### 2.6 Batch Create Tasks

```bash
DATABASE_ID="database_id"

create_task() {
  local name="$1"
  local task_status="$2"
  local priority="$3"
  local phase="$4"
  local module="$5"

  curl -s -X POST "https://api.notion.com/v1/pages" \
    -H "Authorization: Bearer $TOKEN" \
    -H "Notion-Version: 2022-06-28" \
    -H "Content-Type: application/json" \
    -d "{
      \"parent\": {\"database_id\": \"$DATABASE_ID\"},
      \"properties\": {
        \"Task Name\": {\"title\": [{\"text\": {\"content\": \"$name\"}}]},
        \"Status\": {\"select\": {\"name\": \"$task_status\"}},
        \"Priority\": {\"select\": {\"name\": \"$priority\"}},
        \"Phase\": {\"select\": {\"name\": \"$phase\"}},
        \"Module\": {\"select\": {\"name\": \"$module\"}}
      }
    }"
}

# Example usage
create_task "Task Name" "Not Started" "P1-Important" "Phase 1: Design" "Module A"
```

### 2.7 Append Content to Page

```bash
curl -s -X PATCH "https://api.notion.com/v1/blocks/page_id/children" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "children": [
      {"object": "block", "type": "heading_2", "heading_2": {"rich_text": [{"type": "text", "text": {"content": "New Section"}}]}},
      {"object": "block", "type": "to_do", "to_do": {"rich_text": [{"type": "text", "text": {"content": "Todo item"}}], "checked": false}}
    ]
  }'
```

---

## Part 3: PRD Breakdown Best Practices

### 3.1 Task Breakdown Principles

1. **Split by functional modules**: Each module as a separate group
2. **Divide by development phases**: Design → Development → Testing → Release
3. **Priority levels**:
   - P0: Core features, blocking release
   - P1: Important features, planned
   - P2: Optimizations, nice-to-have

### 3.2 Typical PRD Structure Mapping

| PRD Section | Notion Equivalent |
|-------------|-------------------|
| Project Background | Project page overview |
| Product Goals | Success metrics list |
| Functional Requirements | Task database entries |
| Non-functional Requirements | Task acceptance criteria |
| Milestones | Phase divisions |

### 3.3 Task Granularity Guidelines

- Estimated effort per task: 1-3 days
- Task description includes: What to do, acceptance criteria, dependencies
- Avoid oversized tasks, split when necessary

---

## Troubleshooting

### Q1: "Could not find page with ID" error

**Cause**: Page not authorized for the integration

**Solution**: Open the page in Notion → `···` → Connections → Select your integration

### Q2: MCP tools unavailable

**Cause**: MCP tools require a new session after being added

**Solution**: Exit current Claude Code session and restart

### Q3: API returns 401

**Cause**: Invalid or expired token

**Solution**: Obtain a new token and update configuration

---

## Supported Block Types

When creating page content, these block types are available:

| Block Type | Description |
|------------|-------------|
| `paragraph` | Regular text paragraph |
| `heading_1` | H1 heading |
| `heading_2` | H2 heading |
| `heading_3` | H3 heading |
| `bulleted_list_item` | Bullet point |
| `numbered_list_item` | Numbered item |
| `to_do` | Checkbox item |
| `toggle` | Collapsible section |
| `code` | Code block |
| `quote` | Block quote |
| `divider` | Horizontal line |
| `callout` | Callout box |

---

## Database Property Types

When creating databases, these property types are available:

| Property Type | Use Case |
|---------------|----------|
| `title` | Primary name field (required) |
| `select` | Single choice dropdown |
| `multi_select` | Multiple choice tags |
| `status` | Status with groups |
| `date` | Date/datetime picker |
| `people` | User assignment |
| `checkbox` | Boolean toggle |
| `url` | Link field |
| `email` | Email address |
| `phone_number` | Phone number |
| `number` | Numeric value |
| `rich_text` | Text content |
| `relation` | Link to another database |

---

## Complete Workflow Example

```bash
#!/bin/bash

TOKEN="ntn_xxx"
PARENT_PAGE="parent-page-id"

# Step 1: Create project page
PROJECT_RESPONSE=$(curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d "{
    \"parent\": {\"page_id\": \"$PARENT_PAGE\"},
    \"properties\": {
      \"title\": {\"title\": [{\"text\": {\"content\": \"My Project\"}}]}
    }
  }")

PROJECT_ID=$(echo $PROJECT_RESPONSE | jq -r '.id')
echo "Created project page: $PROJECT_ID"

# Step 2: Create task database
DB_RESPONSE=$(curl -s -X POST "https://api.notion.com/v1/databases" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d "{
    \"parent\": {\"page_id\": \"$PROJECT_ID\"},
    \"title\": [{\"type\": \"text\", \"text\": {\"content\": \"Tasks\"}}],
    \"properties\": {
      \"Name\": {\"title\": {}},
      \"Status\": {\"select\": {\"options\": [{\"name\": \"Todo\", \"color\": \"gray\"}]}}
    }
  }")

DB_ID=$(echo $DB_RESPONSE | jq -r '.id')
echo "Created database: $DB_ID"

# Step 3: Add tasks
curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d "{
    \"parent\": {\"database_id\": \"$DB_ID\"},
    \"properties\": {
      \"Name\": {\"title\": [{\"text\": {\"content\": \"First Task\"}}]},
      \"Status\": {\"select\": {\"name\": \"Todo\"}}
    }
  }"

echo "Workflow complete!"
```

---

## References

- [Notion API Documentation](https://developers.notion.com/docs)
- [Notion MCP Server](https://github.com/makenotion/notion-mcp-server)
- [Claude Code MCP Guide](https://docs.anthropic.com/claude-code/mcp)
