# WeChat Article to Notion Auto-Practice Workflow

> From reading to doing: AI automatically simulates author's operations, building knowledge base while learning by doing

**Document ID**: 007
**Date**: 2025-12-29
**Tags**: `WeChat` `Notion` `Claude Code` `MCP` `Automation` `Knowledge Management`

---

## Overview

When you find a technical article on WeChat and want to practice immediately, the traditional approach requires:
1. Read article → 2. Manually copy commands → 3. Execute step by step → 4. Go back when encountering issues

This workflow lets **Claude Code automatically simulate all author's operations**, achieving:
- **Instant Practice** — Article content automatically converted to executable steps
- **Knowledge Preservation** — Articles automatically archived to Notion knowledge base
- **Skill Solidification** — Practical experience can be converted into reusable Skills

---

## Workflow Architecture

```
WeChat Article
      ↓
  Safari Browser (open externally)
      ↓
  Share to Notion (clip)
      ↓
  Copy Notion Link
      ↓
  Claude Code CLI
      ↓
  MCP reads Notion content
      ↓
  AI analyzes and auto-executes installation/configuration
      ↓
  ✅ Practice complete + Knowledge preserved
```

---

## Part 1: Environment Setup

### 1.1 Required Tools

| Tool | Purpose | Check Installation |
|------|---------|-------------------|
| Claude Code CLI | AI programming assistant | `claude --version` |
| Notion MCP | Read Notion content | `claude mcp list` |
| Notion Account | Knowledge base storage | - |
| Safari/Chrome | External browser | - |

### 1.2 Notion MCP Configuration

If Notion MCP is not configured, execute:

```bash
# Add Notion MCP server
claude mcp add notion npx -y @notionhq/notion-mcp-server

# Need to set NOTION_TOKEN environment variable
# Get it from: Notion Settings → Connections → Develop your own integration
```

For detailed configuration, see [003 - Cursor IDE Skills & MCP Setup](./003-cursor-openskills-mcp-setup.md)

---

## Part 2: Complete Operation Flow

### Step 1: WeChat Article → External Browser

1. Open article in WeChat Official Account
2. Tap `···` menu in top right corner
3. Select **"Open in Default Browser"** or **"Open in Safari"**

> **Why use external browser?**
> - WeChat's built-in browser cannot share directly to Notion
> - Safari/Chrome support Notion Web Clipper extension

### Step 2: Browser → Notion

**Method A: Using Notion Web Clipper (Recommended)**

1. Install [Notion Web Clipper](https://www.notion.so/web-clipper) browser extension
2. Click extension icon on the article page
3. Select save location (recommend creating a dedicated "Article Collection" database)
4. Click **Save page**

**Method B: Using iOS/macOS Share Menu**

1. Tap Safari share button
2. Select **Notion**
3. Choose save location and confirm

### Step 3: Get Notion Link

1. Open Notion, find the saved article
2. Click `···` in top right → **Copy link**
3. Or copy directly from browser address bar

Link format example:
```
https://www.notion.so/Article-Title-2d83f9a673f781d0bb56ed599d8905cb
```

### Step 4: Claude Code Auto-Practice

Send Notion link to Claude Code:

```bash
# Start Claude Code
claude

# Send request
> Help me install the tool from this article: https://www.notion.so/xxx
```

Claude Code will automatically:
1. **Call Notion MCP** to read article content
2. **Analyze article structure** to extract installation steps
3. **Execute installation commands** simulating author's operations
4. **Handle exceptions** automatically resolving dependency issues
5. **Verify installation** confirming functionality works

### Step 5: Knowledge Solidification (Optional)

After practice, you can further:

**A. Create Skill**
```
> Package the installation steps into a skill
```

**B. Write to Blueprint**
```
> Write an SOP document for this tool's usage
```

---

## Part 3: Real Case Study

### Case: Installing claude-mem Memory Plugin

**Original Article**: "Say Goodbye to Goldfish Memory! The Best Memory Plugin for Claude Code"

**Operation Process**:

1. Open article in WeChat → Open in Safari
2. Share to Notion knowledge base
3. Copy Notion link to Claude Code:
   ```
   > Help me install this: https://www.notion.so/Claude-Code-2d83f9a673f781d0bb56ed599d8905cb
   ```

4. Claude Code auto-executes:
   ```bash
   # 1. Read Notion page content
   # 2. Extract installation commands
   claude plugin marketplace add thedotmack/claude-mem
   claude plugin install claude-mem
   # 3. Verify installation success
   ```

5. Result:
   ```
   ✅ claude-mem installed successfully
   ├─ marketplace: thedotmack (added)
   └─ plugin: claude-mem@thedotmack (scope: user)
   ```

---

## Part 4: Advanced Tips

### 4.1 Batch Processing Articles

If you have multiple articles to practice:

```
> I have a "To Practice" database in Notion, help me install tools from each one
```

### 4.2 Auto-Create Skills

Automatically package into reusable skills after practice:

```
> Package the {tool name} installation steps into a skill for one-click installation later
```

### 4.3 Integration with Task Management

Combined with task-manager skill:

```
> Add task: Practice claude-mem memory plugin
```

---

## FAQ

### Q1: Notion MCP Cannot Read Page Content

**Cause**: Notion Integration doesn't have page access permission

**Solution**:
1. Open the Notion page
2. Click `···` in top right → **Connections**
3. Add your Integration

### Q2: Article Too Long Causing Parse Failure

**Solution**: Have Claude Code process in segments

```
> First extract the installation steps from this article, then execute
```

### Q3: Installation Commands Outdated or Incorrect

**Solution**: Claude Code will automatically:
- Check GitHub repo for latest version
- Find official documentation to verify
- Prompt user confirmation before executing

---

## Workflow Advantages

| Traditional Approach | This Workflow |
|---------------------|---------------|
| Manually copy-paste commands | AI auto-extracts and executes |
| Debug errors yourself | AI auto-handles exceptions |
| Knowledge scattered everywhere | Unified in Notion |
| Relearn every time | Practice once, package as Skill |

---

## Best Practices

### Do's

- Create a dedicated "Tech Articles" database in Notion for easy management
- Write notes or create Skills immediately after practice, while it's fresh
- Keep original article links for future reference

### Don'ts

- Don't auto-execute installation commands in untrusted environments
- Don't skip verification steps, ensure installation succeeds
- Don't ignore prerequisite requirements mentioned in articles

---

## References

- [Notion Web Clipper](https://www.notion.so/web-clipper)
- [Notion MCP Server](https://github.com/notionhq/notion-mcp-server)
- [003 - Cursor IDE Skills & MCP Setup](./003-cursor-openskills-mcp-setup.md)
- [006 - Claude Skill Creation SOP](./006-claude-skill-creation-sop.md)
