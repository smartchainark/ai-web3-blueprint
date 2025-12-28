# Claude Code 配置 Notion MCP 及 PRD 转任务 SOP

## 概述

本文档介绍如何在 Claude Code 中配置 Notion MCP 服务器，并将本地 PRD 文档自动转化为 Notion 项目页面和任务数据库。

---

## 第一部分：配置 Notion MCP 服务器

### 1.1 获取 Notion API Token

1. 访问 [Notion Integrations](https://www.notion.so/profile/integrations)
2. 点击「新建集成」(New integration)
3. 填写集成名称（如 "Claude Code"）
4. 选择关联的工作区
5. 点击「提交」后复制生成的 Token（以 `ntn_` 或 `secret_` 开头）

### 1.2 在 Claude Code 中添加 MCP 服务器

**方式一：项目级配置（仅当前项目可用）**

```bash
claude mcp add notion -e NOTION_TOKEN=你的token -- npx -y @notionhq/notion-mcp-server
```

**方式二：用户级配置（所有项目可用）**

```bash
claude mcp add notion -s user -e NOTION_TOKEN=你的token -- npx -y @notionhq/notion-mcp-server
```

### 1.3 验证连接

```bash
claude mcp list
```

预期输出：
```
notion: npx -y @notionhq/notion-mcp-server - ✓ Connected
```

### 1.4 授权页面访问

**重要**：Notion API 只能访问已授权的页面。

1. 打开你想让 Claude 访问的 Notion 页面
2. 点击右上角 `···` 菜单
3. 选择「连接」(Connections)
4. 找到并选择你创建的集成

---

## 第二部分：PRD 转 Notion 任务

### 2.1 准备工作

确保：
- Notion MCP 已配置并连接
- 目标页面已授权给集成
- PRD 文档已准备好（Markdown 格式）

### 2.2 API 调用模式

由于 MCP 工具需要在新会话中生效，可以使用 curl 直接调用 Notion API：

```bash
# 设置环境变量
TOKEN="你的Notion Token"
```

### 2.3 搜索已授权页面

```bash
curl -s -X POST "https://api.notion.com/v1/search" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"page_size": 10}'
```

### 2.4 创建项目页面

```bash
curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"page_id": "父页面ID"},
    "properties": {
      "title": {"title": [{"text": {"content": "项目名称"}}]}
    },
    "children": [
      {
        "object": "block",
        "type": "heading_1",
        "heading_1": {"rich_text": [{"type": "text", "text": {"content": "项目概览"}}]}
      },
      {
        "object": "block",
        "type": "paragraph",
        "paragraph": {"rich_text": [{"type": "text", "text": {"content": "项目描述..."}}]}
      }
    ]
  }'
```

### 2.5 创建任务数据库

```bash
curl -s -X POST "https://api.notion.com/v1/databases" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"page_id": "项目页面ID"},
    "title": [{"type": "text", "text": {"content": "任务列表"}}],
    "properties": {
      "任务名称": {"title": {}},
      "状态": {
        "select": {
          "options": [
            {"name": "待开始", "color": "gray"},
            {"name": "进行中", "color": "blue"},
            {"name": "已完成", "color": "green"},
            {"name": "阻塞", "color": "red"}
          ]
        }
      },
      "优先级": {
        "select": {
          "options": [
            {"name": "P0-紧急", "color": "red"},
            {"name": "P1-重要", "color": "yellow"},
            {"name": "P2-一般", "color": "gray"}
          ]
        }
      },
      "阶段": {
        "select": {
          "options": [
            {"name": "Phase 1", "color": "purple"},
            {"name": "Phase 2", "color": "blue"},
            {"name": "Phase 3", "color": "green"}
          ]
        }
      },
      "模块": {
        "select": {
          "options": [
            {"name": "模块A", "color": "orange"},
            {"name": "模块B", "color": "yellow"}
          ]
        }
      }
    }
  }'
```

### 2.6 批量创建任务

```bash
DATABASE_ID="数据库ID"

create_task() {
  local name="$1"
  local task_status="$2"
  local priority="$3"
  local phase="$4"
  local module="$5"

  curl -s -X POST "https://api.notion.com/v1/pages" \
    -H "Authorization: Bearer $TOKEN" \
    -H "Notion-Version: 2022-06-28" \
    -H "Content-Type: application/json" \
    -d "{
      \"parent\": {\"database_id\": \"$DATABASE_ID\"},
      \"properties\": {
        \"任务名称\": {\"title\": [{\"text\": {\"content\": \"$name\"}}]},
        \"状态\": {\"select\": {\"name\": \"$task_status\"}},
        \"优先级\": {\"select\": {\"name\": \"$priority\"}},
        \"阶段\": {\"select\": {\"name\": \"$phase\"}},
        \"模块\": {\"select\": {\"name\": \"$module\"}}
      }
    }"
}

# 示例调用
create_task "任务名称" "待开始" "P1-重要" "Phase 1" "模块A"
```

### 2.7 追加内容到页面

```bash
curl -s -X PATCH "https://api.notion.com/v1/blocks/页面ID/children" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "children": [
      {"object": "block", "type": "heading_2", "heading_2": {"rich_text": [{"type": "text", "text": {"content": "新章节"}}]}},
      {"object": "block", "type": "to_do", "to_do": {"rich_text": [{"type": "text", "text": {"content": "待办事项"}}], "checked": false}}
    ]
  }'
```

---

## 第三部分：PRD 拆解最佳实践

### 3.1 任务拆解原则

1. **按功能模块拆分**：每个模块独立成组
2. **按开发阶段划分**：设计 → 开发 → 测试 → 发布
3. **优先级分级**：
   - P0：核心功能，阻塞上线
   - P1：重要功能，计划内
   - P2：优化项，有空再做

### 3.2 典型 PRD 结构映射

| PRD 章节 | Notion 对应 |
|---------|-------------|
| 项目背景 | 项目页面概览 |
| 产品目标 | 成功指标列表 |
| 功能需求 | 任务数据库条目 |
| 非功能需求 | 任务验收标准 |
| 里程碑 | 阶段划分 |

### 3.3 任务粒度建议

- 每个任务预估工作量：1-3 天
- 任务描述包含：做什么、验收标准、依赖项
- 避免过大任务，及时拆分

---

## 常见问题

### Q1: 提示 "Could not find page with ID"

**原因**：页面未授权给集成

**解决**：在 Notion 中打开页面 → `···` → 连接 → 选择你的集成

### Q2: MCP 工具不可用

**原因**：MCP 工具在添加后需要新会话才能生效

**解决**：退出当前 Claude Code 会话，重新启动

### Q3: API 调用返回 401

**原因**：Token 无效或过期

**解决**：重新获取 Token 并更新配置

---

## 参考资源

- [Notion API 文档](https://developers.notion.com/docs)
- [Notion MCP Server](https://github.com/makenotion/notion-mcp-server)
- [Claude Code MCP 指南](https://docs.anthropic.com/claude-code/mcp)
