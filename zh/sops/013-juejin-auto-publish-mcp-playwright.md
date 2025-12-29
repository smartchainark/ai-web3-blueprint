# 掘金文章自动发布：MCP Playwright 自动化 SOP

> 使用 MCP Playwright 实现掘金文章一键发布，支持会话持久化和登录检测

**文档编号**: 013
**日期**: 2025-12-29
**标签**: `MCP` `Playwright` `自动化` `掘金` `内容发布`

---

## 概述

本文档详细介绍如何使用 MCP Playwright 自动化发布 Markdown 文章到掘金平台，包括完整的操作流程、Claude Code Skill 实现和常见问题处理。

### 核心价值

- **一键发布**：说"发布到掘金"即可完成全流程
- **会话持久化**：登录一次，长期有效
- **智能检测**：自动识别登录状态，按需提醒
- **格式保留**：Markdown 完美渲染

## 第一部分：前置要求

### 1.1 环境准备

```bash
# 确认 MCP Playwright 已安装
claude mcp list | grep playwright

# 应显示类似：
# playwright@claude-plugins-official: connected
```

### 1.2 技能安装

```bash
# 如果使用技能库管理
cd ~/Projects/my_skils/enabled
ln -s ../library/juejin-publisher .

# 或直接创建
mkdir -p ~/.claude/skills/juejin-publisher
# 复制 SKILL.md 内容
```

## 第二部分：完整操作流程

### 2.1 流程图

```
用户：发布到掘金
        ↓
读取 Markdown 文件
        ↓
打开掘金编辑器
        ↓
检测登录状态 ──→ 未登录 → 提示用户登录 → 等待确认
        ↓ 已登录
填充标题和正文
        ↓
点击发布按钮
        ↓
选择分类 + 添加标签
        ↓
确认发布
        ↓
返回文章链接
```

### 2.2 详细步骤

#### Step 1：读取文章

```javascript
// 读取 Markdown 文件
Read(文件路径)

// 提取内容
const title = 第一行 # 标题
const content = 去掉标题行的正文
```

#### Step 2：打开编辑器并检测登录

```javascript
// 导航到编辑器
mcp__plugin_playwright_playwright__browser_navigate({
  url: "https://juejin.cn/editor/drafts/new"
})

// 获取页面快照
mcp__plugin_playwright_playwright__browser_snapshot()

// 登录状态判断
if (pageUrl.includes('/login')) {
  // 需要登录
  提示用户："检测到登录已失效，请手动登录后告诉我'已登录'"
} else if (pageUrl.includes('/editor/drafts')) {
  // 已登录，继续
}
```

#### Step 3：填充内容

```javascript
// 填充标题
mcp__plugin_playwright_playwright__browser_type({
  element: "文章标题输入框",
  ref: "e6",  // 实际 ref 从快照获取
  text: "文章标题"
})

// 填充正文（CodeMirror 编辑器）
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

#### Step 4：发布设置

```javascript
// 点击发布按钮
mcp__plugin_playwright_playwright__browser_click({
  element: "发布按钮",
  ref: "发布按钮ref"
})

// 选择分类
mcp__plugin_playwright_playwright__browser_click({
  element: "开发工具",  // 或其他分类
  ref: "分类ref"
})

// 添加标签
mcp__plugin_playwright_playwright__browser_run_code({
  code: `async (page) => {
    await page.click('text=请搜索添加标签');
    await page.waitForTimeout(500);
    await page.keyboard.type('Claude');  // 标签关键词
    await page.waitForTimeout(1000);
  }`
})

// 选择搜索结果中的标签
mcp__plugin_playwright_playwright__browser_click({
  element: "Claude标签",
  ref: "标签ref"
})
```

#### Step 5：确认发布

```javascript
// 点击确定并发布
mcp__plugin_playwright_playwright__browser_click({
  element: "确定并发布按钮",
  ref: "确定发布ref"
})

// 发布成功后获取文章链接
// 页面跳转到 https://juejin.cn/post/{article_id}
```

## 第三部分：Claude Code Skill 实现

### 3.1 Skill 目录结构

```
juejin-publisher/
└── SKILL.md    # 技能定义文件
```

### 3.2 SKILL.md 完整内容

```yaml
---
name: juejin-publisher
description: 使用 MCP Playwright 自动发布 Markdown 文章到掘金。当用户说"发布到掘金"、"掘金发文"、"发帖到掘金"、"juejin publish"时触发此技能。支持自动填充标题/内容、选择分类、添加标签、完成发布全流程。自动检测登录状态，保持会话持久化。
---

# 掘金文章发布器

使用 MCP Playwright 自动化发布 Markdown 文章到掘金平台。

## 前置要求

- MCP Playwright 服务已安装并运行
- 用户已有掘金账号

## 会话持久化

MCP Playwright 默认保持浏览器会话，登录一次后无需重复登录。

## 发布流程

[完整流程见第二部分]

## 分类列表

- 后端、前端、Android、iOS
- 人工智能、开发工具
- 代码人生、阅读

## 标签推荐

- AI/Claude → Claude、AI、LLM
- 前端 → React、Vue、TypeScript
- 后端 → Node.js、Python、Go

## 使用示例

发布到掘金：docs/my-article.md
分类：开发工具
标签：Claude
```

### 3.3 启用技能

```bash
# 使用技能库
cd ~/Projects/my_skils/enabled
ln -s ../library/juejin-publisher .

# 验证
ls -la ~/.claude/skills/ | grep juejin
```

## 第四部分：会话持久化机制

### 4.1 原理说明

MCP Playwright 使用持久化的浏览器 Profile，登录信息存储在：
- Cookies
- LocalStorage
- SessionStorage

关闭浏览器后重新打开，会话状态保持不变。

### 4.2 验证方法

```javascript
// 1. 关闭浏览器
mcp__plugin_playwright_playwright__browser_close()

// 2. 重新打开编辑器
mcp__plugin_playwright_playwright__browser_navigate({
  url: "https://juejin.cn/editor/drafts/new"
})

// 3. 检查 URL
// 如果仍是 /editor/drafts/new → 登录有效
// 如果跳转 /login → 需要重新登录
```

### 4.3 登录失效处理

当检测到登录失效时，技能会自动提示：

```
检测到掘金登录已失效，请在浏览器中完成登录。
支持方式：手机验证码 / 微信扫码 / GitHub
登录成功后告诉我"已登录"。
```

## 第五部分：掘金平台特性

### 5.1 编辑器特点

- 使用 **CodeMirror** 富文本编辑器
- 支持 **Markdown** 实时预览
- 自动保存草稿

### 5.2 发布要求

| 字段 | 必填 | 说明 |
|------|------|------|
| 标题 | ✅ | 不超过 100 字符 |
| 正文 | ✅ | 支持 Markdown |
| 分类 | ✅ | 8 个分类选一 |
| 标签 | ✅ | 至少 1 个，最多 1 个 |
| 摘要 | ❌ | 自动生成，可修改 |
| 封面 | ❌ | 可选上传 |

### 5.3 分类说明

| 分类 | 适用内容 |
|------|----------|
| 后端 | 服务端开发、数据库、API |
| 前端 | Web/移动端 UI、框架 |
| Android | 安卓开发 |
| iOS | 苹果开发 |
| 人工智能 | AI、ML、LLM |
| 开发工具 | IDE、CLI、效率工具 |
| 代码人生 | 职业发展、经验分享 |
| 阅读 | 书评、学习笔记 |

## 常见问题

### Q1：CodeMirror 点击超时

**问题**：直接点击编辑器输入框超时

**解决**：使用 `browser_run_code` 通过 JavaScript 操作：

```javascript
await page.evaluate((text) => {
  document.querySelector('.CodeMirror').CodeMirror.setValue(text);
}, content);
```

### Q2："至少添加一个标签"错误

**问题**：发布时提示标签必填

**解决**：确保在发布前已添加标签，流程中增加标签添加步骤。

### Q3：登录状态频繁失效

**可能原因**：
- 浏览器 Profile 被清理
- Cookie 过期
- 账号在其他设备登录

**建议**：定期测试登录状态，失效时手动重新登录。

## 参考资源

- [MCP Playwright 文档](https://github.com/anthropics/claude-code)
- [Claude Code Skills 指南](https://docs.anthropic.com/claude-code/skills)
- [掘金创作者中心](https://juejin.cn/creator)
