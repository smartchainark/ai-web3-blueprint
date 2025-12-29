# Claude Skill Creation Standardization SOP

> Make Claude automatically follow your skill library conventions when creating new skills

**Document ID**: 006
**Date**: 2025-12-29
**Tags**: `Claude Code` `Skills` `SOP` `Automation`

---

## Overview

When you ask Claude to create a new skill, without explicit conventions, Claude might:
- Create directories directly in `~/.claude/skills/` (bypassing your skill library)
- Create actual directories in `enabled/` (instead of symlinks)
- Forget to update the README index

This document explains how to use the **skill-creator skill** to make Claude automatically:
1. Detect if user has a custom skill library
2. Create in the correct location following conventions
3. Enable via symlinks
4. Update the skill index

---

## Part 1: Problem Scenarios

### 1.1 Common Mistakes

| Wrong Action | Problem |
|--------------|---------|
| `mkdir ~/.claude/skills/new-skill` | Bypasses skill library, no Git management |
| `mkdir ~/Projects/my_skils/enabled/new-skill` | Creates actual directory in enabled |
| `cp -r library/skill enabled/` | Copies files instead of symlinking |
| Forgetting to update README | Incomplete skill list |

### 1.2 Correct Approach

```
User says: Help me create an xxx skill
    ↓
Claude checks: Does user have a skill library?
    ↓
Yes → Read README to understand conventions
    ↓
Create in library/
    ↓
Symlink to enabled/
    ↓
Update README index
```

---

## Part 2: Solution

### 2.1 Core Idea

Add creation conventions to the `skill-creator` skill, so Claude automatically reads and follows them when creating new skills.

### 2.2 Skill Library Detection Logic

Add to `skill-creator/SKILL.md`:

```markdown
## User Skill Library Conventions (Must Read)

> **Important**: Before creating any skill, must check if user has a custom skill library.

### Step 0: Check User Skill Library

Before creating any skill, **must** execute:

\`\`\`bash
# Check if user has custom skill library
ls ~/Projects/my_skils/README.md 2>/dev/null && echo "✅ Has skill library" || echo "❌ No skill library"
\`\`\`
```

### 2.3 Two Scenario Handling

**Scenario A: User Has Skill Library**

```markdown
### Scenario A: User Has Skill Library

If `~/Projects/my_skils/` exists, must follow its conventions:

1. **Read README first** — `cat ~/Projects/my_skils/README.md`
2. **Create in `library/`** — Don't create in `~/.claude/skills/`
3. **Enable via symlink** — Don't copy files
4. **Update README** — Add new skill to skill list

\`\`\`bash
# Correct approach
mkdir -p ~/Projects/my_skils/library/new-skill
# Write SKILL.md
cd ~/Projects/my_skils/enabled && ln -s ../library/new-skill .
# Update README skill list
\`\`\`
```

**Scenario B: User Has No Skill Library (First Time)**

```markdown
### Scenario B: User Has No Skill Library (First Time)

**Recommended**: Suggest user use Git-managed skill library.

Introduce to user:

> 💡 **Recommend Git-managed Skill Library**
>
> This approach lets your skills:
> - **Cross-platform reuse** — One set for macOS / Linux / Windows
> - **Multi-device sync** — Git managed, one-click deploy on new devices
> - **Flexible enable** — Enable on demand, keep environment clean
> - **Version iteration** — Skills continuously optimized, history trackable
>
> Would you like me to initialize a skill library for you?

If user agrees, execute initialization script to create complete skill library structure.
```

---

## Part 3: Implementation Steps

### 3.1 Update skill-creator Skill

Add conventions to `~/Projects/my_skils/library/skill-creator/SKILL.md`:

```bash
# Edit skill-creator skill
vim ~/Projects/my_skils/library/skill-creator/SKILL.md
```

Add "User Skill Library Conventions (Must Read)" section (see 2.2-2.3 above).

### 3.2 Update Skill Library README

Add to end of `~/Projects/my_skils/README.md`:

```markdown
## Claude New Skill Conventions

> **Important**: When Claude is asked to create new skills, must follow this process.

### Required Process

1. **Read this README first** — Understand existing skill categories and naming conventions
2. **Check if exists** — `ls ~/Projects/my_skils/library/` avoid duplicate creation
3. **Create in `library/`** — Don't create directly in `enabled/` or `~/.claude/skills/`
4. **Enable on demand** — Use symlinks, don't copy files

### Correct Approach

\`\`\`bash
# 1. Create skill directory
mkdir -p ~/Projects/my_skils/library/new-skill

# 2. Create SKILL.md
# (Write skill content)

# 3. Enable (create symlink)
cd ~/Projects/my_skils/enabled && ln -s ../library/new-skill .

# 4. Update this README's skill list
\`\`\`

### Wrong Approach

\`\`\`bash
# ❌ Create directly in ~/.claude/skills/
mkdir ~/.claude/skills/new-skill

# ❌ Create actual directory in enabled/
mkdir ~/Projects/my_skils/enabled/new-skill

# ❌ Copy files instead of symlink
cp -r library/skill enabled/skill
\`\`\`
```

### 3.3 Commit Updates

```bash
cd ~/Projects/my_skils
git add .
git commit -m "docs: add skill creation SOP to README and skill-creator"
git push
```

---

## Part 4: Verification Testing

### 4.1 Test Scenario

In a new session, tell Claude:

> Help me create a `code-review` skill for code review

### 4.2 Expected Behavior

Claude should:

1. ✅ First check if `~/Projects/my_skils/README.md` exists
2. ✅ Read README to understand existing skills and conventions
3. ✅ Create directory in `library/code-review/`
4. ✅ Write `SKILL.md`
5. ✅ Create symlink in `enabled/`
6. ✅ Update README skill list

### 4.3 Verification Commands

```bash
# Check skill location
ls ~/Projects/my_skils/library/code-review/SKILL.md

# Check symlink
ls -la ~/Projects/my_skils/enabled/code-review
# Should show: code-review -> ../library/code-review

# Check README updated
grep "code-review" ~/Projects/my_skils/README.md
```

---

## Part 5: First-Time User Guidance

For new users without a skill library, skill-creator will prompt:

```
💡 Recommend Git-managed Skill Library

This approach lets your skills:
- Cross-platform reuse — One set for macOS / Linux / Windows
- Multi-device sync — Git managed, one-click deploy on new devices
- Flexible enable — Enable on demand, keep environment clean
- Version iteration — Skills continuously optimized, history trackable

Would you like me to initialize a skill library for you?
```

If user agrees, Claude executes initialization script:

```bash
# 1. Create skill library directory structure
mkdir -p ~/Projects/my_skils/{library,enabled}

# 2. Create README (with complete conventions)
cat > ~/Projects/my_skils/README.md << 'EOF'
# Claude Skills Library
...
EOF

# 3. Create symlink
ln -sf ~/Projects/my_skils/enabled ~/.claude/skills

# 4. Initialize Git (optional)
cd ~/Projects/my_skils && git init
```

---

## FAQ

### Q1: Claude Still Created Skill in Wrong Location

**Cause**: skill-creator skill not enabled, or conventions not properly added.

**Solution**:
```bash
# Ensure skill-creator is enabled
ls ~/Projects/my_skils/enabled/skill-creator

# If not enabled
cd ~/Projects/my_skils/enabled && ln -s ../library/skill-creator .

# Check if conventions exist
grep "User Skill Library" ~/Projects/my_skils/library/skill-creator/SKILL.md
```

### Q2: Conventions Don't Work in New Session

**Cause**: Claude may not automatically read skill-creator in new sessions.

**Solution**: When requesting skill creation, explicitly mention "use skill-creator" or "/skill-creator".

### Q3: How to Make Conventions Auto-Apply in All Sessions

**Approach**: Also put core conventions in README.md, Claude reads project README before creating skills.

---

## Best Practices

### Do's

- ✅ Keep skill-creator skill always enabled
- ✅ Regularly check README skill list is complete
- ✅ `git commit` immediately after new skill creation
- ✅ Maintain latest conventions in skill-creator

### Don'ts

- ❌ Don't maintain same conventions in multiple places (easy to be inconsistent)
- ❌ Don't ignore README updates
- ❌ Don't use different conventions on different devices

---

## Skill Library Structure Reference

```
my_skils/
├── README.md                 # Skill index + creation conventions
├── library/                  # All skills (actual files)
│   ├── skill-creator/        # Skill creator (with conventions)
│   │   └── SKILL.md
│   ├── task-manager/
│   ├── gemini-cli/
│   └── ...
└── enabled/                  # Enabled (symlinks)
    ├── skill-creator -> ../library/skill-creator
    ├── task-manager -> ../library/task-manager
    └── ...
```

---

## References

- [004 - Claude Skills Cross-Platform Setup](./004-claude-skills-setup.md)
- [Claude Code Skills Documentation](https://docs.anthropic.com/claude-code/skills)
- [Skill Library GitHub](https://github.com/smartchainark/my_skils)
