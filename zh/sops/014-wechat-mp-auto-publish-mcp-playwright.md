# 微信公众号自动发布：MCP Playwright 自动化 SOP

> 使用 MCP Playwright 实现微信公众号文章一键发布，支持会话持久化和登录检测

**文档编号**: 014
**日期**: 2025-12-29
**标签**: `MCP` `Playwright` `自动化` `微信公众号` `内容发布`

---

## 概述

本文档详细介绍如何使用 MCP Playwright 自动化发布 Markdown 文章到微信公众号平台，包括完整的操作流程、Claude Code Skill 实现和常见问题处理。

### 核心价值

- **一键发布**：说"发布到微信"即可完成全流程
- **会话持久化**：登录一次，长期有效
- **智能检测**：自动识别登录状态，按需提醒
- **格式保留**：Markdown 内容完美呈现

### 与掘金发布的区别

| 特性 | 微信公众号 | 掘金 |
|------|------------|------|
| 图片要求 | ✅ 必须至少 1 张 | ❌ 可选 |
| 封面图 | 可选（默认用正文首图） | ✅ 必须 |
| 分类/标签 | 无分类系统 | ✅ 必须选择 |
| 发布验证 | 需管理员扫码 | 直接发布 |
| 编辑器类型 | 富文本（非 CodeMirror） | CodeMirror |

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
ln -s ../library/wechat-mp-publisher .

# 或直接创建
mkdir -p ~/.claude/skills/wechat-mp-publisher
# 复制 SKILL.md 内容
```

### 1.3 公众号图片库准备

**重要**：微信公众号要求文章必须包含至少一张图片才能发布。

建议提前在公众号后台上传一些通用图片到图片库，如：
- 技术文章配图
- Logo 或品牌图
- 通用封面图

## 第二部分：完整操作流程

### 2.1 流程图

```
用户：发布到微信
        ↓
读取 Markdown 文件
        ↓
打开公众号编辑器
        ↓
检测登录状态 ──→ 未登录 → 提示用户扫码 → 等待确认
        ↓ 已登录
处理弹窗（小店返佣、一键排版等）
        ↓
填充标题
        ↓
填充正文内容
        ↓
从图片库添加图片（必需）
        ↓
设置封面（可选）
        ↓
点击发布按钮
        ↓
确认群发通知
        ↓
管理员扫码验证
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
const content = 去掉标题行的正文（建议精简到 1000 字以内）
```

#### Step 2：打开编辑器

```javascript
// 导航到编辑器
mcp__plugin_playwright_playwright__browser_navigate({
  url: "https://mp.weixin.qq.com/cgi-bin/appmsg?t=media/appmsg_edit&action=edit&type=77&lang=zh_CN"
})
```

**登录状态判断**：
- 如果页面显示扫码登录界面 → 需要登录
- 如果进入编辑器 → 已登录，继续

#### Step 3：等待编辑器加载

微信编辑器加载较慢，需要等待：

```javascript
mcp__plugin_playwright_playwright__browser_wait_for({
  text: "请在这里输入标题",
  time: 5
})
```

#### Step 4：处理弹窗

微信编辑器可能弹出多个提示：

```javascript
// 小店返佣商品弹窗
mcp__plugin_playwright_playwright__browser_click({
  element: "稍后再说",
  ref: "稍后再说ref"
})

// 一键排版 tooltip
mcp__plugin_playwright_playwright__browser_click({
  element: "我知道了",
  ref: "我知道了ref"
})
```

#### Step 5：填充标题

```javascript
mcp__plugin_playwright_playwright__browser_type({
  element: "标题输入框",
  ref: "标题ref",
  text: "文章标题"
})
```

#### Step 6：填充正文

微信编辑器不是 CodeMirror，使用 keyboard.type 填充：

```javascript
mcp__plugin_playwright_playwright__browser_run_code({
  code: `async (page) => {
    // 点击编辑区域获取焦点
    await page.click('[data-placeholder="请在这里输入正文"]');
    await page.waitForTimeout(500);

    const content = \`
创建时间：2025-12-27

一个通过自然语言管理任务的 Claude Code Skill...

## 概述

Task Manager 是一个 Claude Code Skill...
    \`;

    // 逐字符输入，避免超时
    await page.keyboard.type(content, { delay: 1 });
    return 'Content typed';
  }`
})
```

**注意**：
- 内容不宜过长，建议精简到 1000 字以内
- `delay: 1` 参数可以避免输入过快导致的问题

#### Step 7：添加图片（必需）

微信公众号**必须**包含至少一张图片才能发布：

```javascript
// 点击图片菜单
mcp__plugin_playwright_playwright__browser_click({
  element: "图片",
  ref: "图片菜单ref"
})

// 等待下拉菜单出现
mcp__plugin_playwright_playwright__browser_snapshot()

// 选择"从图片库选择"
mcp__plugin_playwright_playwright__browser_click({
  element: "从图片库选择",
  ref: "从图片库选择ref"
})

// 等待图片库加载
mcp__plugin_playwright_playwright__browser_wait_for({ time: 2 })

// 选择第一张图片
mcp__plugin_playwright_playwright__browser_click({
  element: "图片缩略图",
  ref: "图片ref"
})

// 点击确定
mcp__plugin_playwright_playwright__browser_click({
  element: "确定",
  ref: "确定ref"
})
```

#### Step 8：设置封面（可选）

从正文图片选择封面：

```javascript
// 点击封面区域
mcp__plugin_playwright_playwright__browser_click({
  element: "拖拽或选择封面",
  ref: "封面区域ref"
})

// 选择"从正文选择"
mcp__plugin_playwright_playwright__browser_click({
  element: "从正文选择",
  ref: "从正文选择ref"
})

// 选择图片
mcp__plugin_playwright_playwright__browser_click({
  element: "正文中的图片",
  ref: "图片ref"
})

// 下一步
mcp__plugin_playwright_playwright__browser_click({
  element: "下一步",
  ref: "下一步ref"
})

// 确认（选择裁剪比例）
// 2.35:1 用于消息列表
// 1:1 用于分享卡片
mcp__plugin_playwright_playwright__browser_click({
  element: "确认",
  ref: "确认ref"
})
```

#### Step 9：发布

```javascript
// 点击发表按钮
mcp__plugin_playwright_playwright__browser_click({
  element: "发表",
  ref: "发表按钮ref"
})

// 发布选项对话框
// - 群发通知（默认开启）
// - 分组通知
// - 定时发表

// 点击对话框中的发表
mcp__plugin_playwright_playwright__browser_click({
  element: "发表",
  ref: "对话框发表ref"
})

// 确认继续发表
mcp__plugin_playwright_playwright__browser_click({
  element: "继续发表",
  ref: "继续发表ref"
})
```

#### Step 10：微信验证

发布最后一步需要管理员扫码验证：

```
提示用户：
"请用微信扫描页面上的二维码完成验证。
管理员微信号与运营者微信号可直接扫码验证。
非管理员微信号扫码后需管理员验证通过。"
```

验证完成后，文章自动发布。

## 第三部分：Claude Code Skill 实现

### 3.1 Skill 目录结构

```
wechat-mp-publisher/
└── SKILL.md    # 技能定义文件
```

### 3.2 SKILL.md 完整内容

```yaml
---
name: wechat-mp-publisher
description: 使用 MCP Playwright 自动发布文章到微信公众号。当用户说"发布到微信"、"微信公号发文"、"公众号发布"、"wechat publish"时触发此技能。支持自动填充标题/内容、添加图片、设置封面、完成发布全流程。自动检测登录状态，保持会话持久化。需要管理员扫码验证完成最终发布。
---

# 微信公众号文章发布器

使用 MCP Playwright 自动化发布 Markdown 文章到微信公众号平台。

## 前置要求

- MCP Playwright 服务已安装并运行
- 用户已有微信公众号账号
- 公众号图片库中已有可用图片

## 会话持久化

MCP Playwright 保持浏览器会话，登录一次后无需重复登录。

## 发布流程

[完整流程见第二部分]

## 发布要求

| 字段 | 必填 | 说明 |
|------|------|------|
| 标题 | ✅ | 不超过 64 字符 |
| 正文 | ✅ | 必须包含至少 1 张图片 |
| 封面 | ❌ | 可选，默认使用正文首图 |
| 摘要 | ❌ | 自动截取正文开头，限 120 字 |

## 使用示例

发布到微信：docs/my-article.md
```

### 3.3 启用技能

```bash
# 使用技能库
cd ~/Projects/my_skils/enabled
ln -s ../library/wechat-mp-publisher .

# 验证
ls -la ~/.claude/skills/ | grep wechat
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
  url: "https://mp.weixin.qq.com/cgi-bin/appmsg?t=media/appmsg_edit&action=edit&type=77&lang=zh_CN"
})

// 3. 检查页面
// 如果显示编辑器 → 登录有效
// 如果显示扫码页面 → 需要重新登录
```

### 4.3 登录失效处理

当检测到登录失效时，技能会自动提示：

```
检测到微信公众号登录已失效，请在浏览器中完成登录。
支持方式：微信扫码
登录成功后告诉我"已登录"。
```

## 第五部分：微信公众号平台特性

### 5.1 编辑器特点

- 使用**富文本编辑器**（非 CodeMirror）
- 支持直接粘贴图片
- 自动保存草稿
- 支持多图文消息

### 5.2 发布要求

| 字段 | 必填 | 说明 |
|------|------|------|
| 标题 | ✅ | 不超过 64 字符 |
| 正文 | ✅ | 必须包含至少 1 张图片 |
| 封面 | ❌ | 可选，默认使用正文首图 |
| 摘要 | ❌ | 自动截取正文开头，限 120 字 |
| 原创声明 | ❌ | 默认不声明 |
| 留言 | ❌ | 默认开启 |

### 5.3 封面尺寸

| 比例 | 用途 |
|------|------|
| 2.35:1 | 消息列表（订阅号消息） |
| 1:1 | 分享卡片、公众号主页 |

### 5.4 群发限制

- 订阅号：每天 1 次群发
- 服务号：每月 4 次群发

## 常见问题

### Q1: "必须插入一张图片"

**问题**：发布时提示必须插入图片

**解决**：确保在发布前已从图片库添加至少一张图片到正文。微信公众号文章必须包含图片才能发布。

### Q2: 编辑器加载慢

**问题**：页面元素找不到，操作超时

**解决**：
- 增加 `wait_for` 等待时间
- 多次获取 snapshot 确认加载完成
- 使用 `browser_wait_for({ text: "请在这里输入标题" })`

### Q3: 弹窗拦截点击

**问题**：点击被弹窗拦截，操作失败

**解决**：先关闭所有弹窗：
- "小店返佣商品" → 点击"稍后再说"
- "一键排版" tooltip → 点击"我知道了"
- 其他提示 → 点击关闭按钮

### Q4: 内容填充不完整

**问题**：keyboard.type 超时或内容截断

**解决**：
- 精简内容长度（建议 1000 字以内）
- 增加 `delay` 参数（如 `delay: 1`）
- 分段输入长内容

### Q5: 需要管理员验证

**问题**：发布时弹出微信验证二维码

**说明**：这是微信的安全机制，不是错误。需要：
- 管理员或运营者扫码验证
- 非管理员扫码后需管理员确认

## 参考资源

- [MCP Playwright 文档](https://github.com/anthropics/claude-code)
- [Claude Code Skills 指南](https://docs.anthropic.com/claude-code/skills)
- [微信公众平台](https://mp.weixin.qq.com/)
