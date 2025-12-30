# Notion + Telegram 内部文档发布通知 SOP

> 标准化内部文档发布流程：写文档 → 发布 Notion → Telegram 群通知

**文档编号**: 017
**日期**: 2025-12-31
**标签**: `Notion` `Telegram` `SOP` `自动化` `通知`

---

## 概述

本文档描述了将内部说明文档发布到 Notion 并通过 Telegram 机器人发送群组通知的标准操作流程。适用于项目上线说明、功能发布、技术文档等场景。

### 核心流程

```
本地文档 → Notion API 发布 → Telegram Bot 群通知
```

### 适用场景

- 项目/功能上线说明
- 技术架构文档
- 部署记录
- 团队内部沟通文档

---

## 第一部分：前置配置

### 1.1 Notion 配置

| 配置项 | 说明 |
|--------|------|
| API Token | Notion Integration 的 API Token |
| 目标页面 | 存放文档的父页面（如「输出方案」） |
| 页面 ID | 父页面的 UUID |
| 命名规范 | `{MMDD}-{文档名称}`，如 `1231-功能上线说明` |

**获取 API Token**：
1. 访问 https://www.notion.so/my-integrations
2. 创建新 Integration
3. 复制 Internal Integration Token

**获取页面 ID**：
1. 打开目标 Notion 页面
2. URL 中 `notion.so/` 后的 32 位字符串即为 ID
3. 或使用 Search API 搜索

### 1.2 Telegram 配置

| 配置项 | 说明 |
|--------|------|
| Bot Token | Telegram BotFather 创建的 Bot Token |
| Chat ID | 目标群组的 Chat ID（负数） |

**获取群组 Chat ID**：
1. 将 Bot 添加到群组并设为管理员
2. 调用 `getUpdates` API 获取群组消息
3. 从返回结果中提取 `chat.id`

```bash
curl -s "https://api.telegram.org/bot<BOT_TOKEN>/getUpdates"
```

---

## 第二部分：操作流程

### Step 1: 搜索目标 Notion 页面

```bash
curl -s -X POST 'https://api.notion.com/v1/search' \
  -H 'Authorization: Bearer <NOTION_TOKEN>' \
  -H 'Content-Type: application/json' \
  -H 'Notion-Version: 2022-06-28' \
  -d '{"query": "目标页面名称", "page_size": 5}'
```

**返回结果**：包含 `id` 字段的页面信息。

### Step 2: 创建 Notion 子页面

```bash
curl -s -X POST 'https://api.notion.com/v1/pages' \
  -H 'Authorization: Bearer <NOTION_TOKEN>' \
  -H 'Content-Type: application/json' \
  -H 'Notion-Version: 2022-06-28' \
  -d '{
  "parent": { "page_id": "<PARENT_PAGE_ID>" },
  "properties": {
    "title": {
      "title": [{ "text": { "content": "MMDD-文档标题" } }]
    }
  },
  "children": [
    {
      "object": "block",
      "type": "heading_1",
      "heading_1": {
        "rich_text": [{ "type": "text", "text": { "content": "文档标题" } }]
      }
    },
    {
      "object": "block",
      "type": "paragraph",
      "paragraph": {
        "rich_text": [{ "type": "text", "text": { "content": "文档内容..." } }]
      }
    }
  ]
}'
```

### Step 3: 发送 Telegram 群组通知

```bash
curl -s -X POST "https://api.telegram.org/bot<BOT_TOKEN>/sendMessage" \
  -H "Content-Type: application/json" \
  -d '{
    "chat_id": "<CHAT_ID>",
    "parse_mode": "Markdown",
    "text": "📢 *通知标题*\n\n内容摘要...\n\n📋 [查看完整文档](NOTION_PAGE_URL)"
  }'
```

---

## 第三部分：Notion Block 类型参考

| 类型 | 用途 | 示例 |
|------|------|------|
| `heading_1` | 一级标题 | 文档主标题 |
| `heading_2` | 二级标题 | 章节标题 |
| `heading_3` | 三级标题 | 小节标题 |
| `paragraph` | 段落 | 正文内容 |
| `bulleted_list_item` | 无序列表 | 功能点列表 |
| `numbered_list_item` | 有序列表 | 步骤说明 |
| `divider` | 分割线 | 章节分隔 |
| `code` | 代码块 | 技术内容 |
| `callout` | 提示框 | 警告、注意事项 |

---

## 第四部分：Telegram Markdown 格式

| 格式 | 语法 | 效果 |
|------|------|------|
| 粗体 | `*text*` | **text** |
| 斜体 | `_text_` | *text* |
| 代码 | `` `code` `` | `code` |
| 链接 | `[text](url)` | 可点击链接 |

**注意**：Telegram Markdown 语法与标准 Markdown 略有不同。

---

## 第五部分：文档模板

### 上线说明文档结构

```markdown
# {功能名称} 上线说明

发布日期：{YYYY-MM-DD} | 状态：已上线

---

## 📋 概述
{功能简介，1-2句话}

## 🔗 线上地址
- 前端地址：{URL}
- API 地址：{URL}（如有）
- 管理后台：{URL}（如有）

## 📜 合约地址（如适用）
- 合约1: {地址}
- 合约2: {地址}

## ✨ 功能特性
1. 特性1
2. 特性2
3. 特性3

## 👤 权限说明（如适用）
- 管理员: {地址/角色}

## ⚠️ 注意事项
1. 注意事项1
2. 注意事项2

---

如有问题请联系开发团队。
```

---

## 第六部分：故障排除

### Notion API 错误

| 错误码 | 原因 | 解决方案 |
|--------|------|---------|
| 401 | Token 无效 | 检查 API Token 是否正确 |
| 403 | 无权限 | 确保 Integration 已添加到目标页面 |
| 404 | 页面不存在 | 检查 page_id 是否正确 |
| 400 | 请求格式错误 | 检查 JSON 格式和 block 类型 |

### Telegram API 错误

| 错误码 | 原因 | 解决方案 |
|--------|------|---------|
| 401 | Bot Token 无效 | 检查 Token 是否正确 |
| 400 | Chat ID 错误 | 确认 Chat ID 格式正确（群组为负数） |
| 403 | Bot 无权限 | 确保 Bot 是群组管理员 |

---

## 第七部分：与 Claude Code 集成

### 自动化工作流

在 Claude Code 中，可以将此 SOP 封装为技能（Skill），实现：

1. **文档生成**：根据项目信息自动生成标准化文档
2. **Notion 发布**：调用 API 自动创建页面
3. **群组通知**：发布完成后自动发送 Telegram 通知

### 配置存储

将配置信息存储在项目的 SOP 文档中：

```yaml
notion:
  token: "ntn_xxx..."
  parent_page_id: "xxx-xxx-xxx"
  naming: "{MMDD}-{文档名称}"

telegram:
  bot_token: "123456:ABC..."
  chat_id: "-123456789"
  group_name: "文档通知群"
```

---

## 参考资源

- [Notion API 文档](https://developers.notion.com/)
- [Telegram Bot API 文档](https://core.telegram.org/bots/api)
- [Notion Block 类型参考](https://developers.notion.com/reference/block)

---

*最后更新：2025-12-31*
