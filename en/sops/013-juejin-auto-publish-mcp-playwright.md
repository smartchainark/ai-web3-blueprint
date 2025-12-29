# Juejin Auto-Publish: MCP Playwright Automation SOP

> Automate article publishing to Juejin using MCP Playwright with session persistence and login detection

**Document ID**: 013
**Date**: 2025-12-29
**Tags**: `MCP` `Playwright` `Automation` `Juejin` `Content Publishing`

---

## Overview

This document details how to automate publishing Markdown articles to Juejin (掘金) platform using MCP Playwright, including complete workflow, Claude Code Skill implementation, and troubleshooting.

### Key Benefits

- **One-click publish**: Say "publish to Juejin" to complete the entire flow
- **Session persistence**: Login once, stay logged in
- **Smart detection**: Auto-detect login status, prompt only when needed
- **Format preservation**: Perfect Markdown rendering

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
ln -s ../library/juejin-publisher .

# Or create directly
mkdir -p ~/.claude/skills/juejin-publisher
# Copy SKILL.md content
```

## Part 2: Complete Workflow

### 2.1 Flow Diagram

```
User: Publish to Juejin
        ↓
Read Markdown file
        ↓
Open Juejin editor
        ↓
Check login status ──→ Not logged in → Prompt user → Wait confirmation
        ↓ Logged in
Fill title and content
        ↓
Click publish button
        ↓
Select category + Add tags
        ↓
Confirm publish
        ↓
Return article URL
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

#### Step 2: Open Editor and Check Login

```javascript
// Navigate to editor
mcp__plugin_playwright_playwright__browser_navigate({
  url: "https://juejin.cn/editor/drafts/new"
})

// Get page snapshot
mcp__plugin_playwright_playwright__browser_snapshot()

// Login status check
if (pageUrl.includes('/login')) {
  // Need login
  prompt: "Login expired. Please login manually and say 'logged in'"
} else if (pageUrl.includes('/editor/drafts')) {
  // Already logged in, continue
}
```

#### Step 3: Fill Content

```javascript
// Fill title
mcp__plugin_playwright_playwright__browser_type({
  element: "Article title input",
  ref: "e6",  // Get actual ref from snapshot
  text: "Article Title"
})

// Fill body (CodeMirror editor)
mcp__plugin_playwright_playwright__browser_run_code({
  code: `async (page) => {
    await page.click('.CodeMirror-scroll');
    await page.waitForTimeout(500);
    await page.evaluate((text) => {
      const cm = document.querySelector('.CodeMirror').CodeMirror;
      cm.setValue(text);
    }, content);
    return 'Content filled';
  }`
})
```

#### Step 4: Publish Settings

```javascript
// Click publish button
mcp__plugin_playwright_playwright__browser_click({
  element: "Publish button",
  ref: "publishButtonRef"
})

// Select category
mcp__plugin_playwright_playwright__browser_click({
  element: "开发工具",  // Development Tools
  ref: "categoryRef"
})

// Add tag
mcp__plugin_playwright_playwright__browser_run_code({
  code: `async (page) => {
    await page.click('text=请搜索添加标签');
    await page.waitForTimeout(500);
    await page.keyboard.type('Claude');
    await page.waitForTimeout(1000);
  }`
})

// Select tag from results
mcp__plugin_playwright_playwright__browser_click({
  element: "Claude tag",
  ref: "tagRef"
})
```

#### Step 5: Confirm Publish

```javascript
// Click confirm publish
mcp__plugin_playwright_playwright__browser_click({
  element: "Confirm publish button",
  ref: "confirmRef"
})

// Get article URL after success
// Page redirects to https://juejin.cn/post/{article_id}
```

## Part 3: Claude Code Skill Implementation

### 3.1 Skill Directory Structure

```
juejin-publisher/
└── SKILL.md    # Skill definition file
```

### 3.2 Complete SKILL.md

```yaml
---
name: juejin-publisher
description: Auto-publish Markdown articles to Juejin using MCP Playwright. Triggers on "publish to Juejin", "juejin publish". Supports title/content filling, category selection, tag addition, and full publish flow. Auto-detects login status with session persistence.
---

# Juejin Article Publisher

Automate Markdown article publishing to Juejin platform using MCP Playwright.

## Prerequisites

- MCP Playwright service running
- Juejin account

## Session Persistence

MCP Playwright maintains browser session by default. Login once, no repeated logins needed.

## Workflow

[See Part 2 for full flow]

## Categories

- Backend, Frontend, Android, iOS
- AI, Development Tools
- Code Life, Reading

## Tag Recommendations

- AI/Claude → Claude, AI, LLM
- Frontend → React, Vue, TypeScript
- Backend → Node.js, Python, Go

## Usage Example

Publish to Juejin: docs/my-article.md
Category: 开发工具
Tag: Claude
```

### 3.3 Enable Skill

```bash
# Using skill library
cd ~/Projects/my_skils/enabled
ln -s ../library/juejin-publisher .

# Verify
ls -la ~/.claude/skills/ | grep juejin
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
  url: "https://juejin.cn/editor/drafts/new"
})

// 3. Check URL
// If still /editor/drafts/new → Login valid
// If redirects to /login → Need re-login
```

### 4.3 Login Expiry Handling

When login expires, skill auto-prompts:

```
Juejin login expired. Please login in browser.
Methods: Phone verification / WeChat QR / GitHub
Say "logged in" when done.
```

## Part 5: Juejin Platform Features

### 5.1 Editor Characteristics

- Uses **CodeMirror** rich text editor
- Supports **Markdown** live preview
- Auto-saves drafts

### 5.2 Publish Requirements

| Field | Required | Notes |
|-------|----------|-------|
| Title | ✅ | Max 100 chars |
| Content | ✅ | Markdown supported |
| Category | ✅ | Choose 1 of 8 |
| Tag | ✅ | Min 1, Max 1 |
| Summary | ❌ | Auto-generated |
| Cover | ❌ | Optional upload |

### 5.3 Category Guide

| Category | Suitable Content |
|----------|------------------|
| Backend (后端) | Server dev, DB, API |
| Frontend (前端) | Web/Mobile UI, frameworks |
| Android | Android development |
| iOS | Apple development |
| AI (人工智能) | AI, ML, LLM |
| Dev Tools (开发工具) | IDE, CLI, productivity |
| Code Life (代码人生) | Career, experience sharing |
| Reading (阅读) | Book reviews, notes |

## FAQ

### Q1: CodeMirror Click Timeout

**Problem**: Direct click on editor input times out

**Solution**: Use `browser_run_code` with JavaScript:

```javascript
await page.evaluate((text) => {
  document.querySelector('.CodeMirror').CodeMirror.setValue(text);
}, content);
```

### Q2: "Add at least one tag" Error

**Problem**: Tag required error on publish

**Solution**: Ensure tag is added before publish. Add tag step in workflow.

### Q3: Frequent Login Expiry

**Possible causes**:
- Browser profile cleaned
- Cookie expired
- Account logged in elsewhere

**Suggestion**: Periodically test login status, manually re-login when expired.

## References

- [MCP Playwright Documentation](https://github.com/anthropics/claude-code)
- [Claude Code Skills Guide](https://docs.anthropic.com/claude-code/skills)
- [Juejin Creator Center](https://juejin.cn/creator)
