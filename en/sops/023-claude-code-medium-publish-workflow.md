# From Local Docs to Medium Publishing: Claude Code Automation Workflow

> Complete workflow for automating technical document translation and Medium publishing using Claude Code + Chrome browser automation

**Document ID**: 023
**Date**: 2025-01-01
**Tags**: `Claude Code` `Chrome MCP` `Medium` `Automation` `Content Workflow`

---

## Overview

This document records the automated workflow for publishing from local Markdown documents to Medium platform using Claude Code with Chrome MCP extension. The entire process takes about 10 minutes, with Chrome automation operations taking only about 5 minutes.

### Core Value

- **Efficiency**: From manual copy-paste to fully automated publishing
- **Cross-language Support**: Automatic translation from Chinese to English
- **Browser Automation**: Leverages existing Chrome session, no API required

---

## 1. Prerequisites

### 1.1 Environment Requirements

| Component | Requirement | Notes |
|-----------|-------------|-------|
| Claude Code | v2.0.73+ | CLI tool |
| Chrome Browser | Latest | Claude in Chrome extension required |
| Medium Account | Logged in | Maintain login state in Chrome |

### 1.2 Chrome MCP Connection

```bash
# Start Claude Code with Chrome integration
claude --chrome

# Or check connection status in session
/chrome
```

**Connection Verification**:
```
mcp__claude-in-chrome__tabs_context_mcp(createIfEmpty: true)
```

---

## 2. Workflow

### 2.1 Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│              Claude Code Automated Publishing Flow               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐  │
│  │ Read     │ -> │ Translate│ -> │ Chrome   │ -> │ Publish  │  │
│  │ Source   │    │ Generate │    │ Auto-fill│    │ Confirm  │  │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘  │
│       |              |               |               |         │
│    Markdown       Write           MCP Ops         User OK      │
│    ~1 min        ~3 min          ~5 min          ~1 min       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Execution Steps

#### Step 1: Connect Chrome Browser

```bash
# Claude Code execution
mcp__claude-in-chrome__tabs_context_mcp(createIfEmpty: true)

# Result: Successfully connected, retrieved current tab info
```

#### Step 2: Read Source Document

```bash
# Search for document
Glob("**/*target-document*")

# Read content
Read("/path/to/source-document.md")
```

#### Step 3: Generate English Version

Claude Code automatically:
- Translates core content
- Adapts for English readers
- Optimizes article structure
- Adds Introduction/Conclusion

#### Step 4: Navigate to Medium

```bash
mcp__claude-in-chrome__navigate(
  url: "https://medium.com/new-story",
  tabId: <current_tab_id>
)
```

#### Step 5: Input Content

**Operation Sequence**:
1. Enter title
2. Enter subtitle
3. Input body in sections
4. Add code blocks

```bash
# Input operation
mcp__claude-in-chrome__computer(
  action: "type",
  text: "Article content..."
)
```

#### Step 6: Publish

```bash
# Click Publish button
mcp__claude-in-chrome__computer(
  action: "click",
  coordinate: [x, y]
)

# Add topic tags
# Click "Publish now" after user confirmation
```

---

## 3. Time Statistics

| Phase | Duration | Notes |
|-------|----------|-------|
| Connection & Setup | ~1 min | Chrome MCP connection |
| Document Read & Translation | ~4 min | Generate English version |
| Chrome Automation | ~5 min | Navigate, input, publish |
| **Total** | **~10 min** | End-to-end completion |

---

## 4. Tool Chain

```
Claude Code
    |
    +-- Glob (file search)
    +-- Read (read document)
    +-- Write (generate English version)
    +-- Chrome MCP
          |
          +-- tabs_context_mcp (browser connection)
          +-- navigate (page navigation)
          +-- read_page (read page elements)
          +-- computer (click/type/scroll)
          +-- screenshot (capture confirmation)
```

---

## 5. Best Practices

### 5.1 Success Factors

1. **Keep Chrome Logged In**: Ensure Medium account is logged in
2. **Input Content in Sections**: Split long articles to avoid timeout
3. **User Confirmation**: Get user approval before publishing

### 5.2 Considerations

- Medium code blocks may need manual formatting
- Cover image upload requires additional steps
- Prepare topic tags in advance

### 5.3 Extensible Scenarios

- Batch publish to multiple platforms
- Scheduled publishing
- Auto-add SEO optimized content

---

## 6. FAQ

### Q1: Chrome MCP Connection Failed?

Check:
1. Claude in Chrome extension version >= 1.0.36
2. Chrome browser is running
3. Try `/chrome` command to reconnect

### Q2: Medium Requires Login?

Ensure Medium account is logged in within Chrome. MCP reuses existing session.

### Q3: Content Input Interrupted?

For long articles, input in sections, each section under 500 words.

---

## References

- [Claude Code Official Documentation](https://docs.anthropic.com/claude-code)
- [Chrome MCP Integration Guide](https://docs.anthropic.com/claude-code/chrome)
- [Medium Writing Guide](https://help.medium.com/hc/en-us/categories/201931128-Publishing)
