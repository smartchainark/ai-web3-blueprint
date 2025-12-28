# 内部沟通到多格式导出：完整工作流 SOP

> 从生成项目更新到 PPT/PDF/Word 导出再到 Telegram 通知的完整闭环

**文档编号**: 002
**日期**: 2025-12-28
**标签**: `Claude Code` `内部沟通` `文档导出` `工作流`

---

## 概述

本文档演示如何通过 Claude Code 技能组合，实现内部沟通材料的自动化生成和多格式导出：

| 步骤 | 技能 | 说明 |
|------|------|------|
| 1. 生成项目更新 | `/internal-comms` | 生成 3P 更新（Progress/Plans/Problems） |
| 2. 创建演示文稿 | `/pptx` | 转换为 PowerPoint 演示文稿 |
| 3. 生成 PDF | `/pdf` | 转换为 PDF 文档 |
| 4. 生成 Word 文档 | `/docx` | 转换为 Word 文档 |
| 5. 团队通知 | Telegram Bot API | 推送更新摘要到团队群组 |

```
/internal-comms → /pptx → /pdf → /docx → Telegram 通知
```

**场景延续**：本文档延续 001 文档的 NotebookLM 项目场景，展示从需求确定后如何进行项目进度同步和团队沟通。

---

## 第一部分：生成内部沟通材料

### 1.1 什么是 3P 更新

3P 更新是一种简洁的项目状态汇报格式，适用于向领导层、跨团队或利益相关方同步进度：

- **Progress（进展）**: 本周完成了什么
- **Plans（计划）**: 下周要做什么
- **Problems（问题）**: 当前遇到的阻塞或挑战

### 1.2 使用 internal-comms 技能

internal-comms 技能支持多种内部沟通格式：

| 类型 | 说明 | 模板文件 |
|------|------|----------|
| 3P 更新 | 进展/计划/问题 | `examples/3p-updates.md` |
| 公司通讯 | 公司级别新闻稿 | `examples/company-newsletter.md` |
| FAQ 回答 | 常见问题解答 | `examples/faq-answers.md` |
| 通用沟通 | 其他类型 | `examples/general-comms.md` |

### 1.3 3P 更新格式规范

```
[表情符号] [团队名称] (日期范围)

Progress: [1-3 句话，描述本周完成的工作]
Plans: [1-3 句话，描述下周的计划]
Problems: [1-3 句话，描述当前遇到的问题]
```

**要点**：
- 30-60 秒内可读完
- 数据驱动，包含具体指标
- 语气简洁明了，不要过度修饰

### 1.4 示例：NotebookLM 项目 3P 更新

```markdown
📓 **NotebookLM 集成助手团队** (2025-12-23 ~ 2025-12-28)

**Progress**: 完成了产品 PRD 撰写，通过 Claude Code 技能自动创建了
Notion 项目页面和 8 个任务条目，涵盖认证管理（3 个任务）、笔记本库
管理（3 个任务）和查询功能（2 个任务），并通过 Telegram Bot 向团队
推送了任务摘要。

**Plans**: 下周启动认证模块开发，完成 Patchright 浏览器自动化框架
搭建，实现 Google 账号 OAuth 登录流程，预计完成 P0 级认证功能。

**Problems**: 无阻塞项。当前进度符合预期，开发资源已到位。
```

---

## 第二部分：多格式文档导出

### 2.1 导出为 PowerPoint（/pptx 技能）

#### 使用方式

```
/pptx 将 3P 更新转换为演示文稿，保存到 ./exports/project-update.pptx
```

#### 技能特点

- 支持从零创建或基于模板
- 自动设计配色和布局
- 支持图表、表格等元素
- 使用 html2pptx 或 PptxGenJS 实现

#### 推荐幻灯片结构

| 幻灯片 | 内容 |
|--------|------|
| 1 | 标题页：团队名称 + 日期 |
| 2 | Progress：本周进展 |
| 3 | Plans：下周计划 |
| 4 | Problems：当前问题 |
| 5 | 里程碑表格 |
| 6 | 结束页 |

### 2.2 导出为 PDF（/pdf 技能）

#### 使用方式

```
/pdf 将 3P 更新转换为 PDF 文档，保存到 ./exports/project-update.pdf
```

#### 技能特点

- 使用 reportlab 创建专业 PDF
- 支持中文字体（PingFang）
- 包含表格、样式、分页
- 适合正式文档存档

### 2.3 导出为 Word 文档（/docx 技能）

#### 使用方式

```
/docx 将 3P 更新转换为 Word 文档，保存到 ./exports/project-update.docx
```

#### 技能特点

- 使用 docx-js 创建
- 保留格式和结构
- 支持表格、列表、样式
- 可进一步编辑

---

## 第三部分：Telegram 通知

### 3.1 发送 3P 更新摘要

```bash
BOT_TOKEN="你的Bot Token"
CHAT_ID="你的Chat ID"

MESSAGE="📓 *NotebookLM 集成助手 - 周报更新*

📅 2025-12-23 ~ 2025-12-28

✅ *Progress*:
• 完成 PRD 撰写
• 创建 Notion 项目页面和 8 个任务
• 完成团队通知配置

📋 *Plans*:
• 启动认证模块开发
• 搭建 Patchright 浏览器自动化

⚠️ *Problems*:
• 无阻塞项

📎 附件已生成: PPT / PDF / Word"

curl -s -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendMessage" \
  -H "Content-Type: application/json" \
  -d "{
    \"chat_id\": \"$CHAT_ID\",
    \"text\": \"$MESSAGE\",
    \"parse_mode\": \"Markdown\"
  }"
```

### 3.2 发送文件附件（可选）

```bash
# 发送 PDF 文件
curl -s -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendDocument" \
  -F "chat_id=$CHAT_ID" \
  -F "document=@./exports/project-update.pdf" \
  -F "caption=📄 NotebookLM 项目周报 (PDF)"
```

---

## 第四部分：完整工作流示例

### 4.1 在 Claude Code 中的完整流程

```
# 步骤 1: 生成 3P 更新
用户: 为 NotebookLM 项目生成本周的 3P 更新

Claude: [使用 internal-comms 技能格式，生成 3P 更新并保存]
✅ 已保存到 ./drafts/notebooklm-3p-update.md

# 步骤 2: 转换为 PPT
用户: /pptx 将 3P 更新转换为演示文稿

Claude: [创建 PowerPoint 文件]
✅ 已保存到 ./exports/002-project-update.pptx

# 步骤 3: 转换为 PDF
用户: /pdf 也生成一份 PDF 版本

Claude: [创建 PDF 文件]
✅ 已保存到 ./exports/002-project-update.pdf

# 步骤 4: 转换为 Word
用户: /docx 再生成一份 Word 版本

Claude: [创建 Word 文件]
✅ 已保存到 ./exports/002-project-update.docx

# 步骤 5: 发送 Telegram 通知
用户: 把更新摘要发送到 Telegram

Claude: [发送通知]
✅ 消息已发送
```

### 4.2 工作流总结

| 步骤 | 技能/工具 | 输入 | 输出 |
|------|-----------|------|------|
| 1. 生成更新 | `/internal-comms` | 项目信息 | Markdown 3P 更新 |
| 2. 创建 PPT | `/pptx` | 3P 更新 | .pptx 文件 |
| 3. 创建 PDF | `/pdf` | 3P 更新 | .pdf 文件 |
| 4. 创建 Word | `/docx` | 3P 更新 | .docx 文件 |
| 5. 通知团队 | Telegram API | 摘要 | 消息推送 |

---

## 第五部分：技能依赖

### 5.1 internal-comms 技能

这是一个指南型技能，提供内部沟通材料的格式规范：

- 位置: `~/.claude/skills/internal-comms/`
- 类型: 格式指南（非调用型）
- 模板: 3P 更新、公司通讯、FAQ 等

### 5.2 文档导出技能

| 技能 | 依赖 | 说明 |
|------|------|------|
| `/pptx` | pptxgenjs, playwright, sharp | PowerPoint 创建 |
| `/pdf` | reportlab (Python) | PDF 创建 |
| `/docx` | docx (npm) | Word 文档创建 |

### 5.3 环境准备

```bash
# Node.js 依赖
npm install pptxgenjs docx sharp

# Python 依赖（建议使用 venv）
python3 -m venv venv
source venv/bin/activate
pip install reportlab
```

---

## 常见问题

### Q1: internal-comms 技能如何调用？

**说明**：internal-comms 是指南型技能，不是直接调用的命令。Claude 会根据你的请求自动参考相应的格式模板。

**使用方式**：直接描述你的需求，如"生成项目 3P 更新"或"写一份团队周报"。

### Q2: PDF 创建时中文乱码

**原因**：未正确配置中文字体

**解决**：确保使用 PingFang 字体（macOS）或其他支持中文的字体：
```python
from reportlab.pdfbase.ttfonts import TTFont
pdfmetrics.registerFont(TTFont('PingFang', '/System/Library/Fonts/PingFang.ttc'))
```

### Q3: PPT 导出样式不理想

**建议**：
- 使用 `/pptx` 技能时，提供明确的设计要求
- 可以指定配色方案、字体等
- 先预览缩略图再调整

---

## 与 001 的关系

本文档（002）是 001 文档的延续：

| 文档 | 阶段 | 工作流 |
|------|------|--------|
| 001 | 需求阶段 | PRD → Notion 任务 → 通知 |
| 002 | 执行阶段 | 3P 更新 → 文档导出 → 通知 |

两个工作流构成了完整的项目管理闭环：

```
001: 需求收集 → 任务拆解 → 团队同步
         ↓
002: 进度跟踪 → 状态汇报 → 多格式分发
```

---

## 参考资源

- [reportlab 文档](https://docs.reportlab.com/)
- [pptxgenjs 文档](https://gitbrent.github.io/PptxGenJS/)
- [docx 文档](https://docx.js.org/)
- [Telegram Bot API](https://core.telegram.org/bots/api)
