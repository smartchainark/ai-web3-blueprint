# 需求到任务到通知：完整工作流 SOP

> 从 PRD 撰写到 Notion 任务管理再到 Telegram 通知的完整闭环

**文档编号**: 001
**日期**: 2025-12-28
**标签**: `Claude Code` `Notion` `Telegram` `工作流`

---

## 概述

本文档演示一个完整的项目管理工作流：

```
PRD 撰写 → Notion 任务创建 → Telegram 通知
```

通过 Claude Code 的技能组合，实现从需求分析到团队通知的自动化闭环。

---

## 第一部分：配置环境

### 1.1 配置 Notion MCP 服务器

#### 获取 Notion API Token

1. 访问 [Notion Integrations](https://www.notion.so/profile/integrations)
2. 点击「新建集成」(New integration)
3. 填写集成名称（如 "Claude Code"）
4. 选择关联的工作区
5. 点击「提交」后复制生成的 Token（以 `ntn_` 或 `secret_` 开头）

#### 添加 MCP 服务器

```bash
# 项目级配置（仅当前项目可用）
claude mcp add notion -e NOTION_TOKEN=你的token -- npx -y @notionhq/notion-mcp-server

# 或：用户级配置（所有项目可用）
claude mcp add notion -s user -e NOTION_TOKEN=你的token -- npx -y @notionhq/notion-mcp-server
```

#### 验证连接

```bash
claude mcp list
```

预期输出：
```
notion: npx -y @notionhq/notion-mcp-server - ✓ Connected
```

#### 授权页面访问

**重要**：Notion API 只能访问已授权的页面。

1. 打开你想让 Claude 访问的 Notion 页面
2. 点击右上角 `···` 菜单
3. 选择「连接」(Connections)
4. 找到并选择你创建的集成

### 1.2 配置 Telegram Bot

#### 创建 Bot

1. 在 Telegram 中搜索 `@BotFather`
2. 发送 `/newbot` 并按提示操作
3. 保存获取的 Bot Token

#### 获取 Chat ID

```bash
BOT_TOKEN="你的Bot Token"

# 如果有活跃的 webhook，先删除
curl -s "https://api.telegram.org/bot$BOT_TOKEN/deleteWebhook"

# 向你的 Bot 发送一条消息，然后获取 chat_id
curl -s "https://api.telegram.org/bot$BOT_TOKEN/getUpdates" | jq '.result[0].message.chat.id'
```

---

## 第二部分：撰写 PRD 文档

### 2.1 使用 doc-coauthoring 技能

在 Claude Code 中输入：

```
/doc-coauthoring 写一份 [项目名称] 的 PRD 文档
```

### 2.2 PRD 结构建议

一个简洁的 MVP PRD 应包含：

```markdown
# 项目名称 PRD

## 1. 产品背景
- 痛点描述
- 为什么需要这个产品

## 2. 产品目标
- 核心价值
- 解决什么问题

## 3. 目标用户
- 用户画像

## 4. 功能需求
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 功能A | 说明 | P0 |
| 功能B | 说明 | P1 |

## 5. 非功能需求
- 性能要求
- 安全要求

## 6. MVP 范围
- 包含什么
- 不包含什么

## 7. 成功指标
- 如何衡量成功
```

### 2.3 示例：NotebookLM 集成助手 PRD

```markdown
# NotebookLM 集成助手 PRD

> 通过 Claude Code 查询 Google NotebookLM 笔记本

**版本**: MVP v1.0

## 1. 产品背景

NotebookLM 是 Google 推出的 AI 笔记本工具，能基于用户上传的文档提供精准回答。
但目前只能通过网页访问，无法与开发工具集成。

## 2. 产品目标

让 Claude Code 用户能直接查询 NotebookLM 笔记本，减少上下文切换。

## 3. 功能需求

### 认证管理
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 初始认证 | 浏览器弹窗登录 Google 账号 | P0 |
| 认证状态检查 | 查看当前认证状态 | P0 |
| 重新认证 | Token 过期后重新登录 | P1 |

### 笔记本库管理
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 添加笔记本 | 通过 URL 添加到本地库 | P0 |
| 列出笔记本 | 查看已添加的笔记本 | P0 |
| 设置默认笔记本 | 激活常用笔记本 | P1 |

### 查询功能
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 提问查询 | 向笔记本提问，获取 AI 回答 | P0 |
| 指定笔记本查询 | 查询特定笔记本 | P0 |

## 4. 技术方案

- 浏览器自动化: Patchright
- 运行环境: Python venv
- 数据存储: JSON 本地存储
```

---

## 第三部分：PRD 转 Notion 任务

### 3.1 使用 notion-spec-to-implementation 技能

这是最简单的方式。在 Claude Code 中直接调用技能：

```
/notion-spec-to-implementation
```

然后告诉 Claude：
- PRD 文件路径（如 `./drafts/notebooklm-prd.md`）
- 或直接在对话中提供 PRD 内容

### 3.2 技能自动完成的工作

该技能会自动：

1. **解析 PRD 文档** - 提取功能需求、优先级、模块划分
2. **创建项目页面** - 在 Notion 中创建项目主页
3. **创建任务数据库** - 包含任务名称、状态、优先级、模块等字段
4. **批量创建任务** - 根据 PRD 功能需求自动拆解为具体任务

### 3.3 示例对话

```
用户: /notion-spec-to-implementation

Claude: 请提供 PRD 文件路径或内容。

用户: PRD 在 ./drafts/notebooklm-prd.md

Claude: 我已读取 PRD，将创建以下任务：
- 认证模块：3 个任务（P0: 2, P1: 1）
- 笔记本库模块：3 个任务（P0: 2, P1: 1）
- 查询模块：2 个任务（P0: 2）

正在创建 Notion 项目页面...
✅ 项目页面已创建
✅ 任务数据库已创建
✅ 8 个任务已添加

项目链接: https://notion.so/xxx
```

### 3.4 手动 API 调用（可选）

如果需要更精细的控制，也可以直接调用 Notion API：

```bash
# 设置环境变量
TOKEN="你的Notion Token"
PARENT_PAGE="父页面ID"

# 创建项目页面
curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"page_id": "'$PARENT_PAGE'"},
    "properties": {
      "title": {"title": [{"text": {"content": "项目名称"}}]}
    }
  }'
```

详细 API 用法请参考 [Notion API 文档](https://developers.notion.com/docs)

---

## 第四部分：Telegram 通知

### 4.1 发送任务摘要

```bash
BOT_TOKEN="你的Bot Token"
CHAT_ID="你的Chat ID"

# 构建消息
MESSAGE="📋 *项目任务已创建*

📁 项目: NotebookLM 集成助手
🔗 [查看 Notion 页面](https://notion.so/$PROJECT_ID)

✅ 已创建任务:
• 认证模块: 3 个任务
• 笔记本库模块: 3 个任务
• 查询模块: 2 个任务

📊 总计: 8 个任务"

# 发送消息
curl -s -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendMessage" \
  -H "Content-Type: application/json" \
  -d "{
    \"chat_id\": \"$CHAT_ID\",
    \"text\": \"$MESSAGE\",
    \"parse_mode\": \"Markdown\"
  }"
```

### 4.2 消息格式说明

Telegram 支持 Markdown 格式：

| 格式 | 语法 | 效果 |
|------|------|------|
| 粗体 | `*文字*` | **文字** |
| 斜体 | `_文字_` | *文字* |
| 代码 | `` `代码` `` | `代码` |
| 链接 | `[文字](URL)` | 超链接 |

---

## 第五部分：完整工作流示例

### 5.1 在 Claude Code 中的完整流程

```
# 步骤 1: 撰写 PRD
用户: /doc-coauthoring 写一份 NotebookLM 集成助手的 PRD

Claude: [引导用户完成 PRD 撰写，保存到 ./drafts/notebooklm-prd.md]

# 步骤 2: 转换为 Notion 任务
用户: /notion-spec-to-implementation

Claude: 请提供 PRD 路径。

用户: ./drafts/notebooklm-prd.md

Claude: ✅ 已创建项目页面和 12 个任务
项目链接: https://notion.so/xxx

# 步骤 3: 发送 Telegram 通知
用户: 把任务摘要发送到 Telegram

Claude: [发送通知到 Telegram]
✅ 消息已发送
```

### 5.2 Telegram 通知脚本（可选）

如果需要独立的通知脚本：

```bash
#!/bin/bash

BOT_TOKEN="你的Bot Token"
CHAT_ID="你的Chat ID"
PROJECT_URL="https://notion.so/xxx"
TASK_COUNT=12

MESSAGE="📋 *项目任务已创建*

📁 项目: NotebookLM 集成助手
🔗 [查看 Notion](${PROJECT_URL})
📊 任务数: ${TASK_COUNT}

✅ 工作流完成!"

curl -s -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendMessage" \
  -H "Content-Type: application/json" \
  -d "{
    \"chat_id\": \"$CHAT_ID\",
    \"text\": \"$MESSAGE\",
    \"parse_mode\": \"Markdown\"
  }"
```

### 5.3 工作流总结

| 步骤 | 技能/工具 | 输出 |
|------|-----------|------|
| 1. PRD 撰写 | `/doc-coauthoring` | Markdown PRD 文件 |
| 2. 任务创建 | `/notion-spec-to-implementation` | Notion 项目页面 + 任务数据库 |
| 3. 团队通知 | Telegram Bot API | 消息推送 |

---

## 常见问题

### Q1: Notion 提示 "Could not find page with ID"

**原因**：页面未授权给集成

**解决**：在 Notion 中打开页面 → `···` → 连接 → 选择你的集成

### Q2: Telegram getUpdates 返回空

**原因**：有活跃的 webhook 或没有新消息

**解决**：
1. 先调用 `deleteWebhook` 删除 webhook
2. 向 Bot 发送一条消息
3. 再调用 `getUpdates`

### Q3: MCP 工具不可用

**原因**：MCP 工具在添加后需要新会话才能生效

**解决**：退出当前 Claude Code 会话，重新启动

---

## 参考资源

- [Notion API 文档](https://developers.notion.com/docs)
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Notion MCP Server](https://github.com/makenotion/notion-mcp-server)
- [Claude Code 文档](https://docs.anthropic.com/claude-code)
