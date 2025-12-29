# Codex CLI 代码生成集成

> Skill + 子代理架构：在 Claude Code 中调用 OpenAI Codex 执行代码生成、调试、重构任务

**文档编号**: 009
**日期**: 2025-12-29
**标签**: `Codex CLI` `OpenAI` `Claude Code` `免费` `代码生成`
**系列**: 本地 AI CLI 系列 (2/3)

---

> 📚 **系列导航**
> - [005 - Gemini CLI 多模态处理](./005-gemini-cli-multimodal-integration.md)
> - **009 - Codex CLI 代码生成（当前）**
> - [010 - 本地 AI 军团 - 免费多Agent并行](./010-multi-agent-collaboration.md)

## 概述

OpenAI Codex CLI 是一个本地运行的 AI 编程助手，搭载 GPT-5 级别的代码推理能力。通过 Skill + 子代理架构，我们可以在 Claude Code 中无缝调用 Codex，实现：

- **代码生成** — 从自然语言描述生成完整代码
- **智能调试** — 定位并修复复杂 bug
- **代码重构** — 批量重构代码风格
- **截图实现** — 从设计稿直接生成 UI 代码

---

## 为什么需要 Codex CLI？

### Claude Code vs Codex CLI

| 维度 | Claude Code | Codex CLI |
|------|-------------|-----------|
| **模型** | Claude Opus/Sonnet | GPT-5-Codex |
| **代码能力** | 优秀 | 顶级 |
| **多模态** | 通过子代理 | 原生支持截图 |
| **本地执行** | ✅ | ✅ |
| **MCP 支持** | ✅ | ✅ |

### 何时用 Codex？

| 场景 | 推荐 | 原因 |
|------|------|------|
| 代码生成/重构 | **Codex** | GPT-5 代码能力更强 |
| 复杂 bug 调试 | **Codex** | 深度推理能力 |
| UI 截图实现 | **Codex** | 原生多模态 + 代码生成 |
| 超长文档分析 | Gemini | 100 万 token 上下文 |
| 视频/音频处理 | Gemini | 原生多模态更全面 |

---

## 第一部分：安装配置

### 1.1 安装 Codex CLI

**方式一：npm（推荐）**

```bash
npm install -g @openai/codex
```

**方式二：Homebrew（macOS）**

```bash
brew install --cask codex
```

**方式三：手动下载**

从 [GitHub Releases](https://github.com/openai/codex/releases/latest) 下载：
- macOS (Apple Silicon): `codex-darwin-arm64`
- macOS (Intel): `codex-darwin-x86_64`
- Linux: `codex-linux-x86_64` / `codex-linux-arm64`

### 1.2 验证安装

```bash
codex --version
# 输出: codex-cli 0.77.0
```

### 1.3 认证配置

**方式一：ChatGPT 账号登录（推荐）**

```bash
codex
# 首次启动会提示登录
# 支持 Plus、Pro、Team、Edu、Enterprise 账号
```

**方式二：API Key**

```bash
export OPENAI_API_KEY="sk-xxx"
```

### 1.4 配置文件

配置文件位置：`~/.codex/config.toml`

```toml
# 默认模型
model = "gpt-5-codex"

# 执行策略
# suggest: 每次确认（默认）
# auto-edit: 自动编辑，命令需确认
# full-auto: 完全自动
approval_mode = "suggest"

# 推理级别
reasoning_level = "medium"

# MCP 服务器配置
[mcp.servers.filesystem]
command = "npx"
args = ["-y", "@anthropic-ai/mcp-server-filesystem", "/Users/dev/Projects"]
```

---

## 第二部分：Skill 架构

### 2.1 为什么用 Skill + 子代理？

```
用户: "用 Codex 帮我重构这段代码"
         ↓
Claude Code 主对话
         ↓
识别 codex-cli 技能触发
         ↓
启动 general-purpose 子代理
         ↓
子代理执行 Codex CLI
         ↓
结果返回主对话（上下文隔离）
```

**优势**：
- Codex 的输出在子代理内处理，不污染主对话
- 避免 token 浪费
- 保持主对话简洁

### 2.2 创建 codex-cli 技能

在技能库中创建 `codex-cli/SKILL.md`：

```markdown
---
name: codex-cli
description: 调用 OpenAI Codex CLI 执行编码任务。当用户说"用 Codex 写"、"Codex 调试"、"Codex 重构"等需求时自动触发。
---

# Codex CLI 编程助手

当用户需要使用 OpenAI Codex 执行编码任务时，通过子代理调用 Codex CLI。

## 触发场景

- "用 Codex 写一个..."
- "让 Codex 调试这个 bug..."
- "Codex 帮我重构..."
- "用 Codex 分析这个截图..."

## 调用子代理

使用 Task 工具调用 general-purpose 子代理：

Task 工具参数：
- subagent_type: "general-purpose"
- prompt: 告诉子代理执行 codex 命令
- description: 简短描述
```

### 2.3 技能触发词

| 用户说 | 触发 |
|--------|------|
| "用 Codex 写一个..." | ✅ |
| "Codex 调试这个 bug" | ✅ |
| "让 GPT-5 处理..." | ✅ |
| "Codex 根据截图实现" | ✅ |
| "用 OpenAI CLI 来做" | ✅ |

---

## 第三部分：使用示例

### 3.1 代码生成

**用户请求**：
```
用 Codex 帮我写一个 React Hook，管理表单状态并支持验证
```

**子代理执行**：
```bash
cd /Users/dev/Projects/my-app && codex "创建一个 useForm Hook，支持字段验证、错误提示、提交处理"
```

### 3.2 代码调试

**用户请求**：
```
Codex 帮我看看 src/utils/parser.ts 为什么输入为空时会崩溃
```

**子代理执行**：
```bash
cd /Users/dev/Projects/my-app && codex "src/utils/parser.ts 有 bug，输入为空时崩溃，帮我定位并修复"
```

### 3.3 代码重构

**用户请求**：
```
用 Codex 把 src/services/ 下的类都改成函数式风格
```

**子代理执行**：
```bash
cd /Users/dev/Projects/my-app && codex "把 src/services/ 下所有类重构为函数式组件，使用 hooks"
```

### 3.4 从截图实现 UI

**用户请求**：
```
Codex 根据这个设计稿实现登录页面：/Users/dev/Downloads/login-design.png
```

**子代理执行**：
```bash
codex "根据这个设计稿实现一个 React 登录页面组件" --image /Users/dev/Downloads/login-design.png
```

### 3.5 生成测试用例

**用户请求**：
```
让 Codex 为 src/utils/ 下的函数生成单元测试
```

**子代理执行**：
```bash
cd /Users/dev/Projects/my-app && codex "为 src/utils/ 下所有函数生成 Jest 单元测试"
```

---

## 第四部分：高级用法

### 4.1 命令行参数

```bash
# 指定模型
codex "任务描述" --model gpt-5

# 附加截图
codex "实现 UI" --image ./design.png

# 安静模式
codex "重构" --quiet

# 自动执行（谨慎）
codex "修复" --approval-mode full-auto

# 深度推理
codex "分析复杂逻辑" --reasoning high
```

### 4.2 交互模式命令

在 Codex 交互模式下：

```
/model          切换模型（gpt-5-codex / gpt-5）
/reasoning      调整推理级别（low / medium / high）
/help           显示帮助
/clear          清空上下文
/exit           退出
```

### 4.3 MCP 扩展

Codex 支持 Model Context Protocol，可连接更多工具：

```toml
# ~/.codex/config.toml

[mcp.servers.github]
command = "npx"
args = ["-y", "@anthropic-ai/mcp-server-github"]
env = { GITHUB_TOKEN = "ghp_xxx" }

[mcp.servers.notion]
command = "npx"
args = ["-y", "@notionhq/notion-mcp-server"]
env = { NOTION_TOKEN = "ntn_xxx" }
```

---

## 第五部分：Codex vs Gemini 选择指南

两个 CLI 工具各有优势，根据场景选择：

| 场景 | 推荐 | 原因 |
|------|------|------|
| **代码生成** | Codex | GPT-5 代码能力顶级 |
| **复杂调试** | Codex | 深度推理能力强 |
| **截图实现 UI** | Codex | 代码生成 + 多模态 |
| **超长文档** | Gemini | 100 万 token 上下文 |
| **视频分析** | Gemini | 原生视频理解 |
| **音频转写** | Gemini | 原生音频处理 |
| **项目扫描** | Gemini | 超长上下文优势 |

### 组合使用示例

```
1. 用 Gemini 扫描整个项目，理解架构
2. 用 Codex 针对具体模块生成/重构代码
3. 用 Gemini 分析 PR 的整体影响
```

---

## 常见问题

### Q1: 认证失败

**解决**：
```bash
rm -rf ~/.codex/auth
codex  # 重新登录
```

### Q2: 命令执行被拒绝

**原因**：默认 approval_mode 是 suggest

**解决**：
```bash
# 临时自动模式
codex "任务" --approval-mode full-auto

# 或修改配置
# ~/.codex/config.toml
# approval_mode = "auto-edit"
```

### Q3: 响应太慢

**解决**：
```bash
# 降低推理级别
codex "任务" --reasoning low
```

### Q4: Skill 没有触发

**检查**：
```bash
# 确认技能已启用
ls ~/.claude/skills/codex-cli
```

---

## 最佳实践

### Do's

- 在特定项目目录下执行 Codex
- 使用 AGENTS.md 维护项目上下文
- 对复杂任务使用 `--reasoning high`
- 代码生成后进行审查

### Don'ts

- 不要在 prompt 中包含敏感信息
- 不要在生产环境用 `full-auto`
- 不要跳过代码审查步骤

---

## 参考资源

- [OpenAI Codex CLI 官方文档](https://developers.openai.com/codex/cli/)
- [GitHub 仓库](https://github.com/openai/codex)
- [CLI 命令参考](https://developers.openai.com/codex/cli/reference/)
- [005 - Gemini CLI 多模态集成](./005-gemini-cli-multimodal-integration.md)
