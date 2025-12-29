# Cursor IDE Skills System & MCP Integration: Complete Setup SOP

> Install OpenSkills skill system in Cursor IDE, configure Notion MCP server, and enable automated workflows

**Document ID**: 003
**Date**: 2025-12-29
**Tags**: `Cursor IDE` `OpenSkills` `MCP` `Notion` `Workflow`

---

## Overview

This document demonstrates how to configure the OpenSkills skill system and MCP servers in Cursor IDE to enable automated workflows from PRD to tasks:

| Step | Tool/Skill | Description |
|------|------------|-------------|
| 1. Install skill system | OpenSkills CLI | Install skill loader and core skill packages |
| 2. Configure MCP | Notion MCP Server | Connect Cursor to Notion API |
| 3. Generate implementation plan | `/notion-spec-to-implementation` | PRD → Implementation plan → Notion tasks |
| 4. Write progress report | `/internal-comms` | Generate 3P format progress report |
| 5. Knowledge capture | `/notion-knowledge-capture` | Conversation → Notion document |

```
OpenSkills Installation → MCP Configuration → Skill Invocation → Notion Automation
```

**Target Environment**: Cursor IDE (with MCP support)
**Estimated Time**: 30-45 minutes

---

## Part 1: OpenSkills Skill System Installation

### 1.1 What is OpenSkills

OpenSkills is a universal skill loader that provides AI coding assistants with professional domain knowledge and workflows:

- Install pre-built professional skill packages
- Share skills across projects
- Standardize team collaboration methods

### 1.2 Install OpenSkills CLI

```bash
# Global installation
npm install -g openskills

# Verify installation
openskills --version
```

### 1.3 Initialize Project

```bash
# Navigate to project directory
cd /path/to/your/project

# Initialize OpenSkills
openskills init
```

**Expected Result**:
- Creates `.claude/` directory
- Creates `AGENTS.md` file (skill registry)

### 1.4 Install Core Skill Packages

Skills come from two sources:

**1. Anthropic Official Skills** (install from official marketplace):

```bash
# Install from Anthropic official repository (interactive selection)
openskills install anthropics/skills

# Or install specific skills
openskills install anthropics/skills/doc-coauthoring
openskills install anthropics/skills/internal-comms
```

**2. Notion Official Skills** (get from Notion official page):

Visit [Notion Skills for Claude](https://www.notion.so/notiondevs/Notion-Skills-for-Claude-28da4445d27180c7af1df7d8615723d0) to get these skills:

- notion-knowledge-capture - Knowledge capture
- notion-research-documentation - Research documentation
- notion-spec-to-implementation - Spec to implementation

Follow the installation instructions on the Notion official page.

### 1.5 Sync and Verify

```bash
# Sync skills to AGENTS.md
openskills sync

# List all installed skills
openskills list
```

### 1.6 Core Skills Reference

| Skill Name | Source | Purpose |
|------------|--------|---------|
| doc-coauthoring | Anthropic Official | Write technical docs, proposals, specs |
| internal-comms | Anthropic Official | Progress reports, 3P reports |
| notion-knowledge-capture | Notion Official | Convert conversations to Notion docs |
| notion-research-documentation | Notion Official | Search Notion and generate reports |
| notion-spec-to-implementation | Notion Official | PRD → Task list |

**Skill Sources**:
- Anthropic Official: [github.com/anthropics/skills](https://github.com/anthropics/skills)
- Notion Official: [Notion Skills for Claude](https://www.notion.so/notiondevs/Notion-Skills-for-Claude-28da4445d27180c7af1df7d8615723d0)

---

## Part 2: Notion MCP Configuration

### 2.1 Create Notion Integration

1. Visit [Notion Integrations Page](https://www.notion.so/my-integrations)
2. Click **"+ New integration"**
3. Fill in information:
   - **Name**: `Cursor MCP Integration`
   - **Associated workspace**: Select your workspace
   - **Type**: Internal Integration
4. Click **"Submit"**
5. Copy **Internal Integration Token** (format: `ntn_xxxxxx...`)

### 2.2 Authorize Integration Access

**Important**: You must authorize the integration to access Notion pages!

1. Open target Notion page
2. Click **"..."** in top right
3. Select **"Connections"**
4. Search and select `Cursor MCP Integration`
5. Click **"Confirm"**

### 2.3 Configure Cursor MCP

Find MCP configuration file:

```
# macOS/Linux
~/.cursor/mcp.json

# Windows
C:\Users\<username>\.cursor\mcp.json
```

### 2.4 Edit mcp.json

```json
{
  "mcpServers": {
    "notion": {
      "command": "npx",
      "args": [
        "-y",
        "@notionhq/notion-mcp-server"
      ],
      "env": {
        "NOTION_TOKEN": "ntn_YOUR_TOKEN_HERE"
      }
    }
  }
}
```

**Configuration Notes**:
- `command`: Use `npx` to run npm package
- `args`: `-y` auto-confirm, `@notionhq/notion-mcp-server` official MCP package
- `env.NOTION_TOKEN`: Your Notion API Token

### 2.5 Multiple MCP Server Configuration (Optional)

```json
{
  "mcpServers": {
    "notion": {
      "command": "npx",
      "args": ["-y", "@notionhq/notion-mcp-server"],
      "env": {
        "NOTION_TOKEN": "ntn_YOUR_TOKEN_HERE"
      }
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "alchemy": {
      "command": "npx",
      "args": ["-y", "@alchemy/mcp-server"],
      "env": {
        "ALCHEMY_API_KEY": "your-alchemy-api-key"
      }
    }
  }
}
```

### 2.6 Restart and Verify

1. **Completely exit** Cursor (not minimize)
2. Reopen Cursor
3. Wait for MCP initialization (~5-10 seconds)

**Verify connection**: In Cursor chat, type:
```
Search my Notion workspace for pages containing "project"
```

---

## Part 3: Skill Usage in Practice

### 3.1 Using notion-spec-to-implementation

**Scenario**: You have a PRD document and need to generate implementation plan and task list.

**Invocation**:
```
Use notion-spec-to-implementation skill, source【docs/PRD-ProjectName.md】
```

**Skill automatically**:
1. Read and parse PRD document
2. Extract functional and non-functional requirements
3. Generate implementation plan
4. Define acceptance criteria
5. Break down detailed tasks
6. Create Notion pages and tasks

### 3.2 Using internal-comms for Progress Reports

**Invocation**:
```
Use internal-comms skill with【docs/PRD-ProjectName.md】to write a progress report
```

**Output**: 3P format progress report (Progress/Plans/Problems)

### 3.3 Using notion-knowledge-capture

**Invocation**:
```
Use notion-knowledge-capture skill to save our technical discussion to Notion
```

**Skill automatically**:
1. Extract key information from conversation
2. Structure content (background, decisions, rationale)
3. Create Notion page and save

---

## Part 4: Workflow Summary

### 4.1 Complete Workflow

```
PRD Document (Markdown)
    ↓
OpenSkills Skill Parsing
    ↓
Generate Implementation Plan + Task List
    ↓
Via Notion MCP
    ↓
Auto-create Notion Pages and Tasks
```

### 4.2 Skills and Tools Reference

| Step | Skill/Tool | Input | Output |
|------|------------|-------|--------|
| 1 | OpenSkills CLI | GitHub URL | Skill packages |
| 2 | Notion MCP | Token | API connection |
| 3 | notion-spec-to-implementation | PRD | Implementation plan + tasks |
| 4 | internal-comms | Project info | 3P report |
| 5 | notion-knowledge-capture | Conversation | Notion document |

---

## FAQ

### Q1: OpenSkills Installation Failed

**Cause**: npm package name error or network issue

**Solution**:
```bash
# Check npm configuration
npm config get registry

# Reinstall
npm install -g openskills
```

### Q2: Notion MCP Returns "Unauthorized"

**Possible Causes**:
- Incorrect Notion Token
- Integration not authorized to access page
- Insufficient integration permissions

**Solution**:
1. Confirm Token format (starts with `ntn_`)
2. Add integration connection in Notion page
3. Check integration Capabilities permissions

### Q3: MCP Server Package Name Error

**Wrong package name**:
```json
"@modelcontextprotocol/server-notion"  ❌
```

**Correct package name**:
```json
"@notionhq/notion-mcp-server"  ✅
```

### Q4: Skill Not Recognized

**Cause**: Skill not synced to AGENTS.md

**Solution**:
```bash
openskills sync
cat AGENTS.md
```

### Q5: Cursor Not Loading MCP Configuration

**Solution**:
1. Completely exit Cursor
2. Confirm `mcp.json` path is correct
3. Confirm JSON has no syntax errors
4. Restart Cursor

---

## Best Practices

### Security Recommendations

**Don't commit sensitive information**:
```gitignore
# .gitignore
.cursor/mcp.json
.env
.env.local
```

**Use environment variables** (recommended):
```json
{
  "env": {
    "NOTION_TOKEN": "${NOTION_TOKEN}"
  }
}
```

**Principle of least privilege**:
- Only authorize pages that need access
- Only enable necessary Capabilities

### Team Collaboration

Add skill configuration to version control:
```bash
git add .claude/
git add AGENTS.md
git commit -m "Add OpenSkills configuration"
```

---

## References

- [OpenSkills GitHub](https://github.com/numman-ali/openskills)
- [Anthropic Official Skills Repository](https://github.com/anthropics/skills)
- [Notion Skills for Claude](https://www.notion.so/notiondevs/Notion-Skills-for-Claude-28da4445d27180c7af1df7d8615723d0)
- [Cursor MCP Documentation](https://docs.cursor.com/zh-Hant/tools/mcp)
- [Notion API Documentation](https://developers.notion.com/)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [MCP Servers List](https://github.com/modelcontextprotocol/servers)
