# 知乎专栏自动发布：MCP Playwright 自动化 SOP

> 使用 MCP Playwright 实现知乎专栏文章一键发布，支持会话持久化和登录检测

**文档编号**: 015
**日期**: 2025-12-29
**标签**: `MCP` `Playwright` `自动化` `知乎` `内容发布`

---

## 概述

本文档详细介绍如何使用 MCP Playwright 自动化发布 Markdown 文章到知乎专栏平台，包括完整的操作流程、Claude Code Skill 实现和常见问题处理。

### 核心价值

- **一键发布**：说"发布到知乎"即可完成全流程
- **会话持久化**：登录一次，长期有效
- **智能检测**：自动识别登录状态，按需提醒
- **Markdown 支持**：知乎原生支持 Markdown 语法

### 与其他平台的区别

| 特性 | 知乎 | 掘金 | 微信公众号 |
|------|------|------|------------|
| 图片要求 | 可选 | 可选 | 必须至少 1 张 |
| 话题/标签 | 可选 | 必须选择 | 无 |
| 分类选择 | 无 | 必须选择 | 无 |
| 发布验证 | 无需验证 | 无需验证 | 需管理员扫码 |
| 编辑器类型 | 富文本 | CodeMirror | 富文本 |
| Markdown | 原生支持 | 原生支持 | 不支持 |

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
ln -s ../library/zhihu-publisher .

# 或直接创建
mkdir -p ~/.claude/skills/zhihu-publisher
# 复制 SKILL.md 内容
```

## 第二部分：完整操作流程

### 2.1 流程图

```
用户：发布到知乎
        ↓
读取 Markdown 文件
        ↓
打开知乎编辑器
        ↓
检测登录状态 ──→ 未登录 → 提示用户扫码 → 等待确认
        ↓ 已登录
填充标题
        ↓
填充正文内容
        ↓
添加话题标签（可选）
        ↓
点击发布按钮
        ↓
发布成功
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

#### Step 2：打开编辑器

```javascript
// 导航到编辑器
mcp__plugin_playwright_playwright__browser_navigate({
  url: "https://zhuanlan.zhihu.com/write"
})
```

**登录状态判断**：
- 如果 URL 跳转到登录页面 → 需要登录
- 如果进入编辑器 → 已登录，继续

#### Step 3：填充标题

```javascript
mcp__plugin_playwright_playwright__browser_type({
  element: "标题输入框",
  ref: "标题ref",
  text: "文章标题"
})
```

#### Step 4：填充正文

知乎编辑器支持 Markdown，使用 keyboard.type 填充：

```javascript
mcp__plugin_playwright_playwright__browser_click({
  element: "正文输入区域",
  ref: "正文ref"
})

mcp__plugin_playwright_playwright__browser_run_code({
  code: `async (page) => {
    const content = \`文章内容...\`;
    await page.keyboard.type(content, { delay: 1 });
    return 'Content typed';
  }`
})
```

#### Step 5：添加话题（可选）

```javascript
// 点击添加话题
mcp__plugin_playwright_playwright__browser_click({
  element: "添加话题",
  ref: "添加话题ref"
})

// 搜索话题
mcp__plugin_playwright_playwright__browser_type({
  element: "搜索话题",
  ref: "搜索ref",
  text: "Claude"
})

// 选择话题
mcp__plugin_playwright_playwright__browser_click({
  element: "话题选项",
  ref: "话题ref"
})
```

#### Step 6：发布

```javascript
// 点击发布按钮
mcp__plugin_playwright_playwright__browser_click({
  element: "发布",
  ref: "发布ref"
})
```

发布成功后，页面自动跳转到文章页面。

## 第三部分：Claude Code Skill 实现

### 3.1 Skill 目录结构

```
zhihu-publisher/
└── SKILL.md    # 技能定义文件
```

### 3.2 SKILL.md 完整内容

```yaml
---
name: zhihu-publisher
description: 使用 MCP Playwright 自动发布文章到知乎专栏。当用户说"发布到知乎"、"知乎发文"、"zhihu publish"时触发此技能。支持自动填充标题/内容、添加话题标签、完成发布全流程。自动检测登录状态，保持会话持久化。
---

# 知乎专栏文章发布器

使用 MCP Playwright 自动化发布 Markdown 文章到知乎专栏平台。

## 前置要求

- MCP Playwright 服务已安装并运行
- 用户已有知乎账号

## 发布要求

| 字段 | 必填 | 说明 |
|------|------|------|
| 标题 | Yes | 不超过 100 字符 |
| 正文 | Yes | 支持 Markdown 格式 |
| 话题 | No | 可添加 1-5 个话题标签 |
| 封面 | No | 可选，上传图片 |

## 使用示例

发布到知乎：docs/my-article.md
```

### 3.3 启用技能

```bash
# 使用技能库
cd ~/Projects/my_skils/enabled
ln -s ../library/zhihu-publisher .

# 验证
ls -la ~/.claude/skills/ | grep zhihu
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
  url: "https://zhuanlan.zhihu.com/write"
})

// 3. 检查页面
// 如果显示编辑器 → 登录有效
// 如果显示登录页面 → 需要重新登录
```

### 4.3 登录失效处理

当检测到登录失效时，技能会自动提示：

```
检测到知乎登录已失效，请在浏览器中完成登录。
支持方式：手机验证码、密码、微信/QQ 扫码
登录成功后告诉我"已登录"。
```

## 第五部分：知乎平台特性

### 5.1 编辑器特点

- 使用**富文本编辑器**
- 原生支持 Markdown 语法
- 自动保存草稿
- 支持代码块语法高亮
- 无需强制添加图片

### 5.2 发布要求

| 字段 | 必填 | 说明 |
|------|------|------|
| 标题 | Yes | 不超过 100 字符 |
| 正文 | Yes | 无最低字数限制 |
| 话题 | No | 最多 5 个 |
| 封面 | No | 可选 |
| 创作声明 | No | 可选（原创/转载等） |

### 5.3 知乎专栏功能

- **草稿备份**：自动保存，可恢复历史版本
- **创作助手**：内容检测、智能排版、智能标题
- **投稿至问题**：可将文章投稿到相关问题
- **送礼物设置**：开启/关闭读者打赏

## 常见问题

### Q1: 登录页面无法扫码

**问题**：二维码无法显示或扫码后无反应

**解决**：
- 刷新页面重新加载二维码
- 尝试使用密码登录
- 检查网络连接

### Q2: Markdown 表格不渲染

**问题**：表格显示为原始文本

**说明**：知乎编辑器的 Markdown 表格支持有限，发布后会正常渲染。如果预览时显示异常，发布后检查实际效果。

### Q3: 话题搜索无结果

**问题**：搜索话题时没有匹配结果

**解决**：
- 使用更通用的关键词
- 检查拼写
- 尝试相关话题

### Q4: 发布按钮不可点击

**问题**：发布按钮处于禁用状态

**原因**：标题或正文为空

**解决**：确保标题和正文都已填写内容。

## 参考资源

- [MCP Playwright 文档](https://github.com/anthropics/claude-code)
- [Claude Code Skills 指南](https://docs.anthropic.com/claude-code/skills)
- [知乎专栏](https://zhuanlan.zhihu.com/)
