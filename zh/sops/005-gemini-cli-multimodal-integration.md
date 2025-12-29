# Gemini CLI 多模态集成

> 一键调用 Gemini CLI，实现超长上下文 + 免费多模态处理

**文档编号**: 005
**日期**: 2025-12-29
**标签**: `Claude Code` `Gemini CLI` `多模态` `免费` `子代理`
**系列**: 本地 AI CLI 系列 (1/3)

---

> 📚 **系列导航**
> - **005 - Gemini CLI 多模态处理（当前）**
> - [009 - Codex CLI 代码生成](./009-codex-cli-integration.md)
> - [010 - 本地 AI 军团 - 免费多Agent并行](./010-multi-agent-collaboration.md)

## 概述

本文档介绍如何在 Claude Code 中集成 Gemini CLI，实现多模态文件处理（图片、视频、音频、PDF）和超长上下文分析能力。

### 核心架构

```
用户请求 → Skill 识别意图 → Task 工具 → general-purpose 子代理 → Gemini CLI
                                              ↓
                                    上下文隔离，结果返回主对话
```

### 为什么这样设计？

| 问题 | 解决方案 |
|------|---------|
| Claude 无法直接处理图片/视频/音频 | Gemini CLI 原生多模态支持 |
| Claude 上下文有限 | Gemini 100 万 token 上下文 |
| 直接调用会污染主对话上下文 | 子代理隔离执行 |
| 需要付费 API | Gemini CLI 有免费额度 |

---

## 第一部分：前置条件

### 1.1 安装 Gemini CLI

```bash
# 使用 npm 安装
npm install -g @google/gemini-cli

# 验证安装
gemini --version
```

### 1.2 配置 API Key

```bash
# 设置环境变量
export GOOGLE_API_KEY="your-api-key"

# 或在 ~/.bashrc / ~/.zshrc 中添加
echo 'export GOOGLE_API_KEY="your-api-key"' >> ~/.zshrc
```

### 1.3 确认 Skill 已启用

```bash
# 检查技能是否启用
ls ~/.claude/skills/gemini-cli/SKILL.md

# 如果使用自定义技能库
ls ~/Projects/my_skils/enabled/gemini-cli
```

---

## 第二部分：Gemini CLI 命令参考

### 2.1 基本语法

```bash
gemini "prompt" [file1] [file2] ... [options]
```

### 2.2 关键参数

| 参数 | 说明 | 必须 |
|------|------|------|
| `"prompt"` | 提示词（位置参数） | ✅ |
| `-y` / `--yolo` | 自动确认，非交互模式 | ✅ |
| `-m model` | 指定模型 | ❌ |
| `file1 file2 ...` | 要分析的文件路径 | ❌ |

### 2.3 常用命令示例

```bash
# 分析单个图片
gemini "描述这张图片的内容" /path/to/image.png -y

# 分析多个文件
gemini "比较这两张图片的差异" img1.png img2.png -y

# 分析视频
gemini "总结这段视频的主要内容" /path/to/video.mp4 -y

# 分析音频
gemini "提取这段录音的关键信息" /path/to/audio.mp3 -y

# 分析 PDF
gemini "总结这份文档的要点" /path/to/document.pdf -y

# 分析整个项目（先 cd 进去）
cd /path/to/project && gemini "分析项目架构，列出主要模块" -y

# 分析代码文件
gemini "审查这段代码的安全性" /path/to/code.js -y
```

---

## 第三部分：Claude Code 集成方式

### 3.1 技术实现

使用 **Skill + Task 工具 + general-purpose 子代理** 架构：

1. **Skill (`gemini-cli/SKILL.md`)** — 定义触发条件和工作流程
2. **Task 工具** — 创建隔离的子代理执行环境
3. **general-purpose 子代理** — 执行 Gemini CLI 命令

### 3.2 为什么用 general-purpose 子代理？

- Claude Code 的 Task 工具只支持预定义的子代理类型
- `general-purpose` 是通用执行器，可以运行任意 Bash 命令
- 子代理有独立上下文，Gemini 的超长输出不会污染主对话

### 3.3 调用模板

```yaml
Task 工具参数：
  subagent_type: "general-purpose"
  description: "Gemini 分析图片"  # 简短描述
  prompt: |
    使用 Bash 执行 Gemini CLI 分析图片：
    gemini "描述这张图片的内容" /Users/dev/Downloads/screenshot.png -y
    返回分析结果。
```

---

## 第四部分：触发场景

Skill 会在以下场景自动触发：

| 用户说... | 触发 |
|-----------|------|
| "用 Gemini 分析这个..." | ✅ |
| "Gemini 帮我看看这张图..." | ✅ |
| "让 Gemini 总结这段视频..." | ✅ |
| "Gemini 帮我提取这个音频的要点..." | ✅ |
| "用 Gemini 读这份 PDF..." | ✅ |
| "Gemini 扫描一下这个项目..." | ✅ |
| "让 Gemini 分析整个代码库..." | ✅ |

---

## 第五部分：支持的文件类型

| 类型 | 扩展名 | 示例用途 |
|------|--------|---------|
| 图片 | `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp` | 截图分析、设计审查 |
| 视频 | `.mp4`, `.mov`, `.avi`, `.mkv`, `.webm` | 视频总结、内容提取 |
| 音频 | `.mp3`, `.wav`, `.m4a`, `.ogg`, `.flac` | 会议记录、语音转文字 |
| 文档 | `.pdf`, `.txt`, `.md`, `.csv` | 文档总结、数据分析 |
| 代码 | 任意代码文件 | 代码审查、架构分析 |

---

## 第六部分：完整示例

### 示例 1: 分析截图

用户请求：
> "用 Gemini 帮我看看这张截图 /Users/dev/Downloads/error.png 是什么错误"

Claude 执行：
```yaml
Task:
  subagent_type: general-purpose
  description: Gemini 分析错误截图
  prompt: |
    使用 Bash 执行 Gemini CLI：
    gemini "分析这张截图中的错误信息，告诉我是什么问题以及如何解决" /Users/dev/Downloads/error.png -y
    返回完整分析结果。
```

### 示例 2: 扫描项目架构

用户请求：
> "让 Gemini 扫描一下 ~/Projects/my-app 这个项目的架构"

Claude 执行：
```yaml
Task:
  subagent_type: general-purpose
  description: Gemini 分析项目架构
  prompt: |
    使用 Bash 执行 Gemini CLI：
    cd ~/Projects/my-app && gemini "分析这个项目的整体架构，包括：1. 技术栈 2. 目录结构 3. 主要模块 4. 数据流向" -y
    返回完整分析结果。
```

### 示例 3: 总结会议录音

用户请求：
> "Gemini 帮我提取这个会议录音的要点 /Users/dev/Documents/meeting.mp3"

Claude 执行：
```yaml
Task:
  subagent_type: general-purpose
  description: Gemini 提取会议要点
  prompt: |
    使用 Bash 执行 Gemini CLI：
    gemini "听这段会议录音，提取：1. 主要讨论议题 2. 关键决定 3. 行动事项 4. 待跟进问题" /Users/dev/Documents/meeting.mp3 -y
    返回完整分析结果。
```

---

## 注意事项

### 安全考虑

⚠️ **不要发送敏感文件给 Gemini：**
- `.env` 环境变量文件
- `credentials.json` 凭证文件
- API 密钥文件
- 私钥文件（`.pem`, `.key`）

### 性能考虑

- 大文件（视频、长音频）处理时间较长，耐心等待
- 项目扫描建议先 `cd` 进目录，让 Gemini 读取相关文件
- 超大项目可能触发 API 限制

---

## 常见问题

| 问题 | 解决方案 |
|------|---------|
| 命令卡住不动 | 确保加了 `-y` 参数 |
| 找不到 gemini 命令 | 检查是否正确安装并在 PATH 中 |
| API 报错 | 检查 GOOGLE_API_KEY 是否设置 |
| 文件读取失败 | 检查文件路径是否正确，使用绝对路径 |

---

## 技能文件结构

```
library/gemini-cli/
├── SKILL.md              # 技能定义（触发条件 + 工作流程）
└── references/
    └── SOP.md            # 完整 SOP 文档
```

---

## 参考资源

- [Gemini CLI 官方文档](https://github.com/google-gemini/gemini-cli)
- [Claude Code Skills 文档](https://docs.anthropic.com/claude-code/skills)
- [原始 Notion 文章](https://www.notion.so/Claude-Code-Gemini-CLI-2d83f9a673f781778900f06cc9128c12)
