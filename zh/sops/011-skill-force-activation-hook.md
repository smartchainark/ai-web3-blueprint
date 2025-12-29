# Claude Code Skills 强制激活 Hook

> 通过 UserPromptSubmit Hook 将技能激活成功率从 20% 提升到 84%

**文档编号**: 011
**日期**: 2025-12-29
**标签**: `Claude Code` `Skills` `Hook` `激活率` `承诺机制`

---

## 概述

Claude Code Skills 设计为"自动激活"，但实际成功率只有约 **20%**。国外开发者 Scott Spence 通过 200+ 次系统测试，找到了将成功率提升到 **84%** 的方案。

本文档记录如何配置这个强制激活 Hook。

---

## 问题分析

### 为什么原生激活率这么低？

官方文档说 Claude 会"根据请求自主决定何时使用技能"，但实际情况是：

- Claude 经常**忽略**配置的 Skills
- 直接按自己的理解开始写代码
- 技能激活像"看运气"

### 简单指令为什么不够？

很多人尝试在 Hook 里加一句：

```bash
echo 'INSTRUCTION: If the prompt matches any available skill keywords,
use Skill(skill-name) to activate it.'
```

成功率只有 **50%**——这只是个**被动建议**，Claude 看到后容易忘记。

---

## 解决方案：强制评估 Hook

### 核心原理：承诺机制

强制评估创建**三步承诺机制**：

```
1. 评估 — 对每个技能明确说明 是/否 及理由
2. 激活 — 立即调用 Skill(技能名)
3. 实现 — 只有激活后才能开始
```

一旦 Claude 在回复中写下"是 - 需要代码生成"，它就**承诺**要激活那个技能。

### 测试结果对比

| 方案 | 成功率 |
|------|--------|
| 无 Hook（原生） | ~20% |
| 简单指令 | ~50% |
| **强制评估 Hook** | **84%** ✅ |

基于 5 种典型任务，每种跑 10 次：

| 任务类型 | 简单指令 | 强制评估 |
|----------|----------|----------|
| 表单路由创建 | 0% | 80% |
| 数据加载 | 0% | 100% |
| 服务端操作 | 60% | 40% |
| 远程函数 | 100% | 100% |
| 响应式状态 | 100% | 100% |

---

## 安装配置

### 第一步：创建 Hook 脚本

```bash
# 创建目录
mkdir -p ~/.claude/skills/skill-force-activation/scripts

# 创建脚本
cat > ~/.claude/skills/skill-force-activation/scripts/force-skill-eval.sh << 'EOF'
#!/bin/bash
# UserPromptSubmit Hook - 强制技能评估

cat <<'INSTRUCTION'
INSTRUCTION: Mandatory Skill Activation Protocol

Step 1 - EVALUATE (Must complete in your response):
For each skill in <available_skills>, state: [skill-name] - YES/NO - [reason]

Step 2 - ACTIVATE (Immediately after Step 1):
If any skill is "YES" → Immediately use Skill(skill-name) tool for each relevant skill
If all skills are "NO" → State "No skills needed" and proceed

Step 3 - IMPLEMENT:
Only begin implementation AFTER Step 2 is complete.

CRITICAL: You MUST call the Skill() tool in Step 2. Do NOT skip to implementation.
Evaluation (Step 1) is WORTHLESS without activation (Step 2).

Correct flow example:
- research: NO - not a research task
- gemini-cli: YES - need multimodal processing
- codex-cli: YES - need code generation

[Then IMMEDIATELY use the Skill() tool:]
> Skill(gemini-cli)
> Skill(codex-cli)

[Only AFTER these are complete, begin implementation]
INSTRUCTION
EOF

# 设置执行权限
chmod +x ~/.claude/skills/skill-force-activation/scripts/force-skill-eval.sh
```

### 第二步：配置 settings.json

编辑 `~/.claude/settings.json`，添加 `UserPromptSubmit` Hook：

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/skills/skill-force-activation/scripts/force-skill-eval.sh"
          }
        ]
      }
    ]
  }
}
```

如果已有其他 hooks，只需添加 `UserPromptSubmit` 部分：

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/skills/skill-force-activation/scripts/force-skill-eval.sh"
          }
        ]
      }
    ],
    "Stop": [
      ...existing hooks...
    ]
  }
}
```

### 第三步：重启 Claude Code

配置完成后，重启 Claude Code 使 Hook 生效。

---

## 工作流程

```
用户发送消息
       ↓
UserPromptSubmit Hook 触发
       ↓
注入强制评估指令
       ↓
Claude 必须评估每个技能（是/否 + 理由）
       ↓
Claude 激活匹配的技能
       ↓
开始实现任务
```

### 实际效果示例

用户输入：
```
帮我创建一个表单组件
```

Claude 响应：
```
技能评估：
- frontend-design: YES - 需要创建 UI 组件
- typescript-write: YES - 需要编写 TypeScript 代码
- research: NO - 不是研究任务

> Skill(frontend-design)
> Skill(typescript-write)

现在开始实现表单组件...
```

---

## 为什么这个方案有效？

### 关键词的力量

激进的用词让 Claude 更难忽略指令：

- **"CRITICAL"** — 强调重要性
- **"MUST"** — 强制要求
- **"WORTHLESS"** — 否定不执行的价值

### 承诺的力量

一旦 Claude 在回复中写下：
```
- codex-cli: YES - need code generation
```

它就**公开承诺**要激活这个技能，跳过会显得自相矛盾。

### 结构化流程

明确的三步流程让 Claude 无法"跳过"：

1. 必须先**评估**（在回复中可见）
2. 必须再**激活**（调用 Skill 工具）
3. 最后才能**实现**

---

## 注意事项

### 优点

- 成功率从 20% 提升到 84%
- 无需修改任何技能文件
- 一次配置，全局生效
- 不影响其他 Hooks

### 代价

- 每次回复会多消耗一些 token（评估部分）
- 回复开头会有技能评估内容
- 偶尔仍有 16% 的失败率

### 最佳实践

1. **保持技能描述清晰** — 让 Claude 更容易匹配
2. **技能名称要有意义** — 如 `frontend-design` 而非 `fd`
3. **定期检查激活情况** — 确认 Hook 正常工作

---

## 替代方案：LLM 预评估

另一个方案是用 Claude API 预先评估技能匹配：

```bash
#!/bin/bash
RESPONSE=$(curl -s https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -d '{
    "model": "claude-haiku-4-5-20251001",
    "messages": [{"role": "user", "content": "返回匹配的技能 JSON 数组..."}]
  }')
```

**优点**：每次便宜 10%，快 17%
**缺点**：不稳定，可能完全漏掉某些任务

推荐使用**强制评估 Hook**，更稳定可靠。

---

## 常见问题

### Q1: Hook 没有触发？

检查脚本路径和权限：
```bash
ls -la ~/.claude/skills/skill-force-activation/scripts/force-skill-eval.sh
```

### Q2: 评估后没有激活？

确认 settings.json 格式正确，Hook 配置在正确位置。

### Q3: 想禁用 Hook？

注释掉或删除 settings.json 中的 `UserPromptSubmit` 部分。

---

## 参考资源

- [原文：Claude Code Skills 100% 激活](https://scottspence.com/posts/claude-code-skills-activation)
- [Scott Spence 的测试框架](https://github.com/spences10/claude-skills-testing)
- [004 - Claude Skills 跨平台部署](./004-claude-skills-setup.md)
- [006 - Claude 技能创建规范化](./006-claude-skill-creation-sop.md)
