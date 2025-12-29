# Zhihu Auto-Publish: MCP Playwright Automation SOP

> Automate article publishing to Zhihu using MCP Playwright with session persistence and login detection

**Document ID**: 015
**Date**: 2025-12-29
**Tags**: `MCP` `Playwright` `Automation` `Zhihu` `Content Publishing`

---

## Overview

This document details how to automate publishing Markdown articles to Zhihu (知乎) platform using MCP Playwright, including complete workflow, Claude Code Skill implementation, and troubleshooting.

### Key Benefits

- **One-click publish**: Say "publish to Zhihu" to complete the entire flow
- **Session persistence**: Login once, stay logged in
- **Smart detection**: Auto-detect login status, prompt only when needed
- **Markdown support**: Zhihu natively supports Markdown syntax

### Comparison with Other Platforms

| Feature | Zhihu | Juejin | WeChat MP |
|---------|-------|--------|-----------|
| Image requirement | Optional | Optional | Required (at least 1) |
| Topics/Tags | Optional | Required | None |
| Category selection | None | Required | None |
| Publish verification | None | None | Admin QR scan |
| Editor type | Rich text | CodeMirror | Rich text |
| Markdown | Native support | Native support | Not supported |

## Part 1: Prerequisites

### 1.1 Environment Setup

```bash
# Verify MCP Playwright is installed
claude mcp list | grep playwright

# Should show:
# playwright@claude-plugins-official: connected
```

### 1.2 Skill Installation

```bash
# Using skill library
cd ~/Projects/my_skils/enabled
ln -s ../library/zhihu-publisher .

# Or create directly
mkdir -p ~/.claude/skills/zhihu-publisher
# Copy SKILL.md content
```

## Part 2: Complete Workflow

### 2.1 Flow Diagram

```
User: Publish to Zhihu
        ↓
Read Markdown file
        ↓
Open Zhihu editor
        ↓
Check login status ──→ Not logged in → Prompt QR scan → Wait
        ↓ Logged in
Fill title
        ↓
Fill body content
        ↓
Add topic tags (optional)
        ↓
Click publish button
        ↓
Published successfully
```

### 2.2 Detailed Steps

#### Step 1: Read Article

```javascript
// Read Markdown file
Read(filePath)

// Extract content
const title = firstLine // # Title
const content = bodyWithoutTitle
```

#### Step 2: Open Editor

```javascript
// Navigate to editor
mcp__plugin_playwright_playwright__browser_navigate({
  url: "https://zhuanlan.zhihu.com/write"
})
```

**Login status check**:
- If URL redirects to login page → Need login
- If editor loads → Already logged in, continue

#### Step 3: Fill Title

```javascript
mcp__plugin_playwright_playwright__browser_type({
  element: "Title input",
  ref: "titleRef",
  text: "Article Title"
})
```

#### Step 4: Fill Body Content

Zhihu editor supports Markdown, use keyboard.type:

```javascript
mcp__plugin_playwright_playwright__browser_click({
  element: "Content input area",
  ref: "contentRef"
})

mcp__plugin_playwright_playwright__browser_run_code({
  code: `async (page) => {
    const content = \`Article content...\`;
    await page.keyboard.type(content, { delay: 1 });
    return 'Content typed';
  }`
})
```

#### Step 5: Add Topics (Optional)

```javascript
// Click add topic
mcp__plugin_playwright_playwright__browser_click({
  element: "Add topic",
  ref: "addTopicRef"
})

// Search topic
mcp__plugin_playwright_playwright__browser_type({
  element: "Search topic",
  ref: "searchRef",
  text: "Claude"
})

// Select topic
mcp__plugin_playwright_playwright__browser_click({
  element: "Topic option",
  ref: "topicRef"
})
```

#### Step 6: Publish

```javascript
// Click publish button
mcp__plugin_playwright_playwright__browser_click({
  element: "Publish",
  ref: "publishRef"
})
```

After successful publish, page redirects to the article.

## Part 3: Claude Code Skill Implementation

### 3.1 Skill Directory Structure

```
zhihu-publisher/
└── SKILL.md    # Skill definition file
```

### 3.2 Complete SKILL.md

```yaml
---
name: zhihu-publisher
description: Auto-publish articles to Zhihu using MCP Playwright. Triggers on "publish to Zhihu", "zhihu publish", "知乎发文". Supports title/content filling, topic tags, and full publish flow. Auto-detects login status with session persistence.
---

# Zhihu Article Publisher

Automate Markdown article publishing to Zhihu platform.

## Prerequisites

- MCP Playwright service running
- Zhihu account

## Publishing Requirements

| Field | Required | Notes |
|-------|----------|-------|
| Title | Yes | Max 100 chars |
| Content | Yes | Markdown supported |
| Topics | No | Up to 5 tags |
| Cover | No | Optional image |

## Usage Example

Publish to Zhihu: docs/my-article.md
```

### 3.3 Enable Skill

```bash
# Using skill library
cd ~/Projects/my_skils/enabled
ln -s ../library/zhihu-publisher .

# Verify
ls -la ~/.claude/skills/ | grep zhihu
```

## Part 4: Session Persistence Mechanism

### 4.1 How It Works

MCP Playwright uses persistent browser Profile, storing login info in:
- Cookies
- LocalStorage
- SessionStorage

Session state persists after browser close/reopen.

### 4.2 Verification Method

```javascript
// 1. Close browser
mcp__plugin_playwright_playwright__browser_close()

// 2. Reopen editor
mcp__plugin_playwright_playwright__browser_navigate({
  url: "https://zhuanlan.zhihu.com/write"
})

// 3. Check page
// If editor shows → Login valid
// If login page → Need re-login
```

### 4.3 Login Expiry Handling

When login expires, skill auto-prompts:

```
Zhihu login expired. Please login in browser.
Methods: SMS verification, password, WeChat/QQ scan
Say "logged in" when done.
```

## Part 5: Zhihu Platform Features

### 5.1 Editor Characteristics

- Uses **rich text editor**
- Native Markdown syntax support
- Auto-saves drafts
- Code block syntax highlighting
- No mandatory image requirement

### 5.2 Publishing Requirements

| Field | Required | Notes |
|-------|----------|-------|
| Title | Yes | Max 100 chars |
| Content | Yes | No minimum length |
| Topics | No | Up to 5 |
| Cover | No | Optional |
| Creation statement | No | Optional (original/repost) |

### 5.3 Zhihu Features

- **Draft backup**: Auto-save with version history
- **Creation assistant**: Content check, smart formatting, title suggestions
- **Submit to question**: Link article to relevant questions
- **Gift settings**: Enable/disable reader tips

## FAQ

### Q1: QR Code Not Loading

**Problem**: QR code doesn't display or scan doesn't work

**Solution**:
- Refresh page to reload QR code
- Try password login instead
- Check network connection

### Q2: Markdown Table Not Rendering

**Problem**: Table shows as raw text

**Note**: Zhihu editor has limited table preview. Tables render correctly after publishing. Check the published article for actual formatting.

### Q3: Topic Search No Results

**Problem**: No matching topics found

**Solution**:
- Use more general keywords
- Check spelling
- Try related topics

### Q4: Publish Button Disabled

**Problem**: Publish button is not clickable

**Cause**: Title or content is empty

**Solution**: Ensure both title and content are filled.

## References

- [MCP Playwright Documentation](https://github.com/anthropics/claude-code)
- [Claude Code Skills Guide](https://docs.anthropic.com/claude-code/skills)
- [Zhihu](https://zhuanlan.zhihu.com/)
