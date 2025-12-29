# Codex CLI Integration in Claude Code

> Skill + Subagent Architecture: Invoke OpenAI Codex for code generation, debugging, and refactoring within Claude Code

**Document ID**: 009
**Date**: 2025-12-29
**Tags**: `Codex CLI` `OpenAI` `Claude Code` `Skill` `Code Generation`

---

## Overview

OpenAI Codex CLI is a locally-running AI coding assistant powered by GPT-5 level reasoning. Through Skill + Subagent architecture, we can seamlessly invoke Codex within Claude Code, enabling:

- **Code Generation** — Generate complete code from natural language
- **Smart Debugging** — Locate and fix complex bugs
- **Code Refactoring** — Batch refactor code styles
- **Screenshot Implementation** — Generate UI code directly from design mockups

---

## Why Codex CLI?

### Claude Code vs Codex CLI

| Dimension | Claude Code | Codex CLI |
|-----------|-------------|-----------|
| **Model** | Claude Opus/Sonnet | GPT-5-Codex |
| **Code Ability** | Excellent | Top-tier |
| **Multimodal** | Via subagent | Native screenshot support |
| **Local Execution** | ✅ | ✅ |
| **MCP Support** | ✅ | ✅ |

### When to Use Codex?

| Scenario | Recommended | Reason |
|----------|-------------|--------|
| Code generation/refactoring | **Codex** | GPT-5 superior code ability |
| Complex bug debugging | **Codex** | Deep reasoning capability |
| UI from screenshot | **Codex** | Native multimodal + code gen |
| Long document analysis | Gemini | 1M token context |
| Video/audio processing | Gemini | More comprehensive multimodal |

---

## Part 1: Installation & Configuration

### 1.1 Install Codex CLI

**Method 1: npm (Recommended)**

```bash
npm install -g @openai/codex
```

**Method 2: Homebrew (macOS)**

```bash
brew install --cask codex
```

**Method 3: Manual Download**

Download from [GitHub Releases](https://github.com/openai/codex/releases/latest):
- macOS (Apple Silicon): `codex-darwin-arm64`
- macOS (Intel): `codex-darwin-x86_64`
- Linux: `codex-linux-x86_64` / `codex-linux-arm64`

### 1.2 Verify Installation

```bash
codex --version
# Output: codex-cli 0.77.0
```

### 1.3 Authentication

**Method 1: ChatGPT Account Login (Recommended)**

```bash
codex
# First launch prompts for login
# Supports Plus, Pro, Team, Edu, Enterprise accounts
```

**Method 2: API Key**

```bash
export OPENAI_API_KEY="sk-xxx"
```

### 1.4 Configuration File

Location: `~/.codex/config.toml`

```toml
# Default model
model = "gpt-5-codex"

# Approval mode
# suggest: confirm each time (default)
# auto-edit: auto-edit files, confirm commands
# full-auto: fully automatic
approval_mode = "suggest"

# Reasoning level
reasoning_level = "medium"

# MCP server configuration
[mcp.servers.filesystem]
command = "npx"
args = ["-y", "@anthropic-ai/mcp-server-filesystem", "/Users/dev/Projects"]
```

---

## Part 2: Skill Architecture

### 2.1 Why Skill + Subagent?

```
User: "Use Codex to refactor this code"
         ↓
Claude Code main conversation
         ↓
Detect codex-cli skill trigger
         ↓
Launch general-purpose subagent
         ↓
Subagent executes Codex CLI
         ↓
Results return to main (context isolated)
```

**Benefits**:
- Codex output processed in subagent, doesn't pollute main conversation
- Avoids token waste
- Keeps main conversation clean

### 2.2 Create codex-cli Skill

Create `codex-cli/SKILL.md` in skill library:

```markdown
---
name: codex-cli
description: Invoke OpenAI Codex CLI for coding tasks. Triggers when user says "use Codex to write", "Codex debug", "Codex refactor", etc.
---

# Codex CLI Programming Assistant

When users need OpenAI Codex for coding tasks, invoke Codex CLI through subagent.

## Trigger Scenarios

- "Use Codex to write a..."
- "Have Codex debug this bug..."
- "Codex help me refactor..."
- "Use Codex to analyze this screenshot..."

## Invoke Subagent

Use Task tool to call general-purpose subagent:

Task tool parameters:
- subagent_type: "general-purpose"
- prompt: Tell subagent to execute codex command
- description: Brief description
```

### 2.3 Trigger Keywords

| User Says | Triggers |
|-----------|----------|
| "Use Codex to write..." | ✅ |
| "Codex debug this bug" | ✅ |
| "Have GPT-5 handle..." | ✅ |
| "Codex implement from screenshot" | ✅ |
| "Use OpenAI CLI to do..." | ✅ |

---

## Part 3: Usage Examples

### 3.1 Code Generation

**User Request**:
```
Use Codex to write a React Hook for form state management with validation
```

**Subagent Executes**:
```bash
cd /Users/dev/Projects/my-app && codex "Create a useForm Hook with field validation, error display, submit handling"
```

### 3.2 Code Debugging

**User Request**:
```
Have Codex check why src/utils/parser.ts crashes on empty input
```

**Subagent Executes**:
```bash
cd /Users/dev/Projects/my-app && codex "src/utils/parser.ts has a bug, crashes on empty input, locate and fix"
```

### 3.3 Code Refactoring

**User Request**:
```
Use Codex to convert all classes in src/services/ to functional style
```

**Subagent Executes**:
```bash
cd /Users/dev/Projects/my-app && codex "Refactor all classes in src/services/ to functional components using hooks"
```

### 3.4 UI from Screenshot

**User Request**:
```
Codex implement login page from this design: /Users/dev/Downloads/login-design.png
```

**Subagent Executes**:
```bash
codex "Implement a React login page component based on this design" --image /Users/dev/Downloads/login-design.png
```

### 3.5 Generate Test Cases

**User Request**:
```
Have Codex generate unit tests for functions in src/utils/
```

**Subagent Executes**:
```bash
cd /Users/dev/Projects/my-app && codex "Generate Jest unit tests for all functions in src/utils/"
```

---

## Part 4: Advanced Usage

### 4.1 Command Line Options

```bash
# Specify model
codex "task" --model gpt-5

# Attach screenshot
codex "implement UI" --image ./design.png

# Quiet mode
codex "refactor" --quiet

# Auto-execute (use with caution)
codex "fix" --approval-mode full-auto

# Deep reasoning
codex "analyze complex logic" --reasoning high
```

### 4.2 Interactive Mode Commands

In Codex interactive mode:

```
/model          Switch model (gpt-5-codex / gpt-5)
/reasoning      Adjust reasoning level (low / medium / high)
/help           Show help
/clear          Clear context
/exit           Exit
```

### 4.3 MCP Extensions

Codex supports Model Context Protocol for connecting more tools:

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

## Part 5: Codex vs Gemini Selection Guide

Both CLI tools have their strengths, choose based on scenario:

| Scenario | Recommended | Reason |
|----------|-------------|--------|
| **Code Generation** | Codex | GPT-5 top-tier code ability |
| **Complex Debugging** | Codex | Strong deep reasoning |
| **UI from Screenshot** | Codex | Code gen + multimodal |
| **Long Documents** | Gemini | 1M token context |
| **Video Analysis** | Gemini | Native video understanding |
| **Audio Transcription** | Gemini | Native audio processing |
| **Project Scanning** | Gemini | Long context advantage |

### Combined Usage Example

```
1. Use Gemini to scan entire project, understand architecture
2. Use Codex to generate/refactor code for specific modules
3. Use Gemini to analyze overall PR impact
```

---

## FAQ

### Q1: Authentication Failed

**Solution**:
```bash
rm -rf ~/.codex/auth
codex  # Re-login
```

### Q2: Command Execution Rejected

**Cause**: Default approval_mode is suggest

**Solution**:
```bash
# Temporary auto mode
codex "task" --approval-mode full-auto

# Or modify config
# ~/.codex/config.toml
# approval_mode = "auto-edit"
```

### Q3: Response Too Slow

**Solution**:
```bash
# Lower reasoning level
codex "task" --reasoning low
```

### Q4: Skill Not Triggering

**Check**:
```bash
# Confirm skill is enabled
ls ~/.claude/skills/codex-cli
```

---

## Best Practices

### Do's

- Execute Codex in specific project directories
- Use AGENTS.md to maintain project context
- Use `--reasoning high` for complex tasks
- Review generated code

### Don'ts

- Don't include sensitive info in prompts
- Don't use `full-auto` in production
- Don't skip code review steps

---

## References

- [OpenAI Codex CLI Official Docs](https://developers.openai.com/codex/cli/)
- [GitHub Repository](https://github.com/openai/codex)
- [CLI Command Reference](https://developers.openai.com/codex/cli/reference/)
- [005 - Gemini CLI Multimodal Integration](./005-gemini-cli-multimodal-integration.md)
