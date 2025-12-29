# WeChat Official Account Auto-Publish: MCP Playwright Automation SOP

> Automate article publishing to WeChat Official Account using MCP Playwright with session persistence and login detection

**Document ID**: 014
**Date**: 2025-12-29
**Tags**: `MCP` `Playwright` `Automation` `WeChat` `Content Publishing`

---

## Overview

This document details how to automate publishing Markdown articles to WeChat Official Account (微信公众号) platform using MCP Playwright, including complete workflow, Claude Code Skill implementation, and troubleshooting.

### Key Benefits

- **One-click publish**: Say "publish to WeChat" to complete the entire flow
- **Session persistence**: Login once, stay logged in
- **Smart detection**: Auto-detect login status, prompt only when needed
- **Format preservation**: Perfect content rendering

### Comparison with Juejin Publishing

| Feature | WeChat Official Account | Juejin |
|---------|------------------------|--------|
| Image requirement | ✅ At least 1 required | ❌ Optional |
| Cover image | Optional (defaults to first image) | ✅ Required |
| Category/Tags | No category system | ✅ Required |
| Publish verification | Admin QR scan needed | Direct publish |
| Editor type | Rich text (not CodeMirror) | CodeMirror |

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
ln -s ../library/wechat-mp-publisher .

# Or create directly
mkdir -p ~/.claude/skills/wechat-mp-publisher
# Copy SKILL.md content
```

### 1.3 Image Library Preparation

**Important**: WeChat Official Account requires at least one image in articles for publishing.

Recommended: Upload some general-purpose images to the image library in advance:
- Technical article illustrations
- Logo or brand images
- General cover images

## Part 2: Complete Workflow

### 2.1 Flow Diagram

```
User: Publish to WeChat
        ↓
Read Markdown file
        ↓
Open WeChat editor
        ↓
Check login status ──→ Not logged in → Prompt QR scan → Wait
        ↓ Logged in
Handle popups (store commission, quick format, etc.)
        ↓
Fill title
        ↓
Fill body content
        ↓
Add image from library (required)
        ↓
Set cover (optional)
        ↓
Click publish button
        ↓
Confirm notification settings
        ↓
Admin QR verification
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
const content = bodyWithoutTitle // Recommend under 1000 chars
```

#### Step 2: Open Editor

```javascript
// Navigate to editor
mcp__plugin_playwright_playwright__browser_navigate({
  url: "https://mp.weixin.qq.com/cgi-bin/appmsg?t=media/appmsg_edit&action=edit&type=77&lang=zh_CN"
})
```

**Login status check**:
- If QR scan page shows → Need login
- If editor loads → Already logged in, continue

#### Step 3: Wait for Editor Load

WeChat editor loads slowly, need to wait:

```javascript
mcp__plugin_playwright_playwright__browser_wait_for({
  text: "请在这里输入标题",
  time: 5
})
```

#### Step 4: Handle Popups

WeChat editor may show multiple prompts:

```javascript
// Store commission popup
mcp__plugin_playwright_playwright__browser_click({
  element: "稍后再说",
  ref: "laterRef"
})

// Quick format tooltip
mcp__plugin_playwright_playwright__browser_click({
  element: "我知道了",
  ref: "gotItRef"
})
```

#### Step 5: Fill Title

```javascript
mcp__plugin_playwright_playwright__browser_type({
  element: "Title input",
  ref: "titleRef",
  text: "Article Title"
})
```

#### Step 6: Fill Body Content

WeChat editor is not CodeMirror, use keyboard.type:

```javascript
mcp__plugin_playwright_playwright__browser_run_code({
  code: `async (page) => {
    // Click edit area to focus
    await page.click('[data-placeholder="请在这里输入正文"]');
    await page.waitForTimeout(500);

    const content = \`Article content here...\`;

    // Type character by character to avoid timeout
    await page.keyboard.type(content, { delay: 1 });
    return 'Content typed';
  }`
})
```

**Notes**:
- Content should not be too long, recommend under 1000 characters
- `delay: 1` parameter avoids issues from typing too fast

#### Step 7: Add Image (Required)

WeChat Official Account **requires** at least one image to publish:

```javascript
// Click image menu
mcp__plugin_playwright_playwright__browser_click({
  element: "图片",
  ref: "imageMenuRef"
})

// Select "From image library"
mcp__plugin_playwright_playwright__browser_click({
  element: "从图片库选择",
  ref: "fromLibraryRef"
})

// Wait for library to load
mcp__plugin_playwright_playwright__browser_wait_for({ time: 2 })

// Select first image
mcp__plugin_playwright_playwright__browser_click({
  element: "Image thumbnail",
  ref: "imageRef"
})

// Confirm
mcp__plugin_playwright_playwright__browser_click({
  element: "确定",
  ref: "confirmRef"
})
```

#### Step 8: Set Cover (Optional)

Select cover from body image:

```javascript
// Click cover area
mcp__plugin_playwright_playwright__browser_click({
  element: "拖拽或选择封面",
  ref: "coverAreaRef"
})

// Select "From content"
mcp__plugin_playwright_playwright__browser_click({
  element: "从正文选择",
  ref: "fromContentRef"
})

// Select image → Next → Confirm
// Aspect ratios:
// 2.35:1 for message list
// 1:1 for share cards
```

#### Step 9: Publish

```javascript
// Click publish button
mcp__plugin_playwright_playwright__browser_click({
  element: "发表",
  ref: "publishRef"
})

// Publish options dialog
// - Notification (default on)
// - Group notification
// - Scheduled publish

// Click publish in dialog
mcp__plugin_playwright_playwright__browser_click({
  element: "发表",
  ref: "dialogPublishRef"
})

// Confirm continue
mcp__plugin_playwright_playwright__browser_click({
  element: "继续发表",
  ref: "continueRef"
})
```

#### Step 10: WeChat Verification

Final step requires admin QR scan:

```
Prompt user:
"Please scan the QR code on the page with WeChat to verify.
Admin and operator accounts can verify directly.
Non-admin scans require admin approval."
```

After verification, article publishes automatically.

## Part 3: Claude Code Skill Implementation

### 3.1 Skill Directory Structure

```
wechat-mp-publisher/
└── SKILL.md    # Skill definition file
```

### 3.2 Complete SKILL.md

```yaml
---
name: wechat-mp-publisher
description: Auto-publish articles to WeChat Official Account using MCP Playwright. Triggers on "publish to WeChat", "WeChat publish", "公众号发布". Supports title/content filling, image insertion, cover setup, and full publish flow. Auto-detects login status with session persistence. Requires admin QR verification for final publish.
---

# WeChat Official Account Publisher

Automate Markdown article publishing to WeChat Official Account platform.

## Prerequisites

- MCP Playwright service running
- WeChat Official Account
- Images in the platform's image library

## Session Persistence

MCP Playwright maintains browser session by default. Login once, no repeated logins needed.

## Publishing Requirements

| Field | Required | Notes |
|-------|----------|-------|
| Title | ✅ | Max 64 chars |
| Content | ✅ | Must contain at least 1 image |
| Cover | ❌ | Optional, defaults to first image |
| Summary | ❌ | Auto-generated from content, max 120 chars |

## Usage Example

Publish to WeChat: docs/my-article.md
```

### 3.3 Enable Skill

```bash
# Using skill library
cd ~/Projects/my_skils/enabled
ln -s ../library/wechat-mp-publisher .

# Verify
ls -la ~/.claude/skills/ | grep wechat
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
  url: "https://mp.weixin.qq.com/cgi-bin/appmsg?t=media/appmsg_edit&action=edit&type=77&lang=zh_CN"
})

// 3. Check page
// If editor shows → Login valid
// If QR scan page → Need re-login
```

### 4.3 Login Expiry Handling

When login expires, skill auto-prompts:

```
WeChat Official Account login expired. Please login in browser.
Method: WeChat QR scan
Say "logged in" when done.
```

## Part 5: WeChat Official Account Features

### 5.1 Editor Characteristics

- Uses **rich text editor** (not CodeMirror)
- Supports direct image paste
- Auto-saves drafts
- Supports multi-article messages

### 5.2 Publishing Requirements

| Field | Required | Notes |
|-------|----------|-------|
| Title | ✅ | Max 64 chars |
| Content | ✅ | Must contain at least 1 image |
| Cover | ❌ | Optional, defaults to first image |
| Summary | ❌ | Auto-generated, max 120 chars |
| Original declaration | ❌ | Default off |
| Comments | ❌ | Default on |

### 5.3 Cover Dimensions

| Ratio | Usage |
|-------|-------|
| 2.35:1 | Message list (subscription feed) |
| 1:1 | Share cards, account homepage |

### 5.4 Broadcast Limits

- Subscription accounts: 1 broadcast per day
- Service accounts: 4 broadcasts per month

## FAQ

### Q1: "Must insert at least one image"

**Problem**: Error when publishing without image

**Solution**: Ensure at least one image is added from the image library before publishing. WeChat Official Account articles require images.

### Q2: Editor Loads Slowly

**Problem**: Elements not found, timeout errors

**Solution**:
- Increase `wait_for` time
- Get multiple snapshots to confirm load
- Use `browser_wait_for({ text: "请在这里输入标题" })`

### Q3: Click Intercepted by Popup

**Problem**: Clicks fail due to popup overlay

**Solution**: Close all popups first:
- "小店返佣商品" → Click "稍后再说"
- "一键排版" tooltip → Click "我知道了"
- Other prompts → Click close button

### Q4: Content Not Fully Typed

**Problem**: keyboard.type timeout or truncation

**Solution**:
- Simplify content (recommend under 1000 chars)
- Increase `delay` parameter (e.g., `delay: 1`)
- Split long content into segments

### Q5: Admin Verification Required

**Problem**: QR verification dialog appears

**Explanation**: This is WeChat's security mechanism, not an error. Requires:
- Admin or operator to scan QR code
- Non-admin scans need admin approval

## References

- [MCP Playwright Documentation](https://github.com/anthropics/claude-code)
- [Claude Code Skills Guide](https://docs.anthropic.com/claude-code/skills)
- [WeChat Official Account Platform](https://mp.weixin.qq.com/)
