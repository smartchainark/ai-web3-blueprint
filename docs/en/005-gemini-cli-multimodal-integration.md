# Gemini CLI Multimodal Integration in Claude Code

> One-click Gemini CLI invocation for extended context + free multimodal processing

**Document ID**: 005
**Date**: 2025-12-29
**Tags**: `Claude Code` `Gemini CLI` `Multimodal` `Skills` `Subagent`

---

## Overview

This document explains how to integrate Gemini CLI into Claude Code, enabling multimodal file processing (images, videos, audio, PDFs) and extended context analysis capabilities.

### Core Architecture

```
User Request → Skill Intent Recognition → Task Tool → general-purpose Subagent → Gemini CLI
                                                              ↓
                                                Context Isolated, Results Return to Main Conversation
```

### Why This Design?

| Problem | Solution |
|---------|----------|
| Claude cannot directly process images/videos/audio | Gemini CLI native multimodal support |
| Claude has limited context | Gemini 1M token context |
| Direct calls pollute main conversation context | Subagent isolated execution |
| Requires paid API | Gemini CLI has free tier |

---

## Part 1: Prerequisites

### 1.1 Install Gemini CLI

```bash
# Install via npm
npm install -g @google/gemini-cli

# Verify installation
gemini --version
```

### 1.2 Configure API Key

```bash
# Set environment variable
export GOOGLE_API_KEY="your-api-key"

# Or add to ~/.bashrc / ~/.zshrc
echo 'export GOOGLE_API_KEY="your-api-key"' >> ~/.zshrc
```

### 1.3 Verify Skill is Enabled

```bash
# Check if skill is enabled
ls ~/.claude/skills/gemini-cli/SKILL.md

# If using custom skill library
ls ~/Projects/my_skils/enabled/gemini-cli
```

---

## Part 2: Gemini CLI Command Reference

### 2.1 Basic Syntax

```bash
gemini "prompt" [file1] [file2] ... [options]
```

### 2.2 Key Parameters

| Parameter | Description | Required |
|-----------|-------------|----------|
| `"prompt"` | Prompt text (positional argument) | ✅ |
| `-y` / `--yolo` | Auto-confirm, non-interactive mode | ✅ |
| `-m model` | Specify model | ❌ |
| `file1 file2 ...` | File paths to analyze | ❌ |

### 2.3 Common Command Examples

```bash
# Analyze single image
gemini "Describe this image" /path/to/image.png -y

# Analyze multiple files
gemini "Compare these two images" img1.png img2.png -y

# Analyze video
gemini "Summarize the main content of this video" /path/to/video.mp4 -y

# Analyze audio
gemini "Extract key information from this recording" /path/to/audio.mp3 -y

# Analyze PDF
gemini "Summarize the key points of this document" /path/to/document.pdf -y

# Analyze entire project (cd into directory first)
cd /path/to/project && gemini "Analyze the project architecture, list main modules" -y

# Analyze code file
gemini "Review this code for security issues" /path/to/code.js -y
```

---

## Part 3: Claude Code Integration

### 3.1 Technical Implementation

Uses **Skill + Task Tool + general-purpose Subagent** architecture:

1. **Skill (`gemini-cli/SKILL.md`)** — Defines trigger conditions and workflow
2. **Task Tool** — Creates isolated subagent execution environment
3. **general-purpose Subagent** — Executes Gemini CLI commands

### 3.2 Why Use general-purpose Subagent?

- Claude Code's Task tool only supports predefined subagent types
- `general-purpose` is a universal executor that can run any Bash command
- Subagent has independent context, Gemini's extended output won't pollute main conversation

### 3.3 Invocation Template

```yaml
Task Tool Parameters:
  subagent_type: "general-purpose"
  description: "Gemini analyze image"  # Brief description
  prompt: |
    Use Bash to execute Gemini CLI to analyze image:
    gemini "Describe this image" /Users/dev/Downloads/screenshot.png -y
    Return analysis results.
```

---

## Part 4: Trigger Scenarios

Skill auto-triggers in these scenarios:

| User Says... | Triggers |
|--------------|----------|
| "Use Gemini to analyze this..." | ✅ |
| "Gemini, help me look at this image..." | ✅ |
| "Let Gemini summarize this video..." | ✅ |
| "Gemini, extract key points from this audio..." | ✅ |
| "Use Gemini to read this PDF..." | ✅ |
| "Gemini, scan this project..." | ✅ |
| "Let Gemini analyze the entire codebase..." | ✅ |

---

## Part 5: Supported File Types

| Type | Extensions | Example Use Cases |
|------|------------|-------------------|
| Images | `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp` | Screenshot analysis, design review |
| Videos | `.mp4`, `.mov`, `.avi`, `.mkv`, `.webm` | Video summarization, content extraction |
| Audio | `.mp3`, `.wav`, `.m4a`, `.ogg`, `.flac` | Meeting notes, speech-to-text |
| Documents | `.pdf`, `.txt`, `.md`, `.csv` | Document summarization, data analysis |
| Code | Any code file | Code review, architecture analysis |

---

## Part 6: Complete Examples

### Example 1: Analyze Screenshot

User request:
> "Use Gemini to check this screenshot /Users/dev/Downloads/error.png for errors"

Claude executes:
```yaml
Task:
  subagent_type: general-purpose
  description: Gemini analyze error screenshot
  prompt: |
    Use Bash to execute Gemini CLI:
    gemini "Analyze the error message in this screenshot, explain the problem and how to fix it" /Users/dev/Downloads/error.png -y
    Return complete analysis results.
```

### Example 2: Scan Project Architecture

User request:
> "Let Gemini scan the ~/Projects/my-app project architecture"

Claude executes:
```yaml
Task:
  subagent_type: general-purpose
  description: Gemini analyze project architecture
  prompt: |
    Use Bash to execute Gemini CLI:
    cd ~/Projects/my-app && gemini "Analyze the overall architecture of this project, including: 1. Tech stack 2. Directory structure 3. Main modules 4. Data flow" -y
    Return complete analysis results.
```

### Example 3: Summarize Meeting Recording

User request:
> "Gemini, extract key points from this meeting recording /Users/dev/Documents/meeting.mp3"

Claude executes:
```yaml
Task:
  subagent_type: general-purpose
  description: Gemini extract meeting points
  prompt: |
    Use Bash to execute Gemini CLI:
    gemini "Listen to this meeting recording and extract: 1. Main discussion topics 2. Key decisions 3. Action items 4. Follow-up issues" /Users/dev/Documents/meeting.mp3 -y
    Return complete analysis results.
```

---

## Important Notes

### Security Considerations

⚠️ **Do not send sensitive files to Gemini:**
- `.env` environment variable files
- `credentials.json` credential files
- API key files
- Private key files (`.pem`, `.key`)

### Performance Considerations

- Large files (videos, long audio) take longer to process, be patient
- For project scanning, `cd` into the directory first for Gemini to read relevant files
- Very large projects may hit API limits

---

## FAQ

| Issue | Solution |
|-------|----------|
| Command hangs | Ensure `-y` parameter is added |
| gemini command not found | Check if properly installed and in PATH |
| API error | Check if GOOGLE_API_KEY is set |
| File read failure | Check file path is correct, use absolute paths |

---

## Skill File Structure

```
library/gemini-cli/
├── SKILL.md              # Skill definition (trigger conditions + workflow)
└── references/
    └── SOP.md            # Complete SOP document
```

---

## References

- [Gemini CLI Official Documentation](https://github.com/google-gemini/gemini-cli)
- [Claude Code Skills Documentation](https://docs.anthropic.com/claude-code/skills)
- [Original Notion Article](https://www.notion.so/Claude-Code-Gemini-CLI-2d83f9a673f781778900f06cc9128c12)
