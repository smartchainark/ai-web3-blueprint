# Cursor IDE 技能系统与 MCP 集成：完整配置 SOP

> 在 Cursor IDE 中安装 OpenSkills 技能系统，配置 Notion MCP 服务器，实现自动化工作流

**文档编号**: 003
**日期**: 2025-12-29
**标签**: `Cursor IDE` `OpenSkills` `MCP` `Notion` `工作流`

---

## 概述

本文档演示如何在 Cursor IDE 中配置 OpenSkills 技能系统和 MCP 服务器，实现从 PRD 到任务的自动化工作流：

| 步骤 | 工具/技能 | 说明 |
|------|-----------|------|
| 1. 安装技能系统 | OpenSkills CLI | 安装技能加载器和核心技能包 |
| 2. 配置 MCP | Notion MCP Server | 连接 Cursor 到 Notion API |
| 3. 生成实施计划 | `/notion-spec-to-implementation` | PRD → 实施计划 → Notion 任务 |
| 4. 写进度报告 | `/internal-comms` | 生成 3P 格式进度报告 |
| 5. 知识捕获 | `/notion-knowledge-capture` | 对话 → Notion 文档 |

```
OpenSkills 安装 → MCP 配置 → 技能调用 → Notion 自动化
```

**适用环境**：Cursor IDE（支持 MCP）
**预计完成时间**：30-45 分钟

---

## 第一部分：OpenSkills 技能系统安装

### 1.1 什么是 OpenSkills

OpenSkills 是一个通用的技能加载器，为 AI 编码助手提供专业领域知识和工作流程：

- 安装预制的专业技能包
- 在项目间共享技能
- 标准化团队协作方式

### 1.2 安装 OpenSkills CLI

```bash
# 全局安装
npm install -g openskills

# 验证安装
openskills --version
```

### 1.3 初始化项目

```bash
# 进入项目目录
cd /path/to/your/project

# 初始化 OpenSkills
openskills init
```

**预期结果**：
- 创建 `.claude/` 目录
- 创建 `AGENTS.md` 文件（技能注册表）

### 1.4 安装核心技能包

技能来源分为两类：

**1. Anthropic 官方技能**（从官方市场安装）：

```bash
# 从 Anthropic 官方仓库安装（交互式选择）
openskills install anthropics/skills

# 或安装特定技能
openskills install anthropics/skills/doc-coauthoring
openskills install anthropics/skills/internal-comms
```

**2. Notion 官方技能**（从 Notion 官方页面获取）：

访问 [Notion Skills for Claude](https://www.notion.so/notiondevs/Notion-Skills-for-Claude-28da4445d27180c7af1df7d8615723d0) 获取以下技能：

- notion-knowledge-capture - 知识捕获
- notion-research-documentation - 研究文档
- notion-spec-to-implementation - 规范转实施

按照 Notion 官方页面的安装说明操作。

### 1.5 同步并验证

```bash
# 同步技能到 AGENTS.md
openskills sync

# 列出所有已安装技能
openskills list
```

### 1.6 核心技能说明

| 技能名称 | 来源 | 用途 |
|---------|------|------|
| doc-coauthoring | Anthropic 官方 | 编写技术文档、提案、规范 |
| internal-comms | Anthropic 官方 | 写进度报告、3P 报告 |
| notion-knowledge-capture | Notion 官方 | 将对话转为 Notion 文档 |
| notion-research-documentation | Notion 官方 | 搜索 Notion 并生成报告 |
| notion-spec-to-implementation | Notion 官方 | PRD → 任务清单 |

**技能来源**：
- Anthropic 官方：[github.com/anthropics/skills](https://github.com/anthropics/skills)
- Notion 官方：[Notion Skills for Claude](https://www.notion.so/notiondevs/Notion-Skills-for-Claude-28da4445d27180c7af1df7d8615723d0)

---

## 第二部分：Notion MCP 配置

### 2.1 创建 Notion 集成

1. 访问 [Notion 集成页面](https://www.notion.so/my-integrations)
2. 点击 **"+ New integration"**
3. 填写信息：
   - **Name**: `Cursor MCP Integration`
   - **Associated workspace**: 选择工作区
   - **Type**: Internal Integration
4. 点击 **"Submit"**
5. 复制 **Internal Integration Token**（格式：`ntn_xxxxxx...`）

### 2.2 授权集成访问页面

**重要**：必须为集成授权才能访问 Notion 页面！

1. 打开目标 Notion 页面
2. 点击右上角 **"..."**
3. 选择 **"Connections"**
4. 搜索并选择 `Cursor MCP Integration`
5. 点击 **"Confirm"**

### 2.3 配置 Cursor MCP

找到 MCP 配置文件：

```
# macOS/Linux
~/.cursor/mcp.json

# Windows
C:\Users\<用户名>\.cursor\mcp.json
```

### 2.4 编辑 mcp.json

```json
{
  "mcpServers": {
    "notion": {
      "command": "npx",
      "args": [
        "-y",
        "@notionhq/notion-mcp-server"
      ],
      "env": {
        "NOTION_TOKEN": "ntn_YOUR_TOKEN_HERE"
      }
    }
  }
}
```

**配置说明**：
- `command`: 使用 `npx` 运行 npm 包
- `args`: `-y` 自动确认，`@notionhq/notion-mcp-server` 官方 MCP 包
- `env.NOTION_TOKEN`: 你的 Notion API Token

### 2.5 多 MCP 服务器配置（可选）

```json
{
  "mcpServers": {
    "notion": {
      "command": "npx",
      "args": ["-y", "@notionhq/notion-mcp-server"],
      "env": {
        "NOTION_TOKEN": "ntn_YOUR_TOKEN_HERE"
      }
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "alchemy": {
      "command": "npx",
      "args": ["-y", "@alchemy/mcp-server"],
      "env": {
        "ALCHEMY_API_KEY": "your-alchemy-api-key"
      }
    }
  }
}
```

### 2.6 重启并验证

1. **完全退出** Cursor（不是最小化）
2. 重新打开 Cursor
3. 等待 MCP 初始化（约 5-10 秒）

**验证连接**：在 Cursor 聊天中输入：
```
搜索我的 Notion 工作区中包含 "项目" 的页面
```

---

## 第三部分：技能使用实战

### 3.1 使用 notion-spec-to-implementation

**场景**：有一个 PRD 文档，需要生成实施计划和任务清单。

**调用方式**：
```
使用 notion-spec-to-implementation 技能，数据源【docs/PRD-项目名称.md】
```

**技能自动完成**：
1. 读取并解析 PRD 文档
2. 提取功能需求和非功能需求
3. 生成实施计划
4. 定义验收标准
5. 分解详细任务
6. 创建 Notion 页面和任务

### 3.2 使用 internal-comms 写进度报告

**调用方式**：
```
调用 internal-comms 技能，给它【docs/PRD-项目名称.md】写一份进度报告
```

**输出**：3P 格式进度报告（Progress/Plans/Problems）

### 3.3 使用 notion-knowledge-capture

**调用方式**：
```
使用 notion-knowledge-capture 技能，将我们刚才讨论的技术决策保存到 Notion
```

**技能自动完成**：
1. 提取对话中的关键信息
2. 结构化整理（背景、决策、理由）
3. 创建 Notion 页面并保存

---

## 第四部分：工作流总结

### 4.1 完整工作流

```
PRD 文档 (Markdown)
    ↓
OpenSkills 技能解析
    ↓
生成实施计划 + 任务清单
    ↓
通过 Notion MCP
    ↓
自动创建 Notion 页面和任务
```

### 4.2 技能与工具对照

| 步骤 | 技能/工具 | 输入 | 输出 |
|------|-----------|------|------|
| 1 | OpenSkills CLI | GitHub URL | 技能包 |
| 2 | Notion MCP | Token | API 连接 |
| 3 | notion-spec-to-implementation | PRD | 实施计划 + 任务 |
| 4 | internal-comms | 项目信息 | 3P 报告 |
| 5 | notion-knowledge-capture | 对话 | Notion 文档 |

---

## 常见问题

### Q1: OpenSkills 安装失败

**原因**：npm 包名错误或网络问题

**解决**：
```bash
# 检查 npm 配置
npm config get registry

# 重新安装
npm install -g openskills
```

### Q2: Notion MCP 返回 "Unauthorized"

**可能原因**：
- Notion Token 错误
- 未授权集成访问页面
- 集成权限不足

**解决**：
1. 确认 Token 格式正确（以 `ntn_` 开头）
2. 在 Notion 页面中添加集成连接
3. 检查集成的 Capabilities 权限

### Q3: MCP 服务器包名错误

**错误包名**：
```json
"@modelcontextprotocol/server-notion"  ❌
```

**正确包名**：
```json
"@notionhq/notion-mcp-server"  ✅
```

### Q4: 技能无法识别

**原因**：技能未同步到 AGENTS.md

**解决**：
```bash
openskills sync
cat AGENTS.md
```

### Q5: Cursor 未加载 MCP 配置

**解决**：
1. 完全退出 Cursor
2. 确认 `mcp.json` 路径正确
3. 确认 JSON 格式无语法错误
4. 重新启动 Cursor

---

## 最佳实践

### 安全建议

**不要提交敏感信息**：
```gitignore
# .gitignore
.cursor/mcp.json
.env
.env.local
```

**使用环境变量**（推荐）：
```json
{
  "env": {
    "NOTION_TOKEN": "${NOTION_TOKEN}"
  }
}
```

**最小权限原则**：
- 仅授权需要访问的页面
- 仅启用必要的 Capabilities

### 团队协作

将技能配置加入版本控制：
```bash
git add .claude/
git add AGENTS.md
git commit -m "Add OpenSkills configuration"
```

---

## 参考资源

- [OpenSkills GitHub](https://github.com/numman-ali/openskills)
- [Anthropic 官方技能仓库](https://github.com/anthropics/skills)
- [Notion Skills for Claude](https://www.notion.so/notiondevs/Notion-Skills-for-Claude-28da4445d27180c7af1df7d8615723d0)
- [Cursor MCP 文档](https://docs.cursor.com/zh-Hant/tools/mcp)
- [Notion API 文档](https://developers.notion.com/)
- [Model Context Protocol 规范](https://modelcontextprotocol.io/)
- [MCP Servers 列表](https://github.com/modelcontextprotocol/servers)
