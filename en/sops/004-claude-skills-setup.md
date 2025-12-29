# Claude Skills Cross-Platform Setup & Management Guide

> Cross-platform reusable AI Skill environment — Configure once, sync across devices, carry your AI skills everywhere

**Document ID**: 004
**Date**: 2025-12-29
**Tags**: `Claude Code` `Skills` `Cross-Platform` `Git`

---

## Overview

This document demonstrates how to deploy and manage Claude Code's skill system for cross-platform reuse and multi-device synchronization:

| Step | Action | Description |
|------|--------|-------------|
| 1. Clone repository | `git clone` | Get skill library |
| 2. Create symlink | `ln -s` | Connect to Claude |
| 3. Enable skills | Symlinks | Enable as needed |
| 4. Sync updates | `git pull` | Multi-device sync |

```
Claude Code Runtime
       ↓
~/.claude/skills/ (System read location)
       ↓ [Symlink]
my_skils/enabled/ (Enabled skills)
       ↓ [Symlinks]
my_skils/library/ (Full skill library)
       ↓
GitHub Repository (Version control)
```

**Target Environment**: macOS / Linux / Windows
**Estimated Time**: 10-15 minutes

---

## Part 1: Architecture Understanding

### 1.1 Core Concepts

| Directory | Purpose |
|-----------|---------|
| `~/.claude/skills/` | System directory where Claude Code reads skills |
| `my_skils/enabled/` | Enabled skills directory (symlink collection) |
| `my_skils/library/` | Full skill library (actual files) |

### 1.2 Design Advantages

1. **Single source of truth** — All skills stored once (in `library/`)
2. **Flexible control** — Enable/disable via symlinks
3. **Version control** — Entire repo managed by Git
4. **Cross-platform** — Symlinks supported on macOS/Linux/Windows

---

## Part 2: Initial Deployment

### 2.1 macOS / Linux

```bash
# Step 1: Clone repository
git clone git@github.com:smartchainark/my_skils.git ~/Projects/my_skils

# Step 2: Backup existing skills (if any)
[ -d ~/.claude/skills ] && mv ~/.claude/skills ~/.claude/skills.bak

# Step 3: Create symlink
ln -s ~/Projects/my_skils/enabled ~/.claude/skills

# Step 4: Verify
ls -la ~/.claude/skills
# Should show: ~/.claude/skills -> ~/Projects/my_skils/enabled
```

### 2.2 Windows (CMD - Administrator)

```cmd
REM Step 1: Clone repository
git clone git@github.com:smartchainark/my_skils.git D:\Projects\my_skils

REM Step 2: Backup existing skills (if any)
if exist %USERPROFILE%\.claude\skills move %USERPROFILE%\.claude\skills %USERPROFILE%\.claude\skills.bak

REM Step 3: Create Junction (recommended, no admin required)
mklink /J %USERPROFILE%\.claude\skills D:\Projects\my_skils\enabled

REM Or create symlink (requires admin or developer mode)
mklink /D %USERPROFILE%\.claude\skills D:\Projects\my_skils\enabled
```

### 2.3 Windows (PowerShell)

```powershell
# Step 1: Clone repository
git clone git@github.com:smartchainark/my_skils.git D:\Projects\my_skils

# Step 2: Backup existing skills
if (Test-Path "$env:USERPROFILE\.claude\skills") {
    Move-Item "$env:USERPROFILE\.claude\skills" "$env:USERPROFILE\.claude\skills.bak"
}

# Step 3: Create Junction
cmd /c mklink /J "$env:USERPROFILE\.claude\skills" "D:\Projects\my_skils\enabled"
```

### 2.4 Windows Developer Mode Setup

To use symlinks (instead of Junction):

1. Open **Settings** → **System** → **Developer options**
2. Enable **Developer mode**
3. Configure Git: `git config --global core.symlinks true`

---

## Part 3: Daily Usage

### 3.1 Enable a Skill

```bash
cd ~/Projects/my_skils/enabled
ln -s ../library/skill-name .

# Example: enable blockchain-developer
ln -s ../library/blockchain-developer .
```

**Windows:**
```cmd
cd D:\Projects\my_skils\enabled
mklink /D blockchain-developer ..\library\blockchain-developer
```

### 3.2 Disable a Skill

```bash
rm ~/Projects/my_skils/enabled/skill-name

# Example: disable blockchain-developer
rm ~/Projects/my_skils/enabled/blockchain-developer
```

**Windows:**
```cmd
rmdir D:\Projects\my_skils\enabled\blockchain-developer
```

### 3.3 View Skill Status

```bash
# View all available skills
ls ~/Projects/my_skils/library/

# View enabled skills
ls ~/Projects/my_skils/enabled/

# View skill details
cat ~/Projects/my_skils/library/skill-name/SKILL.md
```

### 3.4 Shell Shortcuts

Add to `~/.bashrc` or `~/.zshrc`:

```bash
alias skills-list='ls ~/Projects/my_skils/library/'
alias skills-enabled='ls ~/Projects/my_skils/enabled/'
alias skills-enable='cd ~/Projects/my_skils/enabled && ln -s ../library/'
alias skills-disable='rm ~/Projects/my_skils/enabled/'

# Usage
skills-list                        # List all skills
skills-enabled                     # List enabled
skills-enable blockchain-developer # Enable skill
skills-disable blockchain-developer # Disable skill
```

---

## Part 4: Skill Management

### 4.1 Add New Skill

```bash
# Step 1: Create skill directory
mkdir -p ~/Projects/my_skils/library/my-new-skill

# Step 2: Create SKILL.md
cat > ~/Projects/my_skils/library/my-new-skill/SKILL.md << 'EOF'
---
name: my-new-skill
description: Brief skill description
---

# My New Skill

Detailed skill instructions...
EOF

# Step 3: Enable skill
cd ~/Projects/my_skils/enabled
ln -s ../library/my-new-skill .

# Step 4: Commit to Git
cd ~/Projects/my_skils
git add library/my-new-skill enabled/my-new-skill
git commit -m "feat: add my-new-skill"
git push
```

### 4.2 Update Skill

```bash
# Edit directly
vim ~/Projects/my_skils/library/skill-name/SKILL.md

# Commit update
cd ~/Projects/my_skils
git add .
git commit -m "update: improve skill-name"
git push
```

### 4.3 Delete Skill

```bash
# Step 1: Disable first
rm ~/Projects/my_skils/enabled/skill-name

# Step 2: Delete source files
rm -rf ~/Projects/my_skils/library/skill-name

# Step 3: Commit
cd ~/Projects/my_skils
git add .
git commit -m "remove: delete skill-name"
git push
```

### 4.4 Batch Enable Skills

```bash
cd ~/Projects/my_skils/enabled

# Enable multiple skills
for skill in task-manager xlsx pdf docx; do
    ln -s ../library/$skill .
done
```

### 4.5 Reset to Default

```bash
cd ~/Projects/my_skils/enabled

# Clear all enabled
rm -f *

# Enable default skills
for skill in doc-coauthoring docx internal-comms pdf pptx xlsx \
             notion-knowledge-capture notion-research-documentation \
             notion-spec-to-implementation notebooklm task-manager; do
    ln -s ../library/$skill .
done
```

---

## Part 5: Cross-Platform Deployment

### 5.1 New Device Checklist

| Step | macOS/Linux | Windows |
|------|-------------|---------|
| 1 | `git clone ...` | `git clone ...` |
| 2 | `ln -s .../enabled ~/.claude/skills` | `mklink /J ...` |
| 3 | Done | Done |

### 5.2 Sync Updates

```bash
cd ~/Projects/my_skils
git pull
# Symlinks automatically point to latest content, no extra action needed
```

### 5.3 Different Configurations Per Device

If different devices need different skills:

```bash
# Option 1: Use Git branches
git checkout -b device-macbook
# Adjust enabled/ contents
git commit -am "macbook config"

# Option 2: Locally ignore enabled/
echo "enabled/" >> .git/info/exclude
# enabled/ changes won't be tracked
```

---

## FAQ

### Q1: Skills Not Working

**Check steps**:

```bash
# Check 1: System link correct?
ls -la ~/.claude/skills
# Should show: ~/.claude/skills -> .../my_skils/enabled

# Check 2: enabled links correct?
ls -la ~/Projects/my_skils/enabled/
# Should show each skill pointing to ../library/xxx

# Check 3: Skill file exists?
cat ~/Projects/my_skils/library/skill-name/SKILL.md
```

### Q2: Broken Symlinks

```bash
# Find broken links
find ~/Projects/my_skils/enabled -type l ! -exec test -e {} \; -print

# Fix: Remove broken link and recreate
rm ~/Projects/my_skils/enabled/broken-link
cd ~/Projects/my_skils/enabled
ln -s ../library/skill-name .
```

### Q3: Windows Symlink Failed

```cmd
REM Use Junction instead (no admin required)
mklink /J target source

REM Or enable developer mode and retry
```

### Q4: Git Not Tracking Symlinks

```bash
# Check Git config
git config core.symlinks

# Set to true
git config --global core.symlinks true

# Re-clone
rm -rf my_skils
git clone git@github.com:smartchainark/my_skils.git
```

---

## Best Practices

### Do's

- ✅ Store skills only in `library/`
- ✅ Control enable state via symlinks
- ✅ Regularly `git pull` to sync updates
- ✅ Test new skills before enabling
- ✅ Keep SKILL.md format standardized

### Don'ts

- ❌ Don't create real directories in `enabled/`
- ❌ Don't manually place files in `~/.claude/skills/`
- ❌ Don't delete skills in `library/` that are in use
- ❌ Don't skip Git commits (keep synced)

### Skill Naming Convention

```
✓ task-manager          # lowercase, hyphen-separated
✓ web3-frontend-scaffold
✗ TaskManager           # Don't use camelCase
✗ task_manager          # Don't use underscores
```

### Directory Structure

```
library/
└── skill-name/
    ├── SKILL.md        # Required, main skill file
    ├── templates/      # Optional, template files
    ├── examples/       # Optional, examples
    └── data/           # Optional, data files
```

---

## Quick Reference

| Action | Command |
|--------|---------|
| List all skills | `ls ~/Projects/my_skils/library/` |
| List enabled | `ls ~/Projects/my_skils/enabled/` |
| Enable skill | `cd enabled && ln -s ../library/xxx .` |
| Disable skill | `rm enabled/xxx` |
| Sync updates | `git pull` |
| Add skill | `mkdir library/xxx && vim library/xxx/SKILL.md` |

---

## References

- [Claude Code Skills Documentation](https://docs.anthropic.com/claude-code/skills)
- [GitHub Repository - my_skils](https://github.com/smartchainark/my_skils)
