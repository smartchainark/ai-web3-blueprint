# GTD + PARA 双系统 Notion MCP 自动化

> 结合 GTD 方法论与 PARA 组织体系，通过 Claude Code 技能实现 AI 驱动的任务管理

**文档编号**: 012
**日期**: 2025-12-29
**标签**: `GTD` `PARA` `Notion` `MCP` `任务管理` `技能`

---

## 概述

本文档介绍一套**双系统架构**的个人知识与任务管理方案：

| 系统 | 创始人 | 用途 |
|------|--------|------|
| **GTD** | David Allen | 任务捕获、澄清与执行 |
| **PARA** | Tiago Forte | 信息跨项目与领域的组织 |

通过将这两套方法论与 **Claude Code 技能** 和 **Notion MCP** 结合，我们实现了：

- 自然语言任务捕获（"添加想法：做一个自动化脚本"）
- 带语义前缀的自动分类
- 零摩擦的收件箱处理
- 通过 Notion 实现跨设备同步

```
用户意图 → 技能识别 → Notion MCP → 有序系统
```

---

## 第一部分：双系统架构

### 1.1 GTD 核心工作流

GTD 的核心是**捕获一切**并**明确下一步行动**：

```
捕获 → 澄清 → 组织 → 回顾 → 执行
```

我们的实现映射如下：

| GTD 阶段 | 实现方式 |
|----------|----------|
| 捕获 | 📥 Inbox - 所有东西先进这里 |
| 澄清 | 处理：是否可执行？ |
| 组织 | 路由到 任务/Areas/Resources/Archive |
| 回顾 | 每周回顾所有分类 |
| 执行 | 从 🔥 进行中 执行 |

### 1.2 PARA 组织体系

PARA 提供**四个分类**来组织所有信息：

| 分类 | 定义 | 我们的实现 |
|------|------|-----------|
| **P**rojects | 有截止日期的成果 | 🎯 任务（活跃的） |
| **A**reas | 持续的责任领域 | 🏢 Areas |
| **R**esources | 参考资料 | 📖 Resources |
| **A**rchives | 已完成/不活跃 | 🗄️ Archive |

### 1.3 组合结构

```
GTD + PARA 系统
├── 📥 Inbox              （GTD：捕获一切，每天清空）
├── 🎯 任务               （GTD：已组织的行动）
│   ├── 🔥 进行中         （正在执行）
│   ├── 💡 想法           （未来某天/也许，需评估）
│   └── 💤 挂起           （等待中，未来）
├── 🏢 Areas              （PARA：持续责任领域）
├── 📖 Resources          （PARA：参考资料）
├── 📋 Playbook           （SOP 和方法论）
└── 🗄️ Archive            （PARA：已完成/不活跃）
```

**关键创新**：任务状态使用 emoji 前缀实现即时视觉识别：
- 🔥 = 活跃，正在处理
- 💡 = 想法，需要澄清
- 💤 = 挂起，暂不处理

---

## 第二部分：Notion 配置

### 2.1 前置条件

1. **Notion 账户**及工作空间
2. **Notion MCP** 已在 Claude Code 中配置
3. **集成访问权限**已授予目标页面

### 2.2 创建页面结构

在 Notion 中创建匹配此层级的页面：

```
GTD 系统（根页面）
├── 📥 Inbox
├── 🎯 任务
├── 🏢 Areas
├── 📖 Resources
├── 📋 Playbook
└── 🗄️ Archive
```

### 2.3 收集页面 ID

每个 Notion 页面都有唯一 ID。从页面 URL 提取：

```
https://notion.so/Your-Page-Title-{PAGE_ID}
                                   ^^^^^^^^
                                   这部分
```

**格式**：`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`（UUID）

保存这些 ID 用于技能配置：

```bash
# 示例（使用你的实际 ID）
GTD_ROOT={你的GTD根页面ID}
INBOX={你的Inbox页面ID}
TASKS={你的任务页面ID}
AREAS={你的Areas页面ID}
RESOURCES={你的Resources页面ID}
PLAYBOOK={你的Playbook页面ID}
ARCHIVE={你的Archive页面ID}
```

### 2.4 授予集成访问权限

对每个页面：
1. 点击 `···` 菜单 → 连接
2. 添加你的 Claude Code 集成
3. 确认访问

---

## 第三部分：技能实现

### 3.1 技能结构

```
gtd-notion/
└── SKILL.md
```

### 3.2 SKILL.md 模板

```yaml
---
name: gtd-notion
description: GTD 任务管理系统整理助手。使用 Notion MCP 工具自动分类内容到
  Inbox/任务/Areas/Resources/Archive。支持添加想法(💡)、进行中(🔥)、挂起(💤)
  任务。当用户说"整理GTD"、"添加任务"、"添加想法"、"挂起"、"GTD归档"、
  "查看GTD状态"时触发。
---

# GTD Notion 整理助手

基于 GTD + PARA 方法论。

## 页面 ID 配置

```
GTD_ROOT={gtd-root-id}
INBOX={inbox-id}
TASKS={tasks-id}
AREAS={areas-id}
RESOURCES={resources-id}
PLAYBOOK={playbook-id}
ARCHIVE={archive-id}
```

## 操作指南

### 添加任务

| 用户说 | 前缀 | 示例 |
|--------|------|------|
| "添加想法：xxx" | 💡 | 💡 做一个自动化脚本 |
| "添加任务：xxx" | 🔥 | 🔥 完成用户认证模块 |
| "挂起：xxx" | 💤 | 💤 学习 Rust 语言 |

**API 调用**：
```
mcp__notion__API-post-page
- parent: {"page_id": "{TASKS}"}
- properties: {"title": [{"text": {"content": "💡 标题"}}]}
```

### 整理 Inbox

1. 获取内容：`mcp__notion__API-get-block-children` (block_id: INBOX)
2. 列出待整理项，询问分类
3. 移动页面：`mcp__notion__API-move-page`
4. 添加前缀：`mcp__notion__API-patch-page`

### 查看状态

1. 获取各分类的子页面
2. 统计各分类数量
3. 显示汇总报告

### 归档

1. 搜索匹配页面：`mcp__notion__API-post-search`
2. 移动到 Archive：`mcp__notion__API-move-page`

## 分类规则

| 特征 | 分类 | 前缀 |
|------|------|------|
| 明确可执行、有截止日期 | 进行中 | 🔥 |
| 灵感、点子、需要评估 | 想法 | 💡 |
| 未来某天、暂时搁置 | 挂起 | 💤 |
| 已完成、不再需要 | 归档 | → Archive |
```

### 3.3 安装技能

**个人技能**（全局可用）：
```bash
mkdir -p ~/.claude/skills/gtd-notion
# 将 SKILL.md 复制到此目录
```

**项目技能**（特定项目）：
```bash
mkdir -p .claude/skills/gtd-notion
# 将 SKILL.md 复制到此目录
```

---

## 第四部分：使用示例

### 4.1 快速捕获

```
用户：添加想法：AI 驱动的代码审查工具

Claude：已在「🎯 任务」页面添加：
- 💡 AI 驱动的代码审查工具
```

### 4.2 整理 Inbox

```
用户：整理 GTD

Claude：正在获取 Inbox 内容...

📥 Inbox 待整理项：
1. "新的营销方案"
2. "React 学习笔记"
3. "修复登录 bug"

请告诉我每项的分类：
- 1 → 任务(想法/进行中/挂起) / Areas / Resources / Archive？
- 2 → ？
- 3 → ？

用户：1→想法，2→resources，3→进行中

Claude：完成！
- 💡 新的营销方案 → 任务
- React 学习笔记 → Resources
- 🔥 修复登录 bug → 任务
```

### 4.3 状态概览

```
用户：查看 GTD 状态

Claude：
📊 GTD 系统状态

🎯 任务：
- 🔥 进行中：3 项
- 💡 想法：5 项
- 💤 挂起：8 项

📥 Inbox：2 项待整理
🏢 Areas：3 个领域
📖 Resources：12 篇资料
🗄️ Archive：25 项

⚠️ 建议：Inbox 有 2 项待整理，建议尽快处理。
```

### 4.4 归档已完成

```
用户：归档 修复登录 bug

Claude：找到匹配页面：「🔥 修复登录 bug」
正在移动到 Archive...

✅ 已归档：修复登录 bug
```

---

## 第五部分：最佳实践

### 5.1 每日工作流

| 时间 | 操作 |
|------|------|
| 早晨 | 快速捕获隔夜的想法到 Inbox |
| 全天 | 捕获一切到 Inbox（先不分类） |
| 一天结束 | 将 Inbox 处理至清空 |

### 5.2 每周回顾

1. **查看状态**：检查所有分类
2. **处理想法**：将可行的想法移到进行中
3. **回顾挂起**：重新评估暂停的项目
4. **归档已完成**：清理完成的任务
5. **反思领域**：责任是否平衡？

### 5.3 前缀纪律

**始终使用前缀**标记任务：
- 使扫描即时化
- 启用过滤功能
- 提供一目了然的视觉状态

**差**："完成报告"
**好**："🔥 完成 Q4 报告"

### 5.4 Inbox 清零

Inbox **不是存储**——它是**处理队列**。

- 项目不应停留超过 24 小时
- 如果不确定，默认放到 💡 想法
- 有疑问时，先捕获——稍后澄清

---

## 第六部分：进阶配置

### 6.1 多领域设置

对于复杂的责任，创建子领域：

```
🏢 Areas
├── 工作
│   ├── 项目 A
│   └── 项目 B
└── 个人
    ├── 健康
    └── 财务
```

### 6.2 Playbook 集成

存储 SOP 和方法论：

```
📋 Playbook
├── 会议模板
├── 代码审查清单
├── 部署 SOP
└── 事故响应
```

### 6.3 自动化扩展

与其他技能组合：

| 触发 | 技能链 |
|------|--------|
| PRD 完成 | `/doc-coauthoring` → `gtd-notion`（自动创建任务） |
| 研究完成 | `/notion-knowledge-capture` → `gtd-notion`（归档到 Resources） |
| 每周回顾 | `gtd-notion` → Telegram 通知（发送汇总） |

---

## 总结

GTD + PARA 双系统提供：

| 收益 | 描述 |
|------|------|
| **认知卸载** | 捕获一切，信任系统 |
| **零摩擦** | 自然语言 → 有序结构 |
| **视觉清晰** | Emoji 前缀即时识别 |
| **跨平台** | Notion 处处同步 |
| **AI 驱动** | Claude 处理分类逻辑 |

**关键洞察**：力量不在于单一系统——而在于它们的组合：
- GTD 提供**工作流**（捕获 → 澄清 → 组织 → 执行）
- PARA 提供**结构**（东西放在哪里）
- Claude 提供**自动化**（自然语言界面）
- Notion 提供**持久化**（跨设备同步）

---

## 参考资料

- [Getting Things Done](https://gettingthingsdone.com/) - David Allen
- [PARA 方法](https://fortelabs.com/blog/para/) - Tiago Forte
- [Notion MCP Server](https://github.com/notionhq/notion-mcp-server)
- [Claude Code 技能文档](https://docs.anthropic.com/claude-code/skills)
